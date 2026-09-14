public List<SemanticValidationWarning> ValidateGeometryClosenessToRoad(DtroSubmit dtroSubmit)
{
var wktReader = new WKTReader();
var wktWriter = new WKTWriter();
var passedInGeometries = dtroSubmit
.Data
.GetValueOrDefault<IList<object>>("Source.Provision".ToBackwardCompatibility(dtroSubmit.SchemaVersion))
.OfType<ExpandoObject>()
.SelectMany(provisions => provisions
.GetValueOrDefault<IList<object>>("RegulatedPlace".ToBackwardCompatibility(dtroSubmit.SchemaVersion))
.OfType<ExpandoObject>())
.Where(expandoObject => Constants.ConcreteGeometries.Any(expandoObject.HasField))
.ToList();

    string lineString;
    var scores = new List<double>();
    // runs at runtime. is that fine?
    // improve error handling
    foreach (dynamic geo in passedInGeometries)
    {
        //this is hardcoded. does it work for all dtros
        lineString = geo.DirectedLinear.directedLineString;
        var geometry = wktReader.Read(lineString);
        var sinuosity = Sinuosity(geometry);
        var num_of_vertices = NoOfVertices(geometry);
        //todo both are giving an exception
        var coverage = NetworkCoverage(geometry);
        // var coverage = 1.0;
        // var (mean, p95) = NetworkDistanceStats(geometry);
        var (mean, p95) = (1.0, 1.0);
        scores.Add(RoadNetworkFitScore(coverage, mean, p95, sinuosity));

    }
    var total_score = scores.Average();

    Console.WriteLine("===SCORE AVERAGE===");
    Console.WriteLine(total_score);
    Console.WriteLine("==========================");
    if (total_score < 1.0)
    {
        var warning = new SemanticValidationWarning
        {
            Name = "Geometry is not that close to road",
            Message = "You achieved score of 0.8. score of 1.0 is needed",
            Path = "->",
            Rule = ""
        };
        _warnings.Add(warning);

    }
    return _warnings;

}

//todo fix up
private static double Sinuosity(Geometry geometry)
{
if (geometry.Coordinates.Length < 2)
{
Console.WriteLine("ERROR: Less than 2 coordinates");
return 0.0;
}
var start = geometry.Coordinates.First();
var end = geometry.Coordinates.Last();
var distance = start.Distance(end);

    if (distance == 0)
    {
        Console.WriteLine("ERROR: Degenerate Line");
        return 0.0;
    }

    return geometry.Length / distance;

}

private int NoOfVertices(Geometry geometry){
return geometry.NumPoints;
}

private double NetworkCoverage(Geometry geometry)
{
var distanceToRoad = 3.0;
var n = 50;

    var indexedLine = new LengthIndexedLine(geometry);
    var totalLength = geometry.Length;
    var points = new List<Point>();
    for (var i = 0; i <= n; i++)
    {
        var fraction = (double)i / n;
        var distanceAlongLine = fraction * totalLength;

        Coordinate coord = indexedLine.ExtractPoint(distanceAlongLine);
        points.Add(new GeometryFactory().CreatePoint(coord));
    }


    var hits = 0;

    foreach (var point in points)
    {
        // 1. Create a search envelope around the point matching point.buffer(distance_to_road)
        var searchEnvelope = point.EnvelopeInternal;
        searchEnvelope.ExpandBy(distanceToRoad);

        // 2. Query the STRtree for road segment candidates within the envelope
        // var candidateRoads = roadTree.Query(searchEnvelope);
        // if (candidateRoads.Count == 0)
        // {
        //     continue;
        // }

        // 3. Find the minimum physical distance among candidates (mimicking Python min())
        // var minDistance = candidateRoads.Min(point.Distance);

        // if (minDistance <= distanceToRoad)
        // {
        //     hits++;
        // }
    }
    if (geometry.GeometryType != "LineString")
    {
        return 0.0;
    }
    else
    {
        return 1.0;
    }

    // return (double)hits / points.Count;

}

public (double Mean, double Percentile95) NetworkDistanceStats(Geometry userGeom)
{

    var n = 50;
    var searchDistance = 50.0;

    var indexedLine = new LengthIndexedLine(userGeom);
    var totalLength = userGeom.Length;
    var points = new List<Point>();

    for (var i = 0; i <= n; i++)
    {
        var fraction = (double)i / n;
        var distanceAlongLine = fraction * totalLength;

        Coordinate coord = indexedLine.ExtractPoint(distanceAlongLine);
        points.Add(new GeometryFactory().CreatePoint(coord));
    }

    var distances = new List<double>();

    foreach (var point in points)
    {
        var searchEnvelope = point.EnvelopeInternal;
        searchEnvelope.ExpandBy(searchDistance);

        var candidateRoads = roadTree.Query(searchEnvelope);

        if (candidateRoads.Count == 0)
        {
            // No road nearby – assign max penalty distance
            distances.Add(searchDistance);
        }
        else
        {
            var minimumDistance = candidateRoads.Min(point.Distance);
            distances.Add(minimumDistance);
        }
    }

    var mean = distances.Average();

    var percentile95 = GetPercentile(distances, 95);

    return (mean, percentile95);

}

private static double GetPercentile(List<double> sequence, double percentile)
{
var sortedSequence = sequence.OrderBy(x => x).ToList();
var count = sortedSequence.Count;

    if (count == 0) return 0;
    if (count == 1) return sortedSequence[0];

    // Map percentile to a virtual fractional index position
    var index = (percentile / 100.0) * (count - 1);
    var lowerIndex = (int)Math.Floor(index);
    var upperIndex = (int)Math.Ceiling(index);

    if (lowerIndex == upperIndex)
    {
        return sortedSequence[lowerIndex];
    }

    var weight = index - lowerIndex;
    return (1.0 - weight) * sortedSequence[lowerIndex] + weight * sortedSequence[upperIndex];

}

public double RoadNetworkFitScore(double coverage, double meanDist, double p95Dist, double sinuosity)
{

    // not sure on the random numbers
    if (coverage < 0.5)
    {
        return 0.0;
    }

    if (sinuosity < 1.02 && p95Dist > 10.0)
    {
        return 10.0;
    }

    var coverageScore = Math.Min(coverage / 0.9, 1.0);

    var meanScore = Math.Max(0.0, 1.0 - (meanDist / 10.0));
    var p95Score = Math.Max(0.0, 1.0 - (p95Dist / 20.0));
    var distanceScore = (0.7 * meanScore) + (0.3 * p95Score);

    var sinuosityScore = Math.Max(0.0, Math.Min((sinuosity - 1.0) / 0.2, 1.0));

    var finalScore = (0.5 * coverageScore) + (0.35 * distanceScore) + (0.15 * sinuosityScore);

    return Math.Round(finalScore * 100.0, 1);

}

private STRtree<Geometry> RoadTree(){
var reader = new GeoPack
}
