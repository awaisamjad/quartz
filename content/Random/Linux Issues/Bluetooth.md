# Bluetooth not working

## Possible issues
- The Bluetooth service isn't running.
The adapter is soft/hard blocked by rfkill.
The firmware/driver didn't load.
A desktop environment service that normally enables Bluetooth isn't present.

Start by checking the basics.

1. Check if the Bluetooth service is running
   systemctl status bluetooth

If it's inactive:

sudo systemctl enable --now bluetooth

Then verify:

systemctl status bluetooth 2. Check whether the adapter is blocked
rfkill list

Look for something like:

Bluetooth
Soft blocked: yes
Hard blocked: no

If soft blocked:

rfkill unblock bluetooth

or

rfkill unblock all

Then check again.

3. See if Fedora detects the Bluetooth hardware
   bluetoothctl list

You should see something like:

Controller XX:XX:XX:XX:XX:XX hostname [default]

If nothing appears, the kernel may not be seeing the adapter.

Check:

lsusb

for USB adapters, or

lspci -nnk

for internal adapters.

4. Check kernel messages
   dmesg | grep -i bluetooth

and

dmesg | grep -i firmware

Often firmware-loading failures show up here.

5. Check BlueZ directly
   bluetoothctl

Then inside the prompt:

show
power on

If power on returns an error, that often points to rfkill or driver issues.

6. Verify required packages
   rpm -qa | grep bluez

You should have packages such as:

bluez
bluez-tools
bluez-libs

If not:

sudo dnf install bluez bluez-tools

Can you paste the output of these commands?

systemctl status bluetooth --no-pager
rfkill list
bluetoothctl list
