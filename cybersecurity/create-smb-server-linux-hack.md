# Create SMB server Linux (HACK)

### Python Version

```
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/$(whoami)/Desktop/
```

### Create the SMB Server without a Username and Password

```shell-session
 sudo impacket-smbserver share -smb2support /tmp/smbshare
```

### Create the SMB Server with a Username and Password

```shell-session
sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```

### samba

```
apt-get install samba
mkdir /tmp/smb
chmod 777 /tmp/smb
#Add to the end of /etc/samba/smb.conf this:
[public]
    comment = Samba on Ubuntu
    path = /tmp/smb
    read only = no
    browsable = yes
    guest ok = Yes
#Start samba
service smbd restart
```
