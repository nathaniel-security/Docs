# Opening SSH with Root Access to the World: A Controlled Honeypot Experiment

> _"If you're going to set a trap, make sure it looks like a jackpot."_

***

### 🐱 It Started with a One-Way Conversation

Before any real hackers showed up, I decided to have a little fun.\
I shared the open SSH port with a friend and told him to "explore freely."

What followed was pure comedy.

He logged in, typed commands like `clear`, `ls`, `cat`, `man`, and even tried chatting with the server:

> `"what the f*** is the server"`\
> `"why have you even exposed this on internet to waste our time"`\
> `"please get a life bro"`

Meanwhile, I sat back, silently watching him yell at a machine that logged everything without a response.

Here’s a live snapshot from that session

<figure><img src="../.gitbook/assets/Screenshot from 2025-04-28 23-33-08.png" alt=""><figcaption></figcaption></figure>

***

### 🎯 The Goal

After laughing way too hard, I got serious.\
This wasn’t about trolling, it was about building a **controlled honeypot** designed to capture real-world attacker behaviour.

The mission:

* **Expose a fake SSH service** offering **root access**.
* **Isolate** the honeypot completely from the real internet.
* **Control** entry points with tight firewalling and routing.
* **Observe** and log attacker behaviour safely.

In short:\
✅ Full control.\
✅ Full visibility.\
✅ Zero risk.

***

### 🛠️ The Architecture

The setup involved two servers:

| Machine                | Role                | IP / Interface                                       | Notes                                                                                                                               |
| ---------------------- | ------------------- | ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `shaddykrupa`          | Cowrie SSH Honeypot | `192.168.237.128` (host-only network)                | No internet access                                                                                                                  |
| `shaddy-reverse-proxy` | Reverse Proxy       | LAN Interface: `ens33`, Host-Only Interface: `ens37` | Port `2000` exposed to the Internet (exposed on port 22 publically so technically 3 layers of network translation 2 NAT and 1 PAT ) |

**Traffic flow:**

```plaintext
[ Attacker ]
   ⬇️
Public IP (Port 22 externally) 
   ⬇️
shaddy-reverse-proxy:2000 (LAN-facing)
   ⬇️
iptables DNAT
   ⬇️
shaddykrupa:2222 (Cowrie Honeypot)
```



### 🔥 The Firewall (iptables) Rules

Here’s the exact firewall script I used on `shaddy-reverse-proxy`:

```sh
# Flush old NAT rules if needed
sudo iptables -t nat -F

# Forward all TCP traffic hitting port 2000 on the LAN-facing interface to Cowrie
sudo iptables -t nat -A PREROUTING -i ens33 -p tcp --dport 2000 -j DNAT --to-destination 192.168.237.128:2222

# Allow forwarding
sudo iptables -A FORWARD -p tcp -d 192.168.237.128 --dport 2222 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT

# Masquerade the packets so they appear to come from the proxy
sudo iptables -t nat -A POSTROUTING -o ens37 -p tcp -d 192.168.237.128 --dport 2222 -j MASQUERADE


sudo iptables -A INPUT -m state --state INVALID -j DROP
sudo iptables -A FORWARD -m state --state INVALID -j DROP

```

This setup ensures:

* Only traffic on port `2000` is accepted and redirected.
* Packets are rewritten so Cowrie sees them correctly.
* Invalid traffic is dropped to harden the proxy itself.

***

### 🧪 Watching the World Come In

Once the trap was live, it didn’t take long.

Bots, opportunistic attackers, and noisy scanners started pouring in each believing they had **found an open root-access SSH server**.

They spammed password attempts, tried `wget` malware downloads, dropped crypto-miners,\
and some immediately fired off `rm -rf /*` like it was Christmas morning.

Meanwhile, Cowrie quietly logged everything.

***

### 📈 Key Observations (So Far)

* **Attackers behave differently** when they think they have root: bold, reckless, and noisy.
* **Speed is insane**: The first bot hit within minutes of exposure.
* **Defence in depth is non-negotiable**: Even honeypots need strong isolation.

***

### 🛡️ What's Next?

This is just the beginning.

I'm **continuing to log** and **analyse** all incoming activity.\
The real fun starts now: studying **what the bots do**, how they behave, and what tools and malware they attempt to deploy when they believe they've found root access.

Will try and put updates soon with:

* Full analysis of attacker behaviour over time
* Common attack patterns spotted in the wild
* Tricks attackers use once they believe they "own" a system

***

### 🧠 Final Thoughts

This wasn’t just about catching hackers / Bots — it was about **understanding them**.\
Learning how fast they move, how they think, and how to outwit them.

By exposing fake root access in a controlled, isolated, and fortified environment,\
I got a front-row seat to the chaos without ever risking a real system.

And honestly? Watching my friend get mad at a fake server was just a bonus.

***

## 🎯 TL;DR

I opened SSH with root access to the world, but the only ones who got pwned were the bots.

And a little bit... my friend too.



## Reference&#x20;

* [cowrie docker compose](https://gist.github.com/nathaniel-security/c7d662d0460af15f73e0c70c938b80f6)
* [iptables](https://gist.github.com/nathaniel-security/dc6165ae26b301f858a749e6a5db8df7)&#x20;

