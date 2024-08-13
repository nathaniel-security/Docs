# Extracting windows wifi password

### Viewing Saved Wireless Networks

```cmd-session
netsh wlan show profile
```

### Retrieving Saved Wireless Passwords

```cmd-session
netsh wlan show profile ilfreight_corp key=clear
```
