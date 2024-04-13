---
description: basic bash commands used in cybersecurity
---

# Basic Bash

## SET TERM

```bash
export TERM=xterm
```

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

## Local Python Server

```bash
python3 -m http.server <PORT NUMBER> #python3 -m http.server 80
```

## NETCAT

### Netcat Listener

```bash
nc -lvp <PORT NUMBER> #nc -lvp 8080
```
