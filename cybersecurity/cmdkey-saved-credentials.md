# Cmdkey Saved Credentials

* The [cmdkey](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/cmdkey) command can be used to create, list, and delete stored usernames and passwords.
* Users may wish to store credentials for a specific host or use it to store credentials for terminal services connections to connect to a remote host using Remote Desktop without needing to enter a password.
* This may help us either move laterally to another system with a different user or escalate privileges on the current host to leverage stored credentials for another user.

```cmd-session
cmdkey /list
```

* We can also attempt to reuse the credentials using `runas` to send ourselves a reverse shell as that user, run a binary, or launch a PowerShell or CMD console with a command such as:

**Run Commands as Another User Windows**

```powershell-session
runas /savecred /user:inlanefreight\bob "COMMAND HERE"
```

* https://github.com/antonioCoco/RunasCs
