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
* Steps to avoid signature based detection are pretty simple:
  * Scan using AMSlTrigger
  * Modify the detected code snippet
  * Rescan using AMSlTrigger
  * Repeat the steps 2 & 3 till we get a result as "AMSI RESULT NOT DETECTED" or "Blank"



### Example

* using powerup
  * Scan using AMSlTrigger
  *

      <figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>


  * need to manually get which word is a problem - it does not give the exact string - needs to be done manually
    * once you know what is the problem find a solution
      * in above example `System.Appomain` was the issue
      * so reverse the strings
      *

          <figure><img src="../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>
