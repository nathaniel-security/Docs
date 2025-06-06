# PRTG Network Monitor Attacks

* [PRTG Network Monitor](https://www.paessler.com/prtg) is agentless network monitor software

### Default Credentials

```
prtgadmin:prtgadmin
```

### Nessus PRTG Network Monitor Detection

* [https://www.tenable.com/plugins/nessus/51874](https://www.tenable.com/plugins/nessus/51874)

### Discovery/Footprinting/Enumeration

```shell-session
sudo nmap -sV -p- --open -T4 10.129.201.50
```

```shell-session
8080/tcp  open  http          Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)
```

* PRTG also shows up in the EyeWitness scan we performed earlier. - EyeWitness lists the [default credentials](app://obsidian.md/Default%20Passwords) `prtgadmin:prtgadmin`

<figure><img src="../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Version

```shell-session
curl -s http://10.129.201.50:8080/index.htm -A "Mozilla/5.0 (compatible;  MSIE 7.01; Windows NT 5.0)" | grep version
```

* better version below

```
curl -s http://10.129.201.50:8080/index.htm -A "Mozilla/5.0 (compatible;  MSIE 7.01; Windows NT 5.0)" | grep prtgversion
```

**Ressult**

```shell-session
  <link rel="stylesheet" type="text/css" href="/css/prtgmini.css?prtgversion=17.3.33.2830__" media="print,screen,projection" />
<div><h3><a target="_blank" href="https://blog.paessler.com/new-prtg-release-21.3.70-with-new-azure-hpe-and-redfish-sensors">New PRTG release 21.3.70 with new Azure, HPE, and Redfish sensors</a></h3><p>Just a short while ago, I introduced you to PRTG Release 21.3.69, with a load of new sensors, and now the next version is ready for installation. And this version also comes with brand new stuff!</p></div>
    <span class="prtgversion">&nbsp;PRTG Network Monitor 17.3.33.2830 </span>
```

## Reference

* [https://codewatch.org/2018/06/25/prtg-18-2-39-command-injection-vulnerability/](https://codewatch.org/2018/06/25/prtg-18-2-39-command-injection-vulnerability/)
