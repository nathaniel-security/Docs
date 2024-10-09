# Attacking Thick Client Applications

### Introduction

* Thick client applications are the applications that are installed locally on our computers
* &#x20;Unlike thin client applications that run on a remote server and can be accessed through the web browser, these applications do not require internet access to run, and they perform better in processing power, memory, and storage capacity
* &#x20;Thick client applications are usually applications used in enterprise environments created to serve specific purposes
* Such applications include project management systems, customer relationship management systems, inventory management tools, and other productivity software.
* These applications are usually developed using Java, C++, .NET, or Microsoft Silverlight.
* Thick client applications can be categorized into two-tier and three-tier architecture

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Information Gathering

* penetration testers have to identify the application architecture, the programming languages and frameworks that have been used, and understand how the application and the infrastructure work
* The following tools will help us gather information.
  * CFF Explorer
  * Detect It Easy
  * Process Monitor
  * Strings

### Client Side attacks

&#x20;\- we can reverse-engineer and examine .NET and Java applications including EXE, DLL, JAR, CLASS, WAR, and other file formats.  - Dynamic analysis should also be performed in this step, as thick client applications store sensitive information in the memory as well.  - [Ghidra](https://www.ghidra-sre.org/)  - [IDA](https://hex-rays.com/ida-pro/)  - [OllyDbg](http://www.ollydbg.de/)  - [Radare2](https://www.radare.org/r/index.html)  - [dnSpy](https://github.com/dnSpy/dnSpy)  - [x64dbg](https://x64dbg.com/)  - [JADX](https://github.com/skylot/jadx)  - [Frida](https://frida.re/)

### Network Side Attacks

* Wireshark
* tcpdump
* TCPView
* Burp Suite

### Server Side Attacks

* Server-side attacks in thick client applications are similar to web application attacks, and penetration testers should pay attention to the most common ones including most of the OWASP Top Ten.

### Retrieving hardcoded Credentials from Thick-Client Applications

