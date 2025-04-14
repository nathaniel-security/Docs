# 🧠💥 How I Routed My Entire Lab's HTB Traffic Through VPN (And Why You Should Too)

### 😤 The Problem

If you’re working in a lab environment and doing serious work with Hack The Box (HTB), chances are you've got automation running scanners, exploit frameworks, recon tools, and maybe even a full orchestration setup.

The catch? HTB traffic only routes through their VPN tunnel. So unless _every single_ box in your lab connects to the VPN (spoiler: bad idea), you’re stuck.

### 🎯 The Goal

Instead of duct-taping VPN clients on every box, here’s what we want:

* **Single system connected to HTB VPN**
* **All LAN devices route HTB-bound traffic through that system**
* **Clean iptables rules that don’t wreck your firewall**
* **Scriptable and reversible setup that “just works”**

### 🧠 The Setup

Here’s the architecture:

* **LAN Interface:** Your main interface (e.g., `ens33`)
* **VPN Interface:** Where your HTB VPN is connected (`tun0`)
* **LAN Subnet:** `192.168.0.0/24`
* **HTB Subnet:** `10.10.0.0/16`

We’re going to build a gateway that transparently routes traffic from your LAN to HTB over the VPN.

### 🛠️ The Script

```
# Define variables
LAN_IFACE="ens33"
# LAN_IFACE: The interface connected to your local network (home lab or automation server).
VPN_IFACE="tun0"
# VPN_IFACE: The interface created by HTB's OpenVPN connection.
LAN_SUBNET="192.168.0.0/24"
# LAN_SUBNET: The subnet used by your internal LAN devices.
HTB_SUBNET="10.10.0.0/16"  
# HTB_SUBNET: The IP range used by HTB target machines. This is what we want to reach via VPN.


echo "1" | sudo tee /proc/sys/net/ipv4/ip_forward
# 🚀 Enable IP forwarding
# Required so it can forward traffic from LAN to VPN

sudo iptables -F
sudo iptables -t nat -F
# Flushes all filter and NAT rules.
# ⚠️ Warning: This will wipe existing firewall rules. Fine for a lab, dangerous in production.


# NAT outgoing traffic to VPN
sudo iptables -t nat -A POSTROUTING -s $LAN_SUBNET -d $HTB_SUBNET -o $VPN_IFACE -j MASQUERADE
# Changes the source IP of packets from your LAN going to HTB, so they appear to come from your VPN interface IP.

# Allow traffic from LAN to VPN and back
sudo iptables -A FORWARD -s $LAN_SUBNET -d $HTB_SUBNET -i $LAN_IFACE -o $VPN_IFACE -j ACCEPT
# Allows packets going from LAN to HTB (via VPN)
sudo iptables -A FORWARD -d $LAN_SUBNET -s $HTB_SUBNET -i $VPN_IFACE -o $LAN_IFACE -j ACCEPT
# Allows return packets coming from HTB back to LAN

```

👽 Link to updated [code](https://gist.github.com/nathaniel-security/0b63812f96c7dfca8c66b7e6a4176d2f#file-connect_htb-sh)

### 💥Connect my client

```
sudo ip route add 10.10.11.165 via 192.168.0.251 dev wlp0s20f3
```

* `10.10.11.165` is an HTB IP address of the box
* `192.168.0.251` my proxy\_box IP address
* to delete the ip route

```
sudo ip route del 10.10.11.165 via 192.168.0.251 dev wlp0s20f3
```

### ⚡ Real-World Use Case

Now every system on my network even my toaster (I always knew it was destined for great things) can reach any HTB target, without needing its own VPN setup. 🤖🔗💣

### 💬 Final Thoughts

This isn’t about fancy scripts or over-engineering (I tend to do it a lot but don't we all). It’s about having a reliable, controlled setup that routes HTB traffic through your VPN **without reinventing the wheel or smashing your firewall with a sledgehammer**.\
You now have a surgical routing setup. There are no hacks, no weird side effects, and just a clean flow from your lab to HTB.
