# 🔥 HTB: Traverxec – From RCE to Root with a Nostalgic Bang

<figure><img src="../.gitbook/assets/ChatGPT Image Jun 7, 2025, 08_39_54 AM.png" alt="" width="375"><figcaption></figcaption></figure>

{% hint style="info" %}
**“Give me 3 minutes and I’ll show you how a misconfigured web server handed me SSH keys on a silver platter.”**
{% endhint %}

### 🧠 Reconnaissance

We kick things off with a good old `nmap` scan:

```
nmap -Pn -A -p- 10.10.10.165
```

```
22/tcp open  ssh     OpenSSH 7.9p1 Debian
80/tcp open  http    nostromo 1.9.6
```

### 🚀 Initial Foothold – CVE-2019-16278

A quick search reveals [CVE-2019-16278](https://github.com/AnubisSec/CVE-2019-16278), a remote code execution vulnerability in `nostromo 1.9.6`.

I used a simple Python script to exploit it and got a remote shell as `www-data`.

```
python nostroSploit.py 10.10.10.165 80 "id"
```

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 🛠️ Post-Exploitation – Looting Configs

Once inside, I poked around

* Quick Google and `nostromo` has conf stored in `/var/nostromo/conf/nhttpd.conf`

```
serverroot		/var/nostromo
homedirs		/home
homedirs_public		public_www
```

That `public_www` bit? Jackpot.

I couldn't list `/home/david` directly due to `drwx--x--x` permissions, but `/home/david/public_www` was accessible.

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* this is cause of the permission i have on the dir

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* On David's home dir, i have `drwx--x--x` in order for me to read the content of the directory i need read permission, which would mean `drwx--xr-x`&#x20;

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



Inside, I found a zipped SSH key bundle. Extracted it, and then:

```
ssh -o IdentitiesOnly=yes -i id_rsa david@10.10.10.165
```

* crack the password with john
*

    <figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>



### 🧗 Privilege Escalation – From David to Root

As `david`, I spotted a custom script directory in his home: `/home/david/bin/`.

<figure><img src="../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

#### ⚔️ The Root Strike

`journalctl` can spawn a shell by executing:

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

And with that…\
**Root. Owned. Game over.**

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

