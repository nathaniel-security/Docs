# Windows Event Log Readers

* Organizations may enable logging of process command lines to help defenders monitor and identify possibly malicious behaviour and identify binaries that should not be present on a system
* The tools would then flag any potentially malicious activity, such as the `whoami`, `netstat`, and `tasklist` commands being run from a marketing executive's workstation.
* [microsoft-guide-to-all-windows-command.md](microsoft-guide-to-all-windows-command.md "mention")
* Many Windows commands support passing a password as a parameter,
  * if auditing of process command lines is enabled,
  * this sensitive information will be captured.
  * We can query Windows events from the command line using the [wevtutil](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/wevtutil) utility and the [Get-WinEvent](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.1) PowerShell cmdlet.

### Confirming Group Membership

```cmd-session
net localgroup "Event Log Readers"
```

```cmd-session
Alias name     Event Log Readers
Comment        Members of this group can read event logs from local machine

Members

-------------------------------------------------------------------------------
logger
The command completed successfully.
```

### Searching Security Logs Using wevtutil

```powershell-session
wevtutil qe Security /rd:true /f:text | Select-String "/user"
```

* We can also specify alternate credentials for `wevtutil` using the parameters `/u` and `/p`.

### Passing Credentials to wevtutil

```cmd-session
wevtutil qe Security /rd:true /f:text /r:share01 /u:julie.clay /p:Welcome1 | findstr "/user"
```

* For `Get-WinEvent`, the syntax is as follows. In this example, we filter for process creation events (4688), which contain `/user` in the process command line.
* **Note: Searching the `Security` event log with `Get-WInEvent` requires administrator access or permissions adjusted on the registry key `HKLM\System\CurrentControlSet\Services\Eventlog\Security`. Membership in just the `Event Log Readers` group is not sufficient.**

### Searching Security Logs Using Get-WinEvent

```powershell-session
Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}
```

* The cmdlet can also be run as another user with the `-Credential` parameter.
