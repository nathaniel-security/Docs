# Why SSH Says client\_loop send disconnect Broken pipe

_And How to Kill It...With Extreme Prejudice_

### TL;DR

A “pipe,” in this context, is a metaphor for any data channel most often a network socket for SSH. When you see **client\_loop: send disconnect: Broken pipe**, SSH just tried to send your precious bits to a connection long gone. Fix it with proper keepalive settings. And remember: lost connections shouldn’t tank your automation wrap, retry, and keep calm.

***

**“Broken pipe.”** The error haunts SSH users everywhere. You’re in the middle of a remote job, deep into tweaks or long-running automation, and BOOM session gone. Time to pull back the curtain:

### What Is a “Pipe” (and Why Is It Broken)?

* **Classic Unix Pipe:**
  * A pipe is an in-memory channel one process writes, one reads. Removed in real-time, never hits the disk.
* **Network Sockets (What SSH Actually Uses):**
  * _Pipes made network-aware_: A socket is a two-way stream connecting apps, often over TCP, letting data flow between devices.
* **Under the Hood:**
  * When a program tries to write to a pipe (or socket) and the other end has bailed (closed, crashed, lost the network), Unix gives the process a "broken pipe" (`SIGPIPE`). The next write dead on arrival[^1].

> "**Broken pipe** here is Unix’s way of telling you: ‘I yelled, but nobody’s listening on the other side anymore.’ Not a polite goodbye more like a door slammed shut.”

### The Real Reason SSH Drops the Connection

#### &#x20;Why the “Broken Pipe” Triggers in SSH:

* **Inactivity Timeout:** No commands, no network traffic SSH server closes idle connections to save resources.
* **Network Hiccups:** Packet loss, WiFi flickers, VPN drops, or bad routing? Session gets chopped.
* **Client/Server Keepalive Misconfig:** If either side expects regular “I’m alive!” pings (but doesn’t get them), it’s curtains for your session[^2].
* **Resource Problems & Network Firewall Timeouts**: ISPs or corporate firewalls can kill dormant connections faster than you can say `Ctrl+C`.

### "Broken Pipe" = Socket Graveyard

* **SSH Is Built On Sockets:** The error lingo is legacy; modern SSH is about sockets, not shell pipes, but the OS’s behavior and error messages are the same[^1].
* **Result:** Your SSH client tries to send data but gets the cold shoulder connection is toast.

### Real-World Ripples

* **Lone SSH Client Dies:** The connection is gone, but if you use `tmux` or `screen`, your remote stuff keeps running. Reconnect, reattach, no harm done.
* **Automation Alert:** Pushing files or running scripts on remote servers? _Broken pipe_ can cause partial uploads, failed steps, and very messy ops.

### How to NEVER See "Broken Pipe" Again 🚀

**The fast fix:** Make SSH keep the connection “breathing,” even when you’re AFK.

#### Client-Side Fix

Edit (or create) your `~/.ssh/config`:

```
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 5
```

Or add the options to your SSH call:

```
ssh -o ServerAliveInterval=60 -o ServerAliveCountMax=5 user@host
```

* `ServerAliveInterval`: (seconds) How often the client pings the server[^3].
* `ServerAliveCountMax`: How many pings the client will try before giving up.
  * **Example:** 5 tries × 60sec = 5 minutes of trying before disconnect[^4].

#### Server-Side Fix

With root, in `/etc/ssh/sshd_config`:

```
ClientAliveInterval 60
ClientAliveCountMax 5
```

Then:

```
sudo systemctl restart sshd
```

* `ClientAliveInterval` & `ClientAliveCountMax`: Similar concept, but from the server’s point of view how long it’ll wait and how many times it’ll shout into the void before killing the session[^5].

### For the Hackers, Ops and Defence

* **Automation-Ready:** Always wrap file transfers or infra ops in retry logic, so a “broken pipe” doesn’t break your CI/CD pipeline.
* **Debug Like a Pro:**
  * Use `ssh -vvv` for deep diagnostics.
  * Count how long until the disconnect, pinpoint which timeout is killing you.
* **Cloud/NAT Environments:** Some cloud networks drop _silent_ TCP connections; tune your keepalives to run just under the timeout trigger.





⁂

[^1]: https://unix.stackexchange.com/questions/84813/what-makes-a-unix-process-die-with-broken-pipe

[^2]: https://www.tecmint.com/client\_loop-send-disconnect-broken-pipe/

[^3]: https://hackernoon.com/keeping-ssh-connection-alive-for-longer-durations

[^4]: https://www.baeldung.com/linux/ssh-keep-alive

[^5]: https://johngai.com/2025/04/24/how-to-fix-ssh-client\_loop-send-disconnect-broken-pipe-error/
