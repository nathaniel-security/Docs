---
description: docker management tool
---

# Portainer

- Good Docker Management tool 
- is opensource

## Update
```bash
sudo apt update && sudo apt upgrade -y
```

## Install Portainer
```bash
mkdir portainer_data
current_path=$(pwd)
sudo docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v $current_path/portainer_data:/data portainer/portainer-ce:latest
```