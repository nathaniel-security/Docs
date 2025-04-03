# Cloudflare Tunnels: The Cybersecurity Pro's Secret Weapon for Secure Cloud Access

### 🔥 Backstory: Why I Set This Up

* One of my internal apps was **exposed on the internet**  despite being **proxied through Cloudflare**.
* Anyone scanning the internet (like Shodan or Censys) could hit it directly, even though Cloudflare was enabled
* Enter: **Cloudflare Tunnel**  no exposed ports, no firewall nightmares, no BS.

***

Would you like me to add a little `shodan.io` search reference or a quote from your internal logs to give it that gritty realness?

### ✨ TL;DR

Use **Cloudflare Tunnel** to securely expose multiple web apps (on different ports) to the internet using **just one tunnel**, without opening a single port on your firewall. This blog walks through:

* Creating the tunnel
* Mapping subdomains to services
* Auto-starting with systemd

***

### 🧠 Why Use Cloudflare Tunnel?

* 🔐 No port forwarding
* 💸 Free for personal and dev use
* 🌍 Expose multiple apps via subdomains
* 📦 Works great with local HTTPS and Docker

***

### ⚙️ My Setup

* OS: Ubuntu 22.04 (but any Linux works)
* Cloudflare domain: `wehost.co.in`
* Goal: Expose multiple local apps like:
  * `test.wehost.co.in` → `localhost:8002`

***

### 🚀 Step 1: Install `cloudflared`

```
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
```

### 🔐 Step 2: Authenticate with Cloudflare

```
cloudflared tunnel login
```

* This opens a browser window. Select your domain, and Cloudflare will generate a credentials file.

### 🌪️ Step 3: Create the Tunnel

```
cloudflared tunnel create test
```

This creates a tunnel and saves a `.json` credentials file under `~/.cloudflared/`.

You can confirm with:

```
cloudflared tunnel list
```

### 🛠️ Step 4: Create Config File

Create a `config.yml` in `~/.cloudflared/`:

```
nano ~/.cloudflared/config.yml
```

```
tunnel: <YOUR_TUNNEL_ID>
credentials-file: /home/groot/.cloudflared/<YOUR_TUNNEL_ID>.json

ingress:
  - hostname: test.wehost.co.in
    service: https://localhost:8002
    originRequest:
      noTLSVerify: true
  - service: http_status:404

```

Replace `<YOUR_TUNNEL_ID>` with the one from `cloudflared tunnel list`.

### 🌐 Step 5: Route the Subdomain

```
cloudflared tunnel route dns test test.wehost.co.in
```

This links your tunnel to the subdomain in Cloudflare DNS.

### ✅ Step 6: Run the Tunnel

```
cloudflared tunnel --config ~/.cloudflared/config.yml run test
```

You should see output like:

```
Starting tunnel tunnelID=...
Using CurveP256...
```

### 🔁 Step 7: Auto-Start on Boot with systemd

Create the service file:

```
sudo nano /etc/systemd/system/cloudflared-test.service
```

```
[Unit]
Description=Cloudflare Tunnel - test
After=network.target

[Service]
TimeoutStartSec=0
Restart=always
ExecStart=/usr/local/bin/cloudflared tunnel --config /home/groot/.cloudflared/config.yml run test
User=groot
Environment=HOME=/home/groot

[Install]
WantedBy=multi-user.target

```

```
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable cloudflared-test
sudo systemctl start cloudflared-test
```

### 🔮 What’s Next: Auto-Magic with CI/CD

Now that the tunnel’s solid, I’m planning to take it a step further:

> **The next iteration of this setup will integrate with a CI/CD pipeline** so every time I deploy a new app or update port mappings, the pipeline will:
>
> * Automatically update the `config.yml`
> * Push it to the server
> * Restart the Cloudflare tunnel service
> * Route any new subdomains on the fly

Think: zero-touch deployments with automatic subdomain provisioning. No more manual edits, no more downtime  just ship and forget.

Stay tuned — that one’s gonna be fun. 🔧🚀
