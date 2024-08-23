# DNS Zone Transfer Attack

* Synchronization between the servers involved is realized by zone transfer
  * Using a secret key `rndc-key`,
    * the default configuration, the servers make sure that they communicate with their own master or slave
* The slave fetches the `SOA` record of the relevant zone from the master at certain intervals,
  * the so-called refresh time,
    * usually one hour
  * compares the serial numbers
  * If the serial number of the SOA record of the master is greater than that of the slave, the data sets no longer match.

### Remarks

* if box is ubuntu and using dns tcp on port 53
  * might be susceptible to zone transfer attack

***

### Attack Internet

* get the name server

```bash
dig soa ZoneTransfer.me
```

<figure><img src="../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Exploit Zone transfer
  * linux
    * `dig axfr @nsztm1.digi.ninja zonetransfer.me`
    * `host -t axfr zonetransfer.me nsztm1.digi.ninja`
  * windows
    * `nslookup -type=axfr zonetransfer.me nsztm1.digi.ninja`

### Attack HTB

```bash
nslookup
```

```bash
server <server to ask for DNS request>
```

* ask the dns server to query it self

```
10.10.11.166
```

* for zone transfer

```bash
dig axfr @dns.server.ip top_level_domain.com
```

* example

```
dig axfr @10.10.10.123 friendzoneportal.red
```

## Reference

* https://yogesh-verma.medium.com/zone-transfer-attacks-a-practical-guide-to-detection-and-prevention-2e8346d0297e
