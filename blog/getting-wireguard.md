---
description: just a place to documenting my homelab
---

# Getting Wireguard



* update and upgrade&#x20;



```
sudo apt-get update && sudo apt-get upgrade -y
```

\


* install wireguard

```
sudo apt-get install wireguard
```

\


* edit the sysctl.conf

```
sudo nano /etc/sysctl.conf
```

\


* uncomment \`net.ipv4.ip\_forward=1\`



```
net.ipv4.ip_forward=1
```

\


```
sudo sysctl -p
```

\


* wireguard conf

```
cd /etc/wireguard
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
sudo nano /etc/wireguard/wg0.conf
```

\


* Server conf (technically its a peer)

```
[Interface]
PrivateKey = <privatekey server>
Address = 10.0.0.1/24
# PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
# PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE
PostUp = iptables -A FORWARD -i wg0 -o ens33 -j ACCEPT; iptables -A FORWARD -i ens33 -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -o ens33 -j ACCEPT; iptables -D FORWARD -i ens33 -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens33 -j MASQUERADE
ListenPort = 51820
Table = off

[Peer]
PublicKey = <client public key>
AllowedIPs = 10.0.0.2/32 , 192.168.0.0/24 , 192.168.0.146/32
```





* clinet conf



```
wg genkey | tee privatekey | wg pubkey > publickey
```

```
[Interface]
Address = 10.0.0.2/32
PrivateKey =  <client private key key>
# DNS = 1.1.1.1

[Peer]
PublicKey = <server public key>
Endpoint = 110.110.110.110:51820
# Endpoint = <change to public ip>:51820
AllowedIPs = 192.168.0.0/24
PersistentKeepalive = 25

```

## References

* https://upcloud.com/resources/tutorials/get-started-wireguard-vpn
