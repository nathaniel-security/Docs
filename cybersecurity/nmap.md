# Nmap

### Notes

* Passing an ip is better than using domain name
  * else nmap will spend time doing a reverse domain lookup
  * but an ip can also have multiple domain under it
    * which could be used to attack it
* if you know target is online pass the flag for telling it not to do a ping
  * Pass the `-Pn` flag
* Nmap used the response of port scan as well as the network header to determine the OS
  * this is a probability
  * this can be increased by various NSE scripts
* Suggestion
  * Always save it to a file

## Nmap Commands

### Basic

```
nmap -Pn -n -A -vvv -oN nmap/00-basic.txt -iL target.txt
```

### TCP port scan Full

```bash
nmap -n -p- -vvv -oN nmap/01-full-port-scan.txt -iL target.txt
```

### UDP port scan Full

```bash
sudo nmap -vvv -sU -n -p- -oN nmap/02-udp-full-port-scan.txt -iL target.txt
```

### Top 1000 UDP port scan Full

```bash
sudo nmap -vvv -sU -n -p- -oN nmap/02.5-udp-top-1000-port-scan.txt -iL target.txt
```

### Detail scan

```bash
sudo nmap -vvv -sC -A -O -osscan-guess -sV -version-all -oN nmap/03-detail-scan.txt -iL target.txt -p <port number T:80 t is tcp>
```

### Convert nmap xml to html

```shell-session
 xsltproc target.xml -o target.html
```

### Type of Scans

#### -sT

* most stealthy where the whole connection is made
  * most stealthy since many firewalls are tuned to look for half open connections
* also the most accurate
*

## Cheat sheet

| **Nmap Option**      | **Description**                                                        |
| -------------------- | ---------------------------------------------------------------------- |
| `10.10.10.0/24`      | Target network range.                                                  |
| `-sn`                | Disables port scanning.                                                |
| `-Pn`                | Disables ICMP Echo Requests                                            |
| `-n`                 | Disables DNS Resolution.                                               |
| `-PE`                | Performs the ping scan by using ICMP Echo Requests against the target. |
| `--packet-trace`     | Shows all packets sent and received.                                   |
| `--reason`           | Displays the reason for a specific result.                             |
| `--disable-arp-ping` | Disables ARP Ping Requests.                                            |
| `--top-ports=<num>`  | Scans the specified top ports that have been defined as most frequent. |
| `-p-`                | Scan all ports.                                                        |
| `-p22-110`           | Scan all ports between 22 and 110.                                     |
| `-p22,25`            | Scans only the specified ports 22 and 25.                              |
| `-F`                 | Scans top 100 ports.                                                   |
| `-sS`                | Performs an TCP SYN-Scan.                                              |
| `-sA`                | Performs an TCP ACK-Scan.                                              |
| `-sU`                | Performs an UDP Scan.                                                  |
| `-sV`                | Scans the discovered services for their versions.                      |
| `-sC`                | Perform a Script Scan with scripts that are categorized as "default".  |
| `--script <script>`  | Performs a Script Scan by using the specified scripts.                 |
| `-O`                 | Performs an OS Detection Scan to determine the OS of the target.       |
| `-A`                 | Performs OS Detection, Service Detection, and traceroute scans.        |
| `-D RND:5`           | Sets the number of random Decoys that will be used to scan the target. |
| `-e`                 | Specifies the network interface that is used for the scan.             |
| `-S 10.10.10.200`    | Specifies the source IP address for the scan.                          |
| `-g`                 | Specifies the source port for the scan.                                |
| `--dns-server <ns>`  | DNS resolution is performed by using a specified name server.          |

### Output Options

| **Nmap Option** | **Description**                                                                   |
| --------------- | --------------------------------------------------------------------------------- |
| `-oA filename`  | Stores the results in all available formats starting with the name of "filename". |
| `-oN filename`  | Stores the results in normal format with the name "filename".                     |
| `-oG filename`  | Stores the results in "grepable" format with the name of "filename".              |
| `-oX filename`  | Stores the results in XML format with the name of "filename".                     |

### Performance Options

| **Nmap Option**              | **Description**                                              |
| ---------------------------- | ------------------------------------------------------------ |
| `--max-retries <num>`        | Sets the number of retries for scans of specific ports.      |
| `--stats-every=5s`           | Displays scan's status every 5 seconds.                      |
| `-v/-vv`                     | Displays verbose output during the scan.                     |
| `--initial-rtt-timeout 50ms` | Sets the specified time value as initial RTT timeout.        |
| `--max-rtt-timeout 100ms`    | Sets the specified time value as maximum RTT timeout.        |
| `--min-rate 300`             | Sets the number of packets that will be sent simultaneously. |
| `-T <0-5>`                   | Specifies the specific timing template.                      |

