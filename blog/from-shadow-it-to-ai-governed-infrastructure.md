---
cover: >-
  https://images.unsplash.com/photo-1600267204091-5c1ab8b10c02?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwxfHxkb2N1bWVudGF0aW9ufGVufDB8fHx8MTc2NjI2MDgzMXww&ixlib=rb-4.1.0&q=85
coverY: 0
---

# From Shadow IT to AI-Governed Infrastructure

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

### Fighting Shadow IT with the Shadow I Created

**Problem**: My homelab is pretty big multiple VLANs, dozens of services, mix of VMs and containers. Plus I have family members who need servers for testing things, and friends working on shared projects with dedicated VMs that have different network access. I used to remember everything: which server runs what, which configs are weird, which services depend on each other, who has access to what. That doesn't scale beyond a certain point, and definitely doesn't work with AI agents.

**Solution**: Built an automated SSH-based discovery system that documents what's actually running where, in a format both humans and AI can understand.

### Why This Matters in a Homelab

* Rebuilding servers takes forever when you can't remember what they do
* You're the single point of failure for your own infrastructure
* Family and friends need access but you can't explain what everything does
* New services pop up and you forget to document them
* AI agents can't help manage what they can't see

### Real Problems I Hit

#### "What runs on that server again?"

Spent 3 hours SSHing around trying to figure out what 192.168.1.42 actually does before I could safely reboot it.

#### "Can I use this VM for my project?"

Friend needs a server for testing. I have no idea which VMs are free, which have special network configs, or what's safe to repurpose.

#### Things become production without me noticing

A test Docker container becomes critical infrastructure. Disk fills up, no monitoring, service dies silently.

#### Configs drift, monitoring doesn't

Migrated my VPN from WireGuard to Tailscale. Everything works fine, but my monitoring keeps checking old WireGuard ports and spamming "VPN DOWN" alerts.

#### Emergency fixes with no documentation

"Why did I disable this service? Why does this firewall rule exist?" Six months later, I have no idea. AI agents will "helpfully" fix these "bugs" and break things.

### My Solution: SSH-Based Discovery

**Core idea**: Instead of manually maintaining docs that get stale, build scripts that SSH into everything and document what's actually running.

#### What I built

* Scripts that scan my network and SSH into reachable boxes
* Collect system info with read-only commands
* Generate structured docs (YAML + Markdown)
* Keep manual notes but auto-update system facts
* Safe boundaries for what AI agents can do

### How It Works

#### The Scripts

1. **`scan_network.sh`** - Finds hosts with SSH open on my network
2. **`collect_host_multikey.sh`** - SSHs into each box and runs safe commands:
   * `cat /etc/os-release` - What OS?
   * `ip addr` - What IPs?
   * `ss -tuln` - What ports are open?
   * `df -h` - How much disk space?
3. **Smart analysis** - Looks at hostname and running services:
   * `personal-docker-server` → "Container host running web services"
   * `wireguard-wehost-local` → "VPN gateway"
4. **Docs generation** - Creates `docs/hostname_ip.md` with YAML + Markdown
5. **Smart updates** - Refreshes system data, keeps my manual notes

#### Claude Integration

* **Safe actions**: Claude can restart services, check logs, generate reports
* **Forbidden actions**: Can't delete files, modify configs, access secrets
* **Context awareness**: Knows what each server does, what depends on what
* **Same format everywhere**: Homelab and cloud docs use identical structure

### What I Got Out of This

* **Faster troubleshooting**: 3 hours → 15 minutes to figure out what's broken
* **Confident rebuilds**: No guesswork, just follow the documented setup
* **Better collaboration**: Friends helping with my lab can understand what things do
* **AI assistance**: Claude can actually help manage stuff instead of just guessing

### Technical Details

#### Security Setup

* SSH keys in `ssh/` directory, script tries all of them for each host
* Does a TCP check first, skips boxes that aren't reachable
* Only runs read-only commands (`cat`, `ls`, `ip`, etc.)
* Skips hosts where SSH auth fails

#### Doc Format

```yaml
---
hostname: personal-docker-server
ip_address: 192.168.1.42
os_release: Ubuntu 22.04.3 LTS
services: [docker, nginx, postgresql]
summary: "Container host running web services"
---

# My manual notes (these stick around)
# System info (gets refreshed automatically)
```

### Why This Matters

My homelab is too big to keep in my head anymore. And if I want AI agents to help manage it safely, they need to understand what everything does.

The solution isn't perfect documentation it's systems that document themselves and stay current automatically.

Build infrastructure that explains itself. Build documentation that enables safe automation. Build automation that preserves your manual insights.

Link to Claude Skill (I will be uploading the skill soon)