* Scenario is you have a exe file
* Using  `ProcMon64` from [SysInternals](https://learn.microsoft.com/en-gb/sysinternals/downloads/procmon) and monitoring the process reveals that the executable indeed creates a temp file in `C:\Users\Matt\AppData\Local\Temp`.

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* In order to capture the files, it is required to change the permissions of the `Temp` folder to disallow file deletions.
* To do this,
  * we right-click the folder&#x20;
    * `C:\Users\Matt\AppData\Local\Temp`&#x20;
    * and under `Properties` -> `Security` -> `Advanced` -> `cybervaca` -> `Disable inheritance` -> `Convert inherited permissions into explicit permissions on this object` -> `Edit` -> `Show advanced permissions`,
    * we deselect the `Delete subfolders and files`, and `Delete` checkboxes.
    *

        <figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>


    * Finally, we click `OK` -> `Apply` -> `OK` -> `OK` on the open windows.&#x20;
    * Once the folder permissions have been applied we simply run again the `Restart-OracleService.exe` and check the `temp` folder.&#x20;
    * The file `6F39.bat` is created under the `C:\Users\cybervaca\AppData\Local\Temp\2`.&#x20;
    * The names of the generated files are random every time the service is running.
*

    ```cmd-session
    dir C:\Users\cybervaca\AppData\Local\Temp\2
    ```

    ```cmd-session
    ...SNIP...
    04/03/2023  02:09 PM         1,730,212 6F39.bat
    04/03/2023  02:09 PM                 0 6F39.tmp
    ```

    * Listing the content of the `6F39` batch file reveals the following.

    ```batch
    @shift /0
    @echo off

    if %username% == matt goto correcto
    if %username% == frankytech goto correcto
    if %username% == ev4si0n goto correcto
    goto error

    :correcto
    echo TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA > c:\programdata\oracle.txt
    echo AAAAAAAAAAgAAAAA4fug4AtAnNIbgBTM0hVGhpcyBwcm9ncmFtIGNhbm5vdCBiZSBydW4g >> c:\programdata\oracle.txt
    <SNIP>
    echo AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA >> c:\programdata\oracle.txt

    echo $salida = $null; $fichero = (Get-Content C:\ProgramData\oracle.txt) ; foreach ($linea in $fichero) {$salida += $linea }; $salida = $salida.Replace(" ",""); [System.IO.File]::WriteAllBytes("c:\programdata\restart-service.exe", [System.Convert]::FromBase64String($salida)) > c:\programdata\monta.ps1
    powershell.exe -exec bypass -file c:\programdata\monta.ps1
    del c:\programdata\monta.ps1
    del c:\programdata\oracle.txt
    c:\programdata\restart-service.exe
    del c:\programdata\restart-service.exe
    ```

    * Inspecting the content of the file reveals that two files are being dropped by the batch file and being deleted before anyone can get access to the leftovers.
    * We can try to retrieve the content of the 2 files, by modifying the batch script and removing the deletion.

    ```batch
    @shift /0
    @echo off

    echo TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA > c:\programdata\oracle.txt
    echo AAAAAAAAAAgAAAAA4fug4AtAnNIbgBTM0hVGhpcyBwcm9ncmFtIGNhbm5vdCBiZSBydW4g >> c:\programdata\oracle.txt
    <SNIP>
    echo AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA >> c:\programdata\oracle.txt

    echo $salida = $null; $fichero = (Get-Content C:\ProgramData\oracle.txt) ; foreach ($linea in $fichero) {$salida += $linea }; $salida = $salida.Replace(" ",""); [System.IO.File]::WriteAllBytes("c:\programdata\restart-service.exe", [System.Convert]::FromBase64String($salida)) > c:\programdata\monta.ps1
    ```

    * After executing the batch script by double-clicking on it, we wait a few minutes to spot the `oracle.txt` file which contains another file full of base64 lines, and the script `monta.ps1` which contains the following content, under the directory `c:\programdata\`.
    * Listing the content of the file `monta.ps1` reveals the following code

    ```powershell-session
    cat C:\programdata\monta.ps1
    ```

    ```powershell-session
    $salida = $null; $fichero = (Get-Content C:\ProgramData\oracle.txt) ; foreach ($linea in $fichero) {$salida += $linea }; $salida = $salida.Replace(" ",""); [System.IO.File]::WriteAllBytes("c:\programdata\restart-service.exe", [System.Convert]::FromBase64String($salida))
    ```

    * This script simply reads the contents of the `oracle.txt` file and decodes it to the `restart-service.exe` executable.
    * Running this script gives us a final executable that we can further analyze.

    ```powershell-session
     ls C:\programdata\
    ```

    ```powershell-session

    Mode                LastWriteTime         Length Name
    <SNIP>
    -a----        3/24/2023   1:01 PM            273 monta.ps1
    -a----        3/24/2023   1:01 PM         601066 oracle.txt
    -a----        3/24/2023   1:17 PM         432273 restart-service.exe
    ```

    * Now when executing `restart-service.exe` we are presented with the banner `Restart Oracle` created by `HelpDesk` back in 2010.
    * Inspecting the execution of the executable through `ProcMon64` shows that it is querying multiple things in the registry and does not show anything solid to go by.
    *

        <figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>


    * start `x64dbg`, navigate to `Options` -> `Preferences`, and uncheck everything except `Exit Breakpoint`:
    *

        <figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>


    * By unchecking the other options, the debugging will start directly from the application's exit point, and we will avoid going through any `dll` files that are loaded before the app starts.
      * Then, we can select `file` -> `open` and select the `restart-service.exe` to import it and start the debugging.
      * Once imported, we right click inside the `CPU` view and `Follow in Memory Map`:
      *

          <figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>


      * Checking the memory maps at this stage of the execution, of particular interest is the map with a size of `0000000000003000` with a type of `MAP` and protection set to `-RW--`.
      *

          <figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>


      * Memory-mapped files allow applications to access large files without having to read or write the entire file into memory at once. Instead, the file is mapped to a region of memory that the application can read and write as if it were a regular buffer in memory.
        * This could be a place to potentially look for hardcoded credentials.
        * If we double-click on it, we will see the magic bytes `MZ` in the `ASCII` column that indicates that the file is a [DOS MZ executable](https://en.wikipedia.org/wiki/DOS\_MZ\_executable).
        *

            <figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>


        * return to the Memory Map pane, then export the newly discovered mapped item from memory to a dump file by right-clicking on the address and selecting `Dump Memory to File`.
          * Running `strings` on the exported file reveals some interesting information.

```powershell-session
C:\TOOLS\Strings\strings64.exe .\restart-service_00000000001E0000.bin
```

```powershell-session

<SNIP>
"#M
z\V
).NETFramework,Version=v4.0,Profile=Client
FrameworkDisplayName
.NET Framework 4 Client Profile
<SNIP>
```

* Reading the output reveals that the dump contains a `.NET` executable.
* We can use `De4Dot` to reverse `.NET` executables back to the source code by dragging the `restart-service_00000000001E0000.bin` onto the `de4dot` executable.

.

```cmd-session
de4dot v3.1.41592.3405

Detected Unknown Obfuscator (C:\Users\cybervaca\Desktop\restart-service_00000000001E0000.bin)
Cleaning C:\Users\cybervaca\Desktop\restart-service_00000000001E0000.bin
Renaming all obfuscated symbols
Saving C:\Users\cybervaca\Desktop\restart-service_00000000001E0000-cleaned.bin


Press any key to exit...
```

* &#x20;we can read the source code of the exported application by dragging and dropping it onto the `DnSpy` executable.

<figure><img src="../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