### Types of Port States

| **State**          | **Description**                                                                                                                                                                                         |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `open`             | This indicates that the connection to the scanned port has been established. These connections can be **TCP connections**, **UDP datagrams** as well as **SCTP associations**.                          |
| `closed`           | When the port is shown as closed, the TCP protocol indicates that the packet we received back contains an `RST` flag. This scanning method can also be used to determine if our target is alive or not. |
| `filtered`         | Nmap cannot correctly identify whether the scanned port is open or closed because either no response is returned from the target for the port or we get an error code from the target.                  |
| `unfiltered`       | This state of a port only occurs during the **TCP-ACK** scan and means that the port is accessible, but it cannot be determined whether it is open or closed.                                           |
| `open\|filtered`   | If we do not get a response for a specific port, `Nmap` will set it to that state. This indicates that a firewall or packet filter may protect the port.                                                |
| `closed\|filtered` | This state only occurs in the **IP ID idle** scans and indicates that it was impossible to determine if the scanned port is closed or filtered by a firewall.                                           |

## Firewall and IDS/IPS Evasion

* think about the kind of \[\[Firewalls]] that would be in place

**Determine Firewalls and Their Rules**

* This is different for `rejected` packets that are returned with an `RST` flag. These packets contain different types of ICMP error codes or contain nothing at all.
  * Such errors can be:
    * Net Unreachable
    * Net Prohibited
    * Host Unreachable
    * Host Prohibited
    * Port Unreachable
    * Proto Unreachable
* Nmap's TCP ACK scan (`-sA`) method is much harder to filter for firewalls and IDS/IPS systems than regular SYN (`-sS`) or Connect scans (`-sT`)
  * because they only send a TCP packet with only the `ACK` flag
  * When a port is closed or open, the host must respond with an `RST` flag.
  * Unlike outgoing connections, all connection attempts (with the `SYN` flag) from external networks are usually blocked by firewalls.
  * However, the packets with the `ACK` flag are often passed by the firewall because the firewall cannot determine whether the connection was first established from the external network or the internal network.
  * works if the firewall is stateless

### Detect IDS/IPS

* Several virtual private servers (`VPS`) with different IP addresses are recommended to determine whether such systems are on the target network during a penetration test

### Scan by Using Decoys

```shell-session
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```

| **Scanning Options** | **Description**                                                                            |
| -------------------- | ------------------------------------------------------------------------------------------ |
| `10.129.2.28`        | Scans the specified target.                                                                |
| `-p 80`              | Scans only the specified ports.                                                            |
| `-sS`                | Performs SYN scan on specified ports.                                                      |
| `-Pn`                | Disables ICMP Echo requests.                                                               |
| `-n`                 | Disables DNS resolution.                                                                   |
| `--disable-arp-ping` | Disables ARP ping.                                                                         |
| `--packet-trace`     | Shows all packets sent and received.                                                       |
| `-D RND:5`           | Generates five random IP addresses that indicates the source IP the connection comes from. |

### SYN-Scan From DNS Port

```shell-session
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```

## Reference

* [https://man7.org/linux/man-pages/man1/nping.1.html](https://man7.org/linux/man-pages/man1/nping.1.html)
* [https://nmap.org/nsedoc/scripts/](https://nmap.org/nsedoc/scripts/)
* [https://securitytrails.com/blog/nmap-commands](https://securitytrails.com/blog/nmap-commands)
* [Eagle Link](eagle://item/LQAUAF58723C8)
  * [https://www.stationx.net/nmap-cheat-sheet/](https://www.stationx.net/nmap-cheat-sheet/)
  * [https://stationx-public-download.s3.us-west-2.amazonaws.com/nmap\_cheet\_sheet\_v7.pdf](https://stationx-public-download.s3.us-west-2.amazonaws.com/nmap\_cheet\_sheet\_v7.pdf)
* [https://www.youtube.com/watch?v=JHAMj2vN2oU](https://www.youtube.com/watch?v=JHAMj2vN2oU)
* [https://www.youtube.com/watch?v=5MTZdN9TEO4\&list=PLBf0hzazHTGM8V\_3OEKhvCM9Xah3qDdIx](https://www.youtube.com/watch?v=5MTZdN9TEO4\&list=PLBf0hzazHTGM8V\_3OEKhvCM9Xah3qDdIx)