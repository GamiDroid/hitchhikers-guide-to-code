---
tags:
  - raspberry-pi
  - guide
---
The Pi has a hardware watchdog that reboots the machine if it stops responding — great for crash recovery.

```bash
# Install watchdog deamon 
sudo apt install watchdog

# Configure it
sudo nano /etc/watchdog.conf
```

Key settings in `/etc/watchdog.conf`:

```
watchdog-device = /dev/watchdog
watchdog-timeout = 15
max-load-1 = 24

# ping the router to check the network is available.
ping = 192.168.2.254
```

Add watchdog deamon as service:

```bash
sudo systemctl enable watchdog
sudo systemctl start watchdog
```

