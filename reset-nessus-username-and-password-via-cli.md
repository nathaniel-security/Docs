# Reset Nessus Username & Password via CLI

Locked out of Nessus? Forget clicking around in the Web UI just drop to the CLI and you can reset, recover, or create accounts in seconds. Fast, scriptable, and way cleaner.

***

### 1. List All Nessus Users

Want to see who exists on your Nessus box? Run:

```bash
sudo /opt/nessus/sbin/nessuscli lsuser
```

This will dump all registered usernames to your terminal. No digging through configs, no guesswork.

***

### 2. Reset Any Nessus User’s Password

Got the username? Reset it like this:

```bash
sudo /opt/nessus/sbin/nessuscli chpasswd <username>
```

You’ll be prompted to enter the new password twice. That’s it—the account is back in action, no GUI involved.

***

### 3. Forgot the Username? Create a New User

If `lsuser` gives you nothing or you’d rather spin up a new account:

```bash
sudo /opt/nessus/sbin/nessuscli adduser
```

Step through the prompts, assign admin rights if needed, and you’re instantly back in control.

***

### Finally

* **Shortcut Trick**: Symlink `nessuscli` into `/usr/local/bin` so you don’t have to type the full path every time.

```bash
sudo ln -s /opt/nessus/sbin/nessuscli /usr/local/bin/nessuscli
```

