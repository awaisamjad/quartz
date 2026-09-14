# Systems / Backend Depth

These are the ones that actually make you *dangerous* as a backend engineer.

### 1. Build a mini reverse proxy (nginx-lite)

* Features:

  * HTTP/1.1 parsing (raw, no frameworks)
  * Reverse proxy to upstreams
  * Load balancing (round robin, least connections)
  * Basic caching layer
* Bonus:

  * TLS termination later
  * Rate limiting

👉 This aligns perfectly with your `volk` server — you can evolve it into this.

---

### 2. **Write your own job queue (like Sidekiq / Celery)**

* Components:

  * Worker pool
  * Persistent queue (Redis or your own disk format)
  * Retry + backoff
  * Delayed jobs
* Stretch:

  * Distributed workers over TCP
  * Priority queues

👉 This teaches real production backend patterns.

---

### 3. **Build a simple database**

Not a toy — something real enough to learn internals.

* Start with:

  * Key-value store (append-only log)
  * In-memory index
* Then:

  * Compaction
  * WAL
  * B+ tree or LSM tree

👉 This will change how you think about data forever.

---

### 4. **Implement a file sync system (Syncthing-lite)**

* Watch filesystem changes
* Send diffs over network
* Conflict resolution
* Resume transfers

👉 You’ve already had pain with Syncthing — perfect motivation.

---

### 5. **Write a TCP-based chat server (no HTTP)**

* Raw sockets
* Custom protocol
* Multiple clients
* Message broadcasting

Stretch:

* Add encryption
* Presence + rooms

👉 This builds real networking intuition.

---

# 🌐 Real-world Backend Apps (portfolio-ready)

### 6. **Auth service (production-grade)**

* JWT + refresh tokens
* OAuth (Google/GitHub)
* Email verification
* Rate limiting
* Audit logs

👉 You literally have this on your Whisp backlog.

---

### 7. **Analytics ingestion service**

* Accept events via HTTP
* Batch + store efficiently
* Query API

Stretch:

* Use columnar storage
* Add real-time streaming (Kafka-lite)

---

### 8. **Multi-tenant SaaS backend**

* Organisations + users
* Role-based access control
* Rate limits per tenant
* Billing hooks (mock Stripe)

👉 This is what companies actually build.

---

### 9. **Webhook delivery system**

* Accept webhook registrations
* Retry logic (exponential backoff)
* Signing (HMAC)
* Dead letter queue

---

### 10. **Search engine (mini Elasticsearch)**

* Index documents
* Inverted index
* Ranking (TF-IDF)

---

# 🤖 AI-related backend projects (not just “call OpenAI API”)

### 11. **LLM gateway (very relevant to you)**

* Single API to:

  * route to local models (llama.cpp)
  * fallback to cloud (OpenAI, etc.)
* Features:

  * rate limiting
  * caching responses
  * prompt logging
  * cost tracking

👉 This fits your Hetzner + subdomain idea perfectly (`ai.awais.me`).

---

### 12. **Vector database (mini Pinecone)**

* Store embeddings
* Similarity search (cosine)
* API layer

Stretch:

* HNSW index
* Disk persistence

---

### 13. **RAG backend**

* Upload documents
* Chunk + embed
* Query with context retrieval

👉 Make it your own knowledge base replacement.

---

### 14. **Code assistant backend**

* Input: codebase
* Features:

  * semantic search
  * “explain this file”
  * “find bugs”

Stretch:

* integrate with Neovim

---

### 15. **AI task worker system**

* Queue tasks like:

  * summarisation
  * embeddings
  * code generation
* Workers process using local LLMs

👉 Combine this with your job queue project = 🔥

---

# ⚙️ Infra / DevOps-focused backend

### 16. **Service orchestrator (mini systemd / docker-lite)**

* Start/stop processes
* Restart policies
* Logging

---

### 17. **Distributed rate limiter**

* Token bucket
* Shared state across nodes (Redis or custom)

---

### 18. **Config-driven deployment tool**

* YAML → deploy services
* SSH + run commands
* Health checks

---

# 🧩 How to pick (important)

Don’t try to do everything. Pick based on your goal:

* **Want deep systems knowledge →**
  → database / proxy / TCP server

* **Want job-ready backend skills →**
  → auth service / SaaS backend / webhooks

* **Want to align with your current stack + interests →**
  → LLM gateway + job queue combo

---

# 🔥 My blunt recommendation for you

Given everything you’ve been doing:

Start this sequence:

1. **Job queue (core infra)**
2. **LLM gateway (uses the queue)**
3. Extend your **volk server into reverse proxy**

That combo is:

* practical
* impressive
* deeply technical
* reusable in your own infra

---

If you want, I can map out a **step-by-step build plan** (with architecture + folder structure) for one of these so you can actually execute instead of just collecting ideas.
