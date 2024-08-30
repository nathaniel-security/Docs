# windapsearch

### Gathering Users with LDAP Anonymous

```shell-session
./windapsearch.py --dc-ip 172.16.5.5 -u "" -U
```

* we can specify anonymous access by providing a blank username with the `-u` flag and the `-U` flag to tell the tool to retrieve just users.

```shell-session
[+] No username provided. Will try anonymous bind.
[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]	...success! Binded as: 
[+]	 None

[+] Enumerating all AD users
[+]	Found 2906 users: 

cn: Guest

cn: Htb Student
userPrincipalName: htb-student@inlanefreight.local

cn: Annie Vazquez
userPrincipalName: avazquez@inlanefreight.local
```

## Reference

* [https://github.com/ropnop/windapsearch?tab=readme-ov-file](https://github.com/ropnop/windapsearch?tab=readme-ov-file)
