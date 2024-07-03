# NTLM Authentication

**Hash Protocol Comparison**

| **Hash/Protocol** | **Cryptographic technique**                      | **Mutual Authentication** | **Message Type**                | **Trusted Third Party**                         |
| ----------------- | ------------------------------------------------ | ------------------------- | ------------------------------- | ----------------------------------------------- |
| `NTLM`            | Symmetric Cryptography                           | No                        | Random number                   | Domain Controller                               |
| `NTLMv1`          | Symmetric Cryptography                           | No                        | MD4 hash, random number         | Domain Controller                               |
| `NTLMv2`          | Symmetric key cryptography                       | No                        | MD4 hash, random number         | Domain Controller                               |
| `Kerberos`        | Symmetric Cryptography & asymmetric cryptography | Yes                       | Encrypted ticket using DES, MD5 | Domain Controller/Key Distribution Center (KDC) |

### LM

* `LAN Manager` (LM or LANMAN) hashes are the oldest password storage mechanism used by the Windows operating system.
* &#x20;If in use, they are stored in the SAM database on a Windows host and the NTDS.DIT database on a Domain Controller.
* Due to significant security weaknesses in the Hash Functions|hashing algorithm used for LM hashes, it has been turned off by default since Windows Vista/Server 2008.
  * However, it is still common to encounter, especially in large environments where older systems are still used.
  * Passwords using LM are limited to a maximum of `14` characters
* Passwords are not case sensitive and are converted to uppercase before generating the hashed value, limiting the keyspace to a total of 69 characters making it relatively easy to crack these hashes using a tool such as Hashcat.

### NTHash (NTLM)

* `NT LAN Manager` (NTLM) hashes are used on modern Windows systems.
* It is a challenge-response authentication protocol and uses three messages to authenticate: a client first sends a `NEGOTIATE_MESSAGE` to the server, whose response is a `CHALLENGE_MESSAGE` to verify the client's identity.
* Lastly, the client responds with an `AUTHENTICATE_MESSAGE`
* These hashes are stored locally in the SAM database or the NTDS.DIT database file on a Domain Controller
* The protocol has two hashed password values to choose from to perform authentication: the LM hash (as discussed above) and the NT hash, which is the MD4 hash of the little-endian UTF-16 value of the password.
* The algorithm can be visualized as: `MD4(UTF-16-LE(password))`.

**NTLM Authentication Request**

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

```shell-session
Rachel:500:aad3c435b514a4eeaad3b935b51304fe:e46b9e548fa0d122de7f59fb6d48eaa2:::
```

* `Rachel` is the username
* `500` is the Relative Identifier (RID). 500 is the known RID for the `administrator` account
* `aad3c435b514a4eeaad3b935b51304fe` is the LM hash and, if LM hashes are disabled on the system, can not be used for anything
* `e46b9e548fa0d122de7f59fb6d48eaa2` is the NT hash. This hash can either be cracked offline to reveal the cleartext value (depending on the length/strength of the password) or used for a pass-the-hash attack. Below is an example of a successful pass-the-hash attack using the CrackMapExec tool:

```
crackmapexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453
```

### NTLMv1 (Net-NTLMv1)

* The NTLM protocol performs a challenge/response between a server and client using the NT hash.
* NTLMv1 uses both the NT and the LM hash, which can make it easier to "crack" offline after capturing a hash using a tool such as [Responder](https://github.com/lgandx/Responder) or via an [NTLM relay attack](https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html)

**V1 Challenge & Response Algorithm**

```shell-session
C = 8-byte server challenge, random
K1 | K2 | K3 = LM/NT-hash | 5-bytes-0
response = DES(K1,C) | DES(K2,C) | DES(K3,C)
```

**NTLMv1 Hash Example**

```shell-session
u4-netntlm::kNS:338d08f8e26de93300000000000000000000000000000000:9526fb8c23a90751cdd619b6cea564742e1e4bf33006ba41:cb8086049ec4736c
```

### NTLMv2 (Net-NTLMv2)

* The NTLMv2 protocol was first introduced in Windows NT 4.0 SP4 and was created as a stronger alternative to NTLMv1.
* It has been the default in Windows since Server 2000.
* It is hardened against certain spoofing attacks that NTLMv1 is susceptible to. NTLMv2 sends two responses to the 8-byte challenge received by the server.
* These responses contain a 16-byte HMAC-MD5 hash of the challenge, a randomly generated challenge from the client, and an HMAC-MD5 hash of the user's credentials.
* A second response is sent, using a variable-length client challenge including the current time, an 8-byte random value, and the domain name.
* The algorithm is as follows

**V2 Challenge & Response Algorithm**

&#x20; \- NTLM Authentication

```shell-session
SC = 8-byte server challenge, random
CC = 8-byte client challenge, random
CC* = (X, time, CC2, domain name)
v2-Hash = HMAC-MD5(NT-Hash, user name, domain name)
LMv2 = HMAC-MD5(v2-Hash, SC, CC)
NTv2 = HMAC-MD5(v2-Hash, SC, CC*)
response = LMv2 | CC | NTv2 | CC*
```

* An example of an NTLMv2 hash is:

**NTLMv2 Hash Example**

&#x20; \- NTLM Authentication

```shell-session
admin::N46iSNekpT:08ca45b7d7ea58ee:88dcbe4446168966a153a0064958dac6:5c7830315c7830310000000000000b45c67103d07d7b95acd12ffa11230e0000000052920b85f78d013c31cdb3b92f5d765c783030
```

### Domain Cached Credentials (MSCache2)

* Microsoft developed the MS Cache v1 and v2 algorithm (also known as Domain Cached Credentials (DCC) to solve the potential issue of a domain-joined host being unable to communicate with a domain controller (i.e., due to a network outage or other technical issue) and, hence, NTLM/Kerberos authentication not working to access the host in question.
* Hosts save the last `ten` hashes for any domain users that successfully log into the machine in the `HKEY_LOCAL_MACHINE\SECURITY\Cache` registry key
* These hashes cannot be used in pass-the-hash attacks.
* Furthermore, the hash is very slow to crack with a tool such as Hashcat, even when using an extremely powerful GPU cracking rig, so attempts to crack these hashes typically need to be extremely targeted or rely on a very weak password in use
* These hashes can be obtained by an attacker or pentester after gaining local admin access to a host and have the following format: `$DCC2$10240#bjones#e4e938d12fe5974dc42a90120bd9c90f`.

