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



