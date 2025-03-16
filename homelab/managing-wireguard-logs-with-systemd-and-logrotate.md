# Managing WireGuard Logs with Systemd and Logrotate 🔥

When managing a **VPN like WireGuard**, logging is crucial for **monitoring activity, debugging issues, and ensuring security**. But if left unchecked, logs can **grow rapidly** and become unmanageable.

In this guide, we’ll **set up Systemd** to capture WireGuard logs dynamically and use **Logrotate** to keep them under control automatically.

### Setup a Systemd file to store logs

**Step 1: Create a Systemd Service to Store Logs**

First, we need to create a Systemd service that continuously logs WireGuard activity.

🔹 Open a new Systemd service file:

```
nano /etc/systemd/system/wireguard-log.service
```

🔹 Add the following configuration:

```
[Unit]
Description=WireGuard Dynamic Debug Logging
After=network.target

[Service]
ExecStart=/bin/bash -c 'dmesg -wT | grep wireguard >> /var/log/wireguard-dyndbg.log'
Restart=always
RestartSec=5
StandardOutput=null
StandardError=null

[Install]
WantedBy=multi-user.target
```

🔹 Reload and restart Systemd to apply changes:

```
sudo systemctl daemon-reload
sudo systemctl restart wireguard-log.service
sudo systemctl enable wireguard-log.service
```

🔹 Verify that the service is running:

```
sudo systemctl status wireguard-log.service
```

### **Set Up Logrotate for Automatic Log Management**

Now, let’s ensure our logs don’t grow indefinitely by setting up **Logrotate**.

🔹 Install Logrotate (if not already installed):

```
sudo apt update && sudo apt install logrotate -y
```

🔹 Create a Logrotate configuration file:

```
nano /etc/logrotate.d/wireguard
```

🔹 Add the following configuration to manage log rotation:

```
/var/log/wireguard-dyndbg.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 root root
    postrotate
        systemctl restart wireguard-log.service > /dev/null 2>&1 || true
    endscript
}

```

Test Your Log Rotation Setup



```
sudo logrotate -v /etc/logrotate.d/wireguard
```

To **force** log rotation manually:

```
sudo rm -f /var/lib/logrotate/status
sudo logrotate -f /etc/logrotate.d/wireguard
```

```
ls -lh /var/log/wireguard-dyndbg.log*
```
