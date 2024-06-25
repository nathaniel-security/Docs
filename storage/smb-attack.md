# SMB Attack

* SMB and SAMBA are different
* SMB was made by IBM and adopted into windows
* SAMBA uses the Common Internet File System (CIFS) to talk to SMB
  * CIFS is a very specific implementation of the SMB protocol
  * This allows Samba to communicate with newer Windows systems
  * it usually is referred to as `SMB / CIFS`.
* IBM developed an `application programming interface` (`API`) for networking computers called the `Network Basic Input/Output System` (`NetBIOS`).
  * The NetBIOS API provided a blueprint for an application to connect and share data with other computers.
  * In a NetBIOS environment, when a machine goes online, it needs a name, which is done through the so-called `name registration` procedure.
  * Either each host reserves its hostname on the network, or the [NetBIOS Name Server](https://networkencyclopedia.com/netbios-name-server-nbns/) (`NBNS`) is used for this purpose.
  * It also has been enhanced to [Windows Internet Name Service](https://networkencyclopedia.com/windows-internet-name-service-wins/) (`WINS`).
* &#x20;**TCP port 139 or 445** SMB over TCP
  * main
* **UDP 138**  SMB over UDP (datagram).
* **UDP 137**  SMB over user datagram protocol (UDP or Name Services).

### Default Configuration

```shell-session
cat /etc/samba/smb.conf | grep -v "#\|\;" 

```

```
[global]
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d

   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes

   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .

   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes

[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700

[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```

| **Setting**                    | **Description**                                                       |
| ------------------------------ | --------------------------------------------------------------------- |
| `[sharename]`                  | The name of the network share.                                        |
| `workgroup = WORKGROUP/DOMAIN` | Workgroup that will appear when clients query.                        |
| `path = /path/here/`           | The directory to which user is to be given access.                    |
| `server string = STRING`       | The string that will show up when a connection is initiated.          |
| `unix password sync = yes`     | Synchronize the UNIX password with the SMB password?                  |
| `usershare allow guests = yes` | Allow non-authenticated users to access defined share?                |
| `map to guest = bad user`      | What to do when a user login request doesn't match a valid UNIX user? |
| `browseable = yes`             | Should this share be shown in the list of available shares?           |
| `guest ok = yes`               | Allow connecting to the service without using a password?             |
| `read only = yes`              | Allow users to read files only?                                       |
| `create mask = 0700`           | What permissions need to be set for newly created files?              |
|                                |                                                                       |

#### Dangerous Settings

| **Setting**                | **Description**                                                     |
| -------------------------- | ------------------------------------------------------------------- |
| `browseable = yes`         | Allow listing available shares in the current share?                |
| `read only = no`           | Forbid the creation and modification of files?                      |
| `writable = yes`           | Allow users to create and modify files?                             |
| `guest ok = yes`           | Allow connecting to the service without using a password?           |
| `enable privileges = yes`  | Honor privileges assigned to specific [SID](app://obsidian.md/SID)? |
| `create mask = 0777`       | What permissions must be assigned to the newly created files?       |
| `directory mask = 0777`    | What permissions must be assigned to the newly created directories? |
| `logon script = script.sh` | What script needs to be executed on the user's login?               |
| `magic script = script.sh` | Which script should be executed when the script gets closed?        |

### Ways of interacting with SMB

#### SMbclient

```
smbclient -L \\\\10.10.11.174\\
```

* \-L to List the shares

```
smbclient -N //10.10.10.123/general
```

* \-N for `null session`
  * which is `anonymous` access without the input of existing users or valid passwords.

```
smbclient //10.10.10.123/general
```

* guest login is different from authenticated login and is also different from Anonymous login

**!ls**

* !ls for listing on local directory (not smb)

```shell-session
!ls
```

### Attacks

#### rpcclient

```shell-session
 rpcclient -U "" 10.129.14.128
```

| **Query**                 | **Description**                                                    |
| ------------------------- | ------------------------------------------------------------------ |
| `srvinfo`                 | Server information.                                                |
| `enumdomains`             | Enumerate all domains that are deployed in the network.            |
| `querydominfo`            | Provides domain, server, and user information of deployed domains. |
| `netshareenumall`         | Enumerates all available shares.                                   |
| `netsharegetinfo <share>` | Provides information about a specific share.                       |
| `enumdomusers`            | Enumerates all domain users.                                       |
| `queryuser <RID>`         | Provides information about a specific user.                        |

**RPCclient - Enumeration**

```shell-session
rpcclient $> srvinfo
```

```shell-session
rpcclient $> enumdomains
```

```shell-session
rpcclient $> querydominfo
```

```shell-session
rpcclient $> netshareenumall
```

```shell-session
rpcclient $> netsharegetinfo notes
```

**Rpcclient - User Enumeration**

```shell-session
rpcclient $> enumdomusers
```

```shell-session
rpcclient $> enumdomusers

user:[mrb3n] rid:[0x3e8]
user:[cry0l1t3] rid:[0x3e9]


rpcclient $> queryuser 0x3e9

        User Name   :   cry0l1t3
        Full Name   :   cry0l1t3
        Home Drive  :   \\devsmb\cry0l1t3
        Dir Drive   :
        Profile Path:   \\devsmb\cry0l1t3\profile
        Logon Script:
        Description :
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Do, 01 Jan 1970 01:00:00 CET
        Logoff Time              :      Mi, 06 Feb 2036 16:06:39 CET
        Kickoff Time             :      Mi, 06 Feb 2036 16:06:39 CET
        Password last set Time   :      Mi, 22 Sep 2021 17:50:56 CEST
        Password can change Time :      Mi, 22 Sep 2021 17:50:56 CEST
        Password must change Time:      Do, 14 Sep 30828 04:48:05 CEST
        unknown_2[0..31]...
        user_rid :      0x3e9
        group_rid:      0x201
        acb_info :      0x00000014
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x00000000
        padding1[0..7]...
        logon_hrs[0..21]...


rpcclient $> queryuser 0x3e8

        User Name   :   mrb3n
        Full Name   :
        Home Drive  :   \\devsmb\mrb3n
        Dir Drive   :
        Profile Path:   \\devsmb\mrb3n\profile
        Logon Script:
        Description :
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Do, 01 Jan 1970 01:00:00 CET
        Logoff Time              :      Mi, 06 Feb 2036 16:06:39 CET
        Kickoff Time             :      Mi, 06 Feb 2036 16:06:39 CET
        Password last set Time   :      Mi, 22 Sep 2021 17:47:59 CEST
        Password can change Time :      Mi, 22 Sep 2021 17:47:59 CEST
        Password must change Time:      Do, 14 Sep 30828 04:48:05 CEST
        unknown_2[0..31]...
        user_rid :      0x3e8
        group_rid:      0x201
        acb_info :      0x00000010
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x00000000
        padding1[0..7]...
        logon_hrs[0..21]...
```

**Rpcclient - Group Information**

```shell-session
rpcclient $> querygroup 0x201

        Group Name:     None
        Description:    Ordinary Users
        Group Attribute:7
        Num Members:2
```

**Brute Forcing User RIDs**

```shell-session
for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done
```

```
User Name   :   sambauser
user_rid :      0x1f5
group_rid:      0x201

User Name   :   mrb3n
user_rid :      0x3e8
group_rid:      0x201

User Name   :   cry0l1t3
user_rid :      0x3e9
group_rid:      0x201
```

#### impacket

**Impacket - Samrdump.py**

```shell-session
cyberaestheticgroot@htb[/htb]$ samrdump.py 10.129.14.128
```

```shell-session
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Retrieving endpoint list from 10.129.14.128
Found domain(s):
 . DEVSMB
 . Builtin
[*] Looking up users in domain DEVSMB
Found user: mrb3n, uid = 1000
Found user: cry0l1t3, uid = 1001
mrb3n (1000)/FullName: 
mrb3n (1000)/UserComment: 
mrb3n (1000)/PrimaryGroupId: 513
mrb3n (1000)/BadPasswordCount: 0
mrb3n (1000)/LogonCount: 0
mrb3n (1000)/PasswordLastSet: 2021-09-22 17:47:59
mrb3n (1000)/PasswordDoesNotExpire: False
mrb3n (1000)/AccountIsDisabled: False
mrb3n (1000)/ScriptPath: 
cry0l1t3 (1001)/FullName: cry0l1t3
cry0l1t3 (1001)/UserComment: 
cry0l1t3 (1001)/PrimaryGroupId: 513
cry0l1t3 (1001)/BadPasswordCount: 0
cry0l1t3 (1001)/LogonCount: 0
cry0l1t3 (1001)/PasswordLastSet: 2021-09-22 17:50:56
cry0l1t3 (1001)/PasswordDoesNotExpire: False
cry0l1t3 (1001)/AccountIsDisabled: False
cry0l1t3 (1001)/ScriptPath: 
[*] Received 2 entries.
```

#### crackmapexec

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

#### SMBmap

```
smbmap -H 10.10.10.123 
```

**With creds**

```
smbmap -H 10.10.10.123 -u admin -p 'WORKWORKHhallelujah@#'
```

#### Nmap

```shell-session
sudo nmap 10.129.14.128 -sV -sC -p139,445
```

#### Enum4Linux-ng

**Installation**

```shell-session
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt
```

**Enumeration**

```shell-session
./enum4linux-ng.py 10.129.14.128 -A
```
