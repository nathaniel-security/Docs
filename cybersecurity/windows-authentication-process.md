# Windows Authentication Process

* The [Windows client authentication process](https://docs.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication)  consists of many different modules that perform
  * the entire logon
  * retrieval
  * verification processes.
* The [Local Security Authority](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/configuring-additional-lsa-protection) (`LSA`) is a protected subsystem that authenticates users and logs them into the local computer.
  * In addition, the LSA maintains information about all aspects of local security on a computer. It also provides various services for translating between names and security IDs (`SIDs`).
  * The security subsystem keeps track of the security policies and accounts that reside on a computer system.
  * In the case of a Domain Controller, these policies and accounts apply to the domain where the Domain Controller is located.
    * These policies and accounts are stored in Active Directory.
  * In addition, the LSA subsystem provides services for checking access to objects, checking user permissions, and generating monitoring messages.

<figure><img src="../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

* `Winlogon` is a trusted process responsible for managing security-related user interactions. These include:
  * Launching LogonUI to enter passwords at login
  * Changing passwords
  * Locking and unlocking the workstation
* Winlogon is the only process that intercepts login requests from the keyboard sent via an RPC message from Win32k.sys.
  * Winlogon immediately launches the LogonUI application at logon to display the user interface for logon.
  * After Winlogon obtains a user name and password from the credential providers, it calls LSASS to authenticate the user attempting to log in.

### LSASS

* [Local Security Authority Subsystem Service](https://en.wikipedia.org/wiki/Local\_Security\_Authority\_Subsystem\_Service) (`LSASS`) is a collection of many modules and has access to all authentication processes
  * &#x20;found in `%SystemRoot%\System32\Lsass.exe`
* is responsible for the
  * local system security policy
  * user authentication
  * sending security audit logs to the `Event log`

| **Authentication Packages** | **Description**                                                                                                                                                                                                                                                |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Lsasrv.dll`                | The LSA Server service both enforces security policies and acts as the security package manager for the LSA. The LSA contains the Negotiate function, which selects either the NTLM or Kerberos protocol after determining which protocol is to be successful. |
| `Msv1_0.dll`                | Authentication package for local machine logons that don't require custom authentication.                                                                                                                                                                      |
| `Samsrv.dll`                | The Security Accounts Manager (SAM) stores local security accounts, enforces locally stored policies, and supports APIs.                                                                                                                                       |
| `Kerberos.dll`              | Security package loaded by the LSA for Kerberos-based authentication on a machine.                                                                                                                                                                             |
| `Netlogon.dll`              | Network-based logon service.                                                                                                                                                                                                                                   |
| `Ntdsa.dll`                 | This library is used to create new records and folders in the Windows registry.                                                                                                                                                                                |

* Each interactive logon session creates a separate instance of the Winlogon service.
  * The [Graphical Identification and Authentication](https://docs.microsoft.com/en-us/windows/win32/secauthn/gina) (`GINA`) architecture is loaded into the process area used by Winlogon, receives and processes the credentials, and invokes the authentication interfaces via the [LSALogonUser](https://docs.microsoft.com/en-us/windows/win32/api/ntsecapi/nf-ntsecapi-lsalogonuser) function.

### SAM Database

* The [Security Account Manager](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc756748\(v=ws.10\)?redirectedfrom=MSDN) (`SAM`) is a database file in Windows operating systems that stores users' passwords.
* It can be used to authenticate local and remote users
* SAM uses cryptographic measures to prevent unauthenticated users from accessing the system.
* User passwords are stored in a hash format in a registry structure as either an `LM` hash or an `NTLM` hash.
  * This file is located in `%SystemRoot%/system32/config/SAM` and is mounted on HKLM/SAM. SYSTEM level permissions are required to view it.
* Windows systems can be assigned to either a workgroup or domain during setup.
  * If the system has been assigned to a workgroup
    * it handles the SAM database locally and stores all existing users locally in this database
  * However, if the system has been joined to a domain
    * the Domain Controller (`DC`) must validate the credentials from the Active Directory database (`ntds.dit`), which is stored in `%SystemRoot%\ntds.dit`.
* Microsoft introduced a security feature in Windows NT 4.0 to help improve the security of the SAM database against offline software cracking.
  * This is the `SYSKEY` (`syskey.exe`) feature, which, when enabled, partially encrypts the hard disk copy of the SAM file so that the password hash values for all local accounts stored in the SAM are encrypted with a key.

### Credential Manager

<figure><img src="../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Credential Manager is a feature built-in to all Windows operating systems that allows users to save the credentials they use to access various network resources and websites.
  * Saved credentials are stored based on user profiles in each user's `Credential Locker`.
  * Credentials are encrypted and stored at the following location:

```powershell-session
C:\Users\[Username]\AppData\Local\Microsoft\[Vault/Credentials]\
```
