# 😤 I Had Write Permissions... So Why Was Linux Saying "No"?

<figure><img src="../.gitbook/assets/ChatGPT Image Jun 6, 2025, 11_30_05 PM.png" alt=""><figcaption></figcaption></figure>

> 🧠 “If I have write access to a file, I should be able to write to it… right?”
>
> That’s what I thought. Until Linux roundhouse kicked me in the brain.

### 🎯 The Setup — HTB: Traverxec

I was knee-deep in the **Traverxec** box on Hack The Box. You know, that one with some misconfigured `nostromo` web server and a classic privilege escalation path. Things were going smoothly. Too smooth.

I landed a shell as `david`. Did some recon. Crawled into the `/home/david/bin` directory — jackpot.

I found a file:

```
server-stats.sh
```

Looked promising. Maybe some crontab magic? Maybe I can inject a command and get root? Let’s gooo.

I checked permissions:

```
-rwx------ 1 david david 363 Oct 25  2019 server-stats.sh
```

Full permissions. Owned by me. Hell yeah.

I tried a quick test:

```
echo "echo pwned" > server-stats.sh
```

💥 BOOM.\
No wait. Not boom. **Smack.**

```
-bash: server-stats.sh: Operation not permitted
```

### 🧪 Sanity Check

Hold up. I checked again:

* ✅ File owned by `david`
* ✅ `rwx` permissions
* ✅ I'm logged in as `david`
* ❌ Can't write, rename, delete, chmod, or even `echo` into it

It felt like I was trying to write to a ghost.

### 🔍 The Sherlock Moment (aka: Googling Like a Madman)

At this point, I was questioning everything. My permissions were correct. I owned the file. The command syntax was fine. So why the hell was Linux stonewalling me?

So, I did what any respectable hacker does in a crisis:\
**I Googled. Hard.**

And after sifting through a dozen StackOverflow posts, some outdated forums, and a Reddit thread titled _“WTF is this i thing in lsattr?”_ — I had my answer.

I found out about `lsattr`, a command that shows **file attributes at the filesystem level**, not just the usual `ls -la` stuff.

I ran:

```
lsattr server-stats.sh
```

And there it was, hiding in plain sight:

```
----i---------e---- server-stats.sh
```

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

That **`i`** stands for **immutable**.

This little gremlin of a flag meant the file was **locked down so hard** that even root couldn’t touch it without removing the bit first.

It was like Linux was saying:

> "Oh, you thought you were in charge? That’s cute."
