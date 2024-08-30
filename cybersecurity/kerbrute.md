# Kerbrute

* Kerbrute can be a stealthier option for domain account enumeration.
* It takes advantage of the fact that Kerberos pre-authentication failures often will not trigger logs or alerts

### Cloning Kerbrute GitHub Repo

```shell-session
sudo git clone https://github.com/ropnop/kerbrute.git
```

#### Install

```
cd kerbrute
```

```shell-session
make help
```

* We can choose to compile just one binary or type `make all` and compile one each for use on Linux, Windows, and Mac systems (an x86 and x64 version for each).

```
sudo make all
```

* The newly created `dist` directory will contain our compiled binaries.

```shell-session
ls dist/
```

### Testing the kerbrute\_linux\_amd64 Binary

```shell-session
/kerbrute_linux_amd64 
```

#### Adding the Tool to our Path

```shell-session
echo $PATH
```

```shell-session
/home/htb-student/.local/bin:/snap/bin:/usr/sandbox/:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/usr/share/games:/usr/local/sbin:/usr/sbin:/sbin:/snap/bin:/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/home/htb-student/.dotnet/tools
```

**Moving the Binary**

```shell-session
sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute
```

### Enumerating Users with Kerbrute

* look into [statistically-likely-usernames.md](statistically-likely-usernames.md "mention") for username list
  * [linkedin2username.md](linkedin2username.md "mention")
  * jsmith.txt is from this list

```shell-session
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt 
```

* &#x20;Using Kerbrute for username enumeration will generate event ID [4768: A Kerberos authentication ticket (TGT) was requested](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4768)
  * will be triggered if [Kerberos event logging](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/enable-kerberos-event-logging) is enabled via Group Policy

### Password Spraying Active Directory

* need valid\_users.txt file
  * use [password-spraying-making-a-target-user-list-active-directory.md](password-spraying-making-a-target-user-list-active-directory.md "mention")

```shell-session
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1
```

## Reference

* [https://github.com/ropnop/kerbrute](https://github.com/ropnop/kerbrute)
