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

      <figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>


  * need to manually get which word is a problem - it does not give the exact string - needs to be done manually
    * once you know what is the problem find a solution
      * in above example `System.Appomain` was the issue
      * so reverse the strings
      *

          <figure><img src="../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>





### Ideas for Bypassing

* Remove default comments.
* Rename the script, function names and variables.
* Modify the variable names of the Win32 API calls that are detected.
* Obfuscate PEBytes content PowerKatz dll using packers.
  * if another tool is used
    * ie daisy chaining
      * you might need to rework that tool
      * https://github.com/mgeeky/ProtectMyTooling
* Implement a reverse function for Bytes to avoid any static signatures.
  * example could be convert the DLL to base 64 and reverse it
* Add a sandbox check to waste dynamic analysis resources.
  *

      <figure><img src="../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>


  * add other DLL to check into the code
    * intentions is to waste time&#x20;
    * also gets to know if your code is executed in a sandbox&#x20;
      * in this case we are looking for vmware and virtual box dll&#x20;
        * since a sandbox will not contain these dll
  * Remove Reflective warnings for a clean output.
    * Use obfuscated commands for example in Invoke-MimiEx execution.
      * `sekurlsa::ekeys` can be broken down to
      *

          <figure><img src="../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>
  * Analysis using DefenderCheck.

