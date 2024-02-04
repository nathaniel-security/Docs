---
description: Install Docker Ubuntu
cover: >-
  https://images.unsplash.com/photo-1605745341112-85968b19335b?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwxfHxkb2NrZXJ8ZW58MHx8fHwxNzA3MDIwNjYxfDA&ixlib=rb-4.0.3&q=85
coverY: 0
---

# Install Docker Ubuntu

## Install docker on ubuntu

```bash
 sudo apt-get update
```

```bash
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```

```bash
sudo apt install docker-ce -y
```

```bash
sudo systemctl status docker
```
