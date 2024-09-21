# Bypassing AV Signatures for PowerShell

* can use the AMSlTrigger
  * https://github.com/RythmStick/AMSITrigger
    * tool to identify the exact part of a script that is detected.
  * https://github.com/matterpreter/DefenderCheck/tree/master
    * https://github.com/t3hbb/DefenderCheck/tree/master
    * identify code and strings from a binary / file that Windows Defender may flag.
  * Simply provide path to the script file to scan it:

```
AmsiTrigger_x64. exe -i powerup.ps1
```

```
DefenderCheck.exe powerup.ps1
```

* For full obfuscation of PowerShell scripts, see
  * https://github.com/danielbohannon/Invoke-Obfuscation
