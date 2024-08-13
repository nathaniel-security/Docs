# Hashcat

```
hashcat --user users.txt /usr/share/wordlists/rockyou.txt -m 3200
```

* users.txt

```
lewis:$2y$10$6V52x.SD8Xc7hNlVwUTrI.ax4BIAYuhVBMVvnYWRceBmy8XdEzm1u
logan:$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12
```

### Default Credentials

* [Default Credentials](app://obsidian.md/Default%20Passwords)

### Generating Rule-based Wordlist

| **Function** | **Description**                                   |
| ------------ | ------------------------------------------------- |
| `:`          | Do nothing.                                       |
| `l`          | Lowercase all letters.                            |
| `u`          | Uppercase all letters.                            |
| `c`          | Capitalize the first letter and lowercase others. |
| `sXY`        | Replace all instances of X with Y.                |
| `$!`         | Add the exclamation character at the end.         |

* Full Rule set
  * [https://hashcat.net/wiki/doku.php?id=rule\_based\_attack](https://hashcat.net/wiki/doku.php?id=rule\_based\_attack)
* less custom.rule

```custom.rule
:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@

```

```bash
hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```

**Hashcat Existing Rules**

```bash
ls /usr/share/hashcat/rules/
```

* [creating-custom-wordlists.md](creating-custom-wordlists.md "mention")

### Attacks

#### Cracking the NT Hash with Hashcat

```
sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

#### Hashcat - Cracking Unshadowed Hashes

```bash
 hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

#### Hashcat - Cracking MD5 Hashes

```bash
cat md5-hashes.list
```

```bash
 hashcat -m 500 -a 0 md5-hashes.list rockyou.txt
```
