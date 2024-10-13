# Install Wiregard VPN

* Wiregard does not follow tradition client server architecture its more of a peer architecture
* wiregard ip address on server has to be different and unique on the network

### On both client and server

```
sudo apt-add-repository universe
```

```
sudo apt-get update
```

```
sudo apt-get install wireguard-tools wireguard
```

```
wg genkey | sudo tee /etc/wireguard/private.key
sudo chmod go= /etc/wireguard/private.key
```

```
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

### On client

```
sudo nano /etc/wireguard/wg0.conf
```

#### Config File

```
[Interface]
PrivateKey = <CLIENT PRIVATE KEY>
Address = <IP ADRESS THAT IS TO BE ASSIGNED TO CLIENT>
ListenPort = <ListenPort>
[Peer]
PublicKey = <Server PUBLIC KEY>
AllowedIPs = <IP ADRESS THE CLIENT IS ALLOWED TO CONNECT TO GENERALLY ALLOW WHOLE NETWORK>
Endpoint = <SERVER ENPOINT WITH PORT NUMBER>
PersistentKeepalive = <KEEP ALIVE>
```

```
[Interface]
PrivateKey = uL4/ae4Yy70Xs0tgcLTbTY96shxEJhoZHdTDmMGC2mk=
Address = 10.10.2.13/24
ListenPort = 51820
[Peer]
PublicKey = kPzafjh7DRS3+rjd44zM3QdXOAnxp4ykxcFqjUB7s3c=
AllowedIPs = 10.10.2.0/24
Endpoint = 192.168.1.189:51820
PersistentKeepalive = 25
```

#### client public key

```
V32y+8IIuARL810iA/QpeDvdbtGP4GPTNDXkO651vSc=
```

#### client private key

```
uL4/ae4Yy70Xs0tgcLTbTY96shxEJhoZHdTDmMGC2mk=
```

### On Server

#### Server Config

```
[Interface]
Address = <IP ADRESS ASSIGNED TO WIREUARD ON SERVER UNIQUE TO WIREGARD BUT ON NETWORK>
ListenPort = <ListenPort>
PrivateKey = <PRIVATE KEY OF SERVER>

[Peer]
PublicKey = <PULIC KEY OF CLIENT>
AllowedIPs = <IP ADRESS CLIENT IS ALLOWED TO TALK TO GENERALLY WHOLE NETWORK>
```

```
[Interface]
Address = 10.10.2.1/24
ListenPort = 51820
PrivateKey = IAhwBAftzCq22C/qyicqEoyi+mSqGRpFhPGv4BSJf0s=

[Peer]
PublicKey = V32y+8IIuARL810iA/QpeDvdbtGP4GPTNDXkO651vSc=
AllowedIPs = 10.10.2.0/24
```

#### server public key

```
kPzafjh7DRS3+rjd44zM3QdXOAnxp4ykxcFqjUB7s3c=
```

#### server private key

```
IAhwBAftzCq22C/qyicqEoyi+mSqGRpFhPGv4BSJf0s=
```

## References

* https://www.digitalocean.com/community/tutorials/how-to-create-a-point-to-point-vpn-with-wireguard-on-ubuntu-16-04
* Look into this for S2S
  * https://ubuntu.com/server/docs/wireguard-vpn-site-to-site
