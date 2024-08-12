# How Nmap gets what OS is running by using different probes



{% embed url="https://www.canva.com/design/DAGNQqTn1qk/bNzmPZXej_Fr85wO3bSGEg/view" fullWidth="true" %}

* \
  NMAP -Pn Flag
  * part of the host discovery process - sends a SYN packet to port 443 - and ACK packet to port 80
    * Firewall evasion:
      * SYN to 443:
        * Many firewalls allow incoming SYN packets to port 443 (HTTPS) to initiate new connections.
      * ACK to 80:
        * Some firewalls are configured to allow ACK packets, assuming they're part of an established connection.
    * Increased accuracy
* nmap fingerprinting
  * In an ideal world,
    * every different OS would correspond to exactly one unique fingerprint.
  * OS vendors don't make life so easy for us
  * The same OS release may fingerprint differently based on
    * what network drivers are in use
    * user-configurable options
    * patch levels
    * processor architecture
    * amount of RAM available
    * firewall settings
  * Because of which an OS can have a little variation wrt fingerprints
  * &#x20;On the other side
    * fingerprints on an embedded devices which share a common OS
      * &#x20;a printer from one vendor and an ethernet switch from another may actually share an embedded OS from a third vendor
        * In many cases, subtle differences between the devices still allow them to be distinguished.

#### nmap os db

* [https://svn.nmap.org/nmap/nmap-os-db](https://svn.nmap.org/nmap/nmap-os-db)
  * OS family includes products such as `Windows`, `Linux`, `IOS` (for Cisco routers), `Solaris`, and `OpenBSD`
    * also hundreds of devices such as switches, broadband routers, and printers which use undisclosed operating systems.
  * When the underlying OS isn't clear,&#x20;
    * `embedded` is used.
  * **nmap OS level scans become more accurate with -p- flag or even -Su flag but take more time**

#### Test expressions

* some Windows XP machines return a Window size of 62500 to the `T1` probe, while others return 64240
  * In any case, we would like to detect Windows XP no matter which window size is used.

#### Probes Sent

#### Response Tests

#### ICMP response code (`CD`)

* say hello audience
* ICMP response code (`CD`)
  * The code value of an ICMP echo reply
    * &#x20;is supposed to be zero
      * But some implementations wrongly send other values,
        * particularly if the echo request has a nonzero code (as one of the `IE` tests does).
        * The response code values for the two probes are combined into a `CD` value as described

| Value    | Description                                                    |
| -------- | -------------------------------------------------------------- |
| Z        | Both code values are zero.                                     |
| S        | Both code values are the same as in the corresponding probe.   |
| _`<NN>`_ | When they both use the same non-zero number, it is shown here. |
| O        | Any other combination.                                         |

#### Returned probe IP ID value (`RID`)

* The `U1` probe has a static IP identifier value of 0x1042 (4162)
  * ![Pasted image 20240811020830.png](app://b75f6ea3707d198027a6845abad5b677c1d2/E:/NATHANIEL%20DATA/Personal/OneDrive/obsidian/Alex/Alex/09-Attachments/Pasted%20image%2020240811020830.png?1723322310677)
  * **data is got from closed ports**
* Some systems, such as Solaris, manipulate IP ID values for raw IP packets that Nmap sends.
  * If that value is returned in the port unreachable message, the value `G` is stored for this test.
  * found that some systems, particularly HP and Xerox printers, flip the bytes and return 0x4210 instead.

#### Returned probe IP total length value (`RIPL`)

* ICMP port unreachable messages
  * as are sent in response to the `U1` probe
  * are required to include the IP header which generated them.
* This header should be returned just as they received it
  * but some implementations send back a corrupted version due to changes they made during IP processing.
  * If the correct value of is returned,
    * the value `G` (for good) is stored
    * else of the actual value.
    * **test simply records the returned IP total length value**

#### Unused port unreachable field nonzero (`UN`)

* An ICMP port unreachable message header is eight bytes long, but only the first four are used.
  * RFC 792 states that the last four bytes must be zero
  * A few implementations
    * mostly ethernet switches and some specialized embedded devices
    * set it anyway.
    * The value of those last four bytes is recorded in this field.

#### TCP RST data checksum (`RD`)

* Some operating systems
  * return ASCII data such as error messages
    * in reset packets
      * This is explicitly allowed by section 4.2.2.12 of [RFC 1122](http://www.rfc-editor.org/rfc/rfc1122.txt)
* When there is no data, `RD` is set to zero
* the data in the RST is stored

#### TCP sequence number (`S`)

* This test examines the 32-bit sequence number field in the TCP header.
* this one examines how it compares to the TCP acknowledgment number from the probe that elicited the response.

| Value | Description                                                                     |
| ----- | ------------------------------------------------------------------------------- |
| Z     | Sequence number is zero.                                                        |
| A     | Sequence number is the same as the acknowledgment number in the probe.          |
| A+    | Sequence number is the same as the acknowledgment number in the probe plus one. |
| O     | Sequence number is something else (other).                                      |

#### TCP acknowledgment number (`A`)

* This test is the same as `S`
  * &#x20;except that it tests how the acknowledgment number in the response compares to the sequence number in the respective probe

| Value | Description                                                                     |
| ----- | ------------------------------------------------------------------------------- |
| Z     | Acknowledgment number is zero.                                                  |
| S     | Acknowledgment number is the same as the sequence number in the probe.          |
| S+    | Acknowledgment number is the same as the sequence number in the probe plus one. |
| O     | Acknowledgment number is something else (other).                                |

#### IP initial time-to-live (`T`)

* Nmap determines how many hops away it is from the target by examining the ICMP port unreachable response to the `U1` probe. ![Pasted image 20240811022531.png](app://b75f6ea3707d198027a6845abad5b677c1d2/E:/NATHANIEL%20DATA/Personal/OneDrive/obsidian/Alex/Alex/09-Attachments/Pasted%20image%2020240811022531.png?1723323331191)

#### TCP initial window size (`W`, `W1`–`W6`)

* records the 16-bit TCP window size of the received packet
* There are more than 80 values that at least one OS
* A down side is that some operating systems have more than a dozen possible values by themselves.
* This leads to false negative results until we collect all of the possible window sizes used by an operating system.

#### Shared IP ID sequence Boolean (`SS`)

* This Boolean value records whether the target shares its IP identifiers sequence between the TCP and ICMP protocols.
* If our six TCP IP ID values are
  * 117, 118, 119, 120, 121, and 122,
    * then our ICMP results are 123 and 124, it is clear
    * that not only are both sequences incremental,
      * but they are both part of the same sequence.
        * result is S
  * TCP IP ID values are 117–122
    * but the ICMP values are 32,917 and 32,918, two different sequences are being used.
      * result is O



