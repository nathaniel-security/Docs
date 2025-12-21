---
cover: >-
  https://images.unsplash.com/photo-1600267204091-5c1ab8b10c02?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwxfHxkb2N1bWVudGF0aW9ufGVufDB8fHx8MTc2NjI2MDgzMXww&ixlib=rb-4.1.0&q=85
coverY: 0
---

# From Shadow IT to AI-Governed Infrastructure

### Scaling Infrastructure Documentation Through Autonomous Discovery and Agent Integration

**TLDR**: Traditional infrastructure management relies on institutional memory and manual documentation, creating operational risk and preventing autonomous systems integration. This blog presents a production system that automatically discovers, documents, and maintains infrastructure state through SSH-based reconnaissance, enabling AI-driven operations while reducing MTTR and eliminating shadow IT.

**The Business Problem**: Infrastructure teams face exponentially growing technical debt from undocumented systems, configuration drift, and tribal knowledge dependency.&#x20;

In environments prioritizing rapid iteration over formal IaC adoption, this creates significant operational risk:

* **Mean Time to Resolution (MTTR)** increases exponentially with system complexity
* **Knowledge bus factor** of 1 creates single points of failure in operations teams
* **Shadow IT discovery** requires manual auditing processes that don't scale
* **Agent-based automation** becomes impossible without machine-readable system state

### The Problems: Why Memory-Driven Infrastructure Fails Agents

1. **Agent Archaeology Is Impossible** When Claude asks "What runs here?", SSH + grep + guess isn't an answer it's a vulnerability. Agents need facts, not forensics.

Real impact: Every rebuild becomes a 3-hour investigation. Every automation attempt risks destroying something critical because intent is undocumented.

2. **Silent Failures Create Invisible Infrastructure** Real example: A test server became "production" without documentation updates. No log rotation. Disk filled to 100%, server died silently. No alerts, just... nothing.

**The agent problem**: If an LLM can't see a system in documentation, it can't manage, monitor, or protect it. Undocumented infrastructure is unmanaged infrastructure.

3. **Configuration Drift Breaks Agent Logic** Real example: Migrated `wireguard-gateway` to Tailscale. VPN worked fine, but monitoring still checked for WireGuard, reporting "VPN down" while family and services connected happily via Tailscale.

**The agent problem**: Documentation drift means agents operate on false assumptions. Green lights + broken reality = agent decisions based on lies.

4. **Tribal Knowledge Is Agent Poison** Why does this firewall rule exist? Why is this service disabled but not removed?

**The agent problem**: Unexplained configurations look like bugs to AI. Agents will "fix" perfectly intentional weirdness because context doesn't exist in any machine-readable form.&#x20;

### The Realization: Infrastructure as a Data Layer

I didn't need better documentation. I needed infrastructure that's rebuildable without my memory and **readable by agents**.

Documentation isn't prose or a wiki. It's a data layer that explains intent, records history, and survives both rebuilds and AI governance.

### The Goal: Agent-Native Infrastructure

I want Claude to safely manage my homelab and AWS. That's impossible if:

* Intent is undocumented
* Exceptions are tribal knowledge
* Drift is invisible
* System state is unknowable

Without a machine-readable data layer, Claude would be guessing. **Guessing in infrastructure burns accounts.**

### How It Works: SSH Discovery → Agent-Ready Documentation

#### The Auto-Discovery Pipeline

1. **Network Scanning**: `scan_network.sh` safely discovers SSH-accessible hosts on `192.168.0.0/24`
2. **SSH Collection**: `collect_host_multikey.sh` connects with repository keys, runs read-only commands (`cat`, `ls`, `ip`, `ss`, `df`)
3. **Intelligent Analysis**: Analyzes hostname patterns and services to generate executive summaries
   * `personal-docker-server` → "Container orchestration platform with web frontend and database backend"
   * `wireguard-wehost-local` → "VPN gateway server providing secure remote network access"
4. **Structured Output**: YAML frontmatter + Markdown content in `docs/hostname_ip.md`
5. **Smart Updates**: Preserves manual annotations while refreshing system data

#### Agent Integration Features

* **Infrastructure Contracts**: Every resource documents purpose, dependencies, rebuild procedures
* **Automation Boundaries**: Claude can restart services, not delete data
* **Drift Detection**: System changes trigger documentation updates
* **One Schema**: Same format for homelab and AWS—unified reasoning model for agents

The Result Rebuilds: no guessing, no archaeology, no fear. Failures: I know intent, boundaries, and history. Claude: has context, not just metrics.

The Hard Truth If your infrastructure only works because you remember it, it doesn't actually work.

Documentation isn't overhead. It's the data layer that makes rebuilds safe, automation sane, and AI operations possible.

Build infrastructure that survives you forgetting.

Link to Github
