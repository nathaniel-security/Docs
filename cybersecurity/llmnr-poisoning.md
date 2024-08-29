# LLMNR Poisoning

### Quick Example - LLMNR/NBT-NS Poisoning

Let's walk through a quick example of the attack flow at a very high level:

1. A host attempts to connect to the print server at \print01.inlanefreight.local, but accidentally types in \printer01.inlanefreight.local.
2. The DNS server responds, stating that this host is unknown.
3. The host then broadcasts out to the entire local network asking if anyone knows the location of \printer01.inlanefreight.local.
4. The attacker (us with `Responder` running) responds to the host stating that it is the \printer01.inlanefreight.local that the host is looking for.
5. The host believes this reply and sends an authentication request to the attacker with a username and NTLMv2 password hash.
6. This hash can then be cracked offline or used in an SMB Relay attack if the right conditions exist.



* what is it
*

    <figure><img src="../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>



    * how it works
    *

        <figure><img src="../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

        <figure><img src="../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>


    *   assumption made that a person will put a wrong Ip or domain of a smb while connecting

        * using responder to capture hashes

        ```
        sudo responder -I eth0 -dwv 
        ```

<figure><img src="../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>



* crack with [hashcat.md](hashcat.md "mention")
* Several tools can be used to attempt LLMNR & NBT-NS poisoning:

| **Tool**                                              | **Description**                                                                                     |
| ----------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| [Responder](https://github.com/lgandx/Responder)      | Responder is a purpose-built tool to poison LLMNR, NBT-NS, and MDNS, with many different functions. |
| [Inveigh](https://github.com/Kevin-Robertson/Inveigh) | Inveigh is a cross-platform MITM platform that can be used for spoofing and poisoning attacks.      |
| [Metasploit](https://www.metasploit.com/)             | Metasploit has several built-in scanners and spoofing modules made to deal with poisoning attacks.  |

* &#x20;Inveigh is written in both C# and PowerShell (considered legacy).

### LLMNR/NBT-NS Poisoning - from Linux

* Some common options we'll typically want to use are `-wf`;
  * this will start the WPAD rogue proxy servers
  * &#x20;`-f` will attempt to fingerprint the remote host operating system and version
* We can use the `-v` flag for increased verbosity if we are running into issues
* Other options such as `-F` and `-P` can be used to force NTLM or Basic authentication and force proxy authentication
  * but may cause a login prompt, so they should be used sparingly
* If you are successful and manage to capture a hash, Responder will print it out on screen and write it to a log file per host located in the `/usr/share/responder/logs` directory
* after you get the hash crack with [hashcat](app://obsidian.md/hashcat)

```shell-session
 hashcat -m 5600 forend_ntlmv2 /usr/share/wordlists/rockyou.txt 
```

### LLMNR/NBT-NS Poisoning - from Windows

* [inveigh.md](inveigh.md "mention")
* Let's start Inveigh with LLMNR and NBNS spoofing, and output to the console and write to a file.

```powershell-session
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

```
cat Inveigh-NTLMv2.txt |clip
```

* copy hashes to [Hashcat](app://obsidian.md/Hashcat)and break

```
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt 
```



### C# Inveigh (InveighZero)

* [inveighzero.md](inveighzero.md "mention")





### Remediation

* &#x20;To ensure that these spoofing attacks are not possible, we can disable LLMNR and NBT-NS
* We can disable LLMNR in Group Policy by going to Computer Configuration --> Administrative Templates --> Network --> DNS Client and enabling "Turn OFF Multicast Name Resolution."

<figure><img src="../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

* NBT-NS cannot be disabled via Group Policy but must be disabled locally on each host
  * We can do this by opening `Network and Sharing Center` under `Control Panel`,
    * clicking on `Change adapter settings`,
    * right-clicking on the adapter to view its properties,
    * selecting `Internet Protocol Version 4 (TCP/IPv4)`,
      * and clicking the `Properties` button,
      * then clicking on `Advanced`&#x20;
      * selecting the `WINS` tab
      * finally selecting `Disable NetBIOS over TCP/IP`.





<figure><img src="../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>



* While it is not possible to disable NBT-NS directly via GPO, we can create a PowerShell script under Computer Configuration --> Windows Settings --> Script (Startup/Shutdown) --> Startup with something like the following:



```powershell
$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
Get-ChildItem $regkey |foreach { Set-ItemProperty -Path "$regkey\$($_.pschildname)" -Name NetbiosOptions -Value 2 -Verbose}
```

* In the Local Group Policy Editor,
  * we will need to double click on `Startup`,
  * choose the `PowerShell Scripts` tab,
  * select "For this GPO, run scripts in the following order" to `Run Windows PowerShell scripts first`,
  * then click on `Add` and choose the script.
  * For these changes to occur, we would have to either reboot the target system or restart the network adapter.



<figure><img src="../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>



* To push this out to all hosts in a domain, we could create a GPO using `Group Policy Management` on the Domain Controller and host the script on the SYSVOL share in the scripts folder and then call it via its UNC path such as:

`\\inlanefreight.local\SYSVOL\INLANEFREIGHT.LOCAL\scripts`

* Once the GPO is applied to specific OUs and those hosts are restarted,
  * the script will run at the next reboot and disable NBT-NS, provided that the script still exists on the SYSVOL share and is accessible by the host over the network.



<figure><img src="../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>





* Other mitigations include filtering network traffic to block LLMNR/NetBIOS traffic and enabling SMB Signing to prevent NTLM relay attacks.
* Network intrusion detection and prevention systems can also be used to mitigate this activity, while network segmentation can be used to isolate hosts that require LLMNR or NetBIOS enabled to operate correctly.

### Detection

* One way is to use the attack against the attackers by injecting LLMNR and NBT-NS requests for non-existent hosts across different subnets and alerting if any of the responses receive answers which would be indicative of an attacker spoofing name resolution responses.
  * This [blog post](https://www.praetorian.com/blog/a-simple-and-effective-way-to-detect-broadcast-name-resolution-poisoning-bnrp/) explains this method more in-depth.
* Furthermore, hosts can be monitored for traffic on
  * ports UDP 5355 and 137,
  * and event IDs [4697](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4697) and [7045](https://www.manageengine.com/products/active-directory-audit/kb/system-events/event-id-7045.html) can be monitored for.
* Finally, we can monitor the registry key `HKLM\Software\Policies\Microsoft\Windows NT\DNSClient` for changes to the `EnableMulticast` DWORD value.
  * A value of `0` would mean that LLMNR is disabled.

## Reference

* https://youtu.be/VXxH4n684HE?t=5888
