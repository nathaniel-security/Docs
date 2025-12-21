# Kerberos

* Kerberos uses symmetric encryption and provides mutual authentication of both clients and servers.
* build by MIT
* Kerberos has the following components:
* Principal: Client (user) or service
* Realm: A logical Kerberos network
* Ticket: Data that authenticates a principal’s identity Access Control technologies
* Credentials: a ticket and a service key
* KDC: Key Distribution Center, which authenticates principals
* TGS: Ticket Granting Service
* TGT: Ticket Granting Ticket
* C/S: Client/Server



#### Kerberos Operational Steps

*

    <figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>
* Alice, wishes to access a printer
  * Kerberos Principal Alice contacts the KDC (Key Distribution Center, which acts as an \[\[Identity and Authentication ,Authorization, ,Accountability and Auditing(AAAA) and Non-repudiation|authentication]] server), requesting \[\[Identity and Authentication ,Authorization, ,Accountability and Auditing(AAAA) and Non-repudiation|authentication]]
  * The KDC sends Alice a session key, encrypted with Alice’s secret key.
    * The KDC also sends a TGT (Ticket Granting Ticket), encrypted with the TGS’s secret key
  * Alice decrypts the session key and uses it to request permission to print from the TGS (Ticket Granting Service)
    * Alice cannot decrypt the TGT
      * only the TGS can
  * Seeing Alice has a valid session key (and therefore has proven her identity claim), the TGS sends Alice a C/S session key (second session key) to use to print.
    * The TGS also sends a service ticket, encrypted with the printer’s key
  * Alice connects to the printer.
    * The printer, seeing a valid C/S session key, knows Alice has permission to print, and also knows that Alice is authentic
  *

      <figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

<figure><img src="/broken/files/IAYLp9kmUmk1gWplnAPA" alt=""><figcaption></figcaption></figure>

* [https://youtu.be/5N242XcKAsM](https://youtu.be/5N242XcKAsM)

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>



#### Kerberos Weaknesses

* primary weakness of Kerberos is that the KDC stores the keys of all principals

