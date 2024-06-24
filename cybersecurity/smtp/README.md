# SMTP

* Simple Mail Transfer Protocol (`SMTP`)
* It can be used
  * between an email client
  * an outgoing mail server
  * or between two SMTP servers
* Ports used
  * 25
  * SMTP servers also use other ports such as TCP port `587`
    * This port is used to receive mail from authenticated users/servers, usually using the STARTTLS command to switch the existing plaintext connection to an encrypted connection
      * authentication data is protected
* At the beginning of the connection, authentication occurs when the client confirms its identity with a user name and password.
  * The emails can then be transmitted.
  * For this purpose, the client sends the server sender and recipient addresses, the email's content, and other information and parameters.
  * After the email has been transmitted, the connection is terminated again.
  * The email server then starts sending the email to another SMTP server.
* SMTP works unencrypted without further measures and transmits all commands, data, or authentication information in plain text.
  * To prevent unauthorized reading of data, the SMTP is used in conjunction with SSL/TLS encryption.
  * Under certain circumstances, a server uses a port other than the standard TCP port `25` for the encrypted connection, for example, TCP port `465`.

### Spam Filter

* most modern SMTP servers support the protocol extension ESMTP with SMTP-Auth.
* &#x20;After sending his e-mail, the SMTP client, also known as `Mail User Agent` (MUA), converts it into a header and a body and uploads both to the SMTP server.
* This has a so-called `Mail Transfer Agent` (`MTA`),
  * the software basis for sending and receiving e-mails
* The MTA checks the e-mail for size and spam and then stores it.
  * To relieve the MTA, it is occasionally preceded by a `Mail Submission Agent` (`MSA`), which checks the validity, i.e., the origin of the e-mail.
  * This `MSA` is also called `Relay` server.
  * These are very important later on, as the so-called `Open Relay Attack` can be carried out on many SMTP servers due to incorrect configuration.
  * We will discuss this attack and how to identify the weak point for it a little later
  * The MTA then searches the DNS for the IP address of the recipient mail server.
* On arrival at the destination SMTP server, the data packets are reassembled to form a complete e-mail. From there, the `Mail delivery agent` (`MDA`) transfers it to the recipient's mailbox.

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td>Client (<code>MUA</code>)</td><td><code>➞</code></td><td>Submission Agent (<code>MSA</code>)</td><td><code>➞</code></td><td>Open Relay (<code>MTA</code>)</td><td><code>➞</code></td><td>Mail Delivery Agent (<code>MDA</code>)</td><td><code>➞</code></td><td>Mailbox (<code>POP3</code>/<code>IMAP</code>)</td></tr></tbody></table>

### SMTP Disadvantages

* But SMTP has two disadvantages inherent to the network protocol.
  * SMTP does not return a usable delivery confirmation
    * Although the specifications of the protocol provide for this type of notification, its formatting is not specified by default, so that usually only an English-language error message, including the header of the undelivered message, is returned.
  * Users are not authenticated when a connection is established, and the sender of an email is therefore unreliable
    * As a result, open SMTP relays are often misused to send spam to the masses
    * The originators use arbitrary fake sender addresses for this purpose to not be traced (mail spoofing).
    * Today, many different security techniques are used to prevent the misuse of SMTP servers
      * For example, suspicious emails are rejected or moved to quarantine (spam folder).
        * For example,
          * responsible for this are the identification protocol DomainKeys (`DKIM`)
          * Sender Policy Framework(`SPF`)
* For this purpose, an extension for SMTP has been developed called `Extended SMTP` (`ESMTP`).
  * When people talk about SMTP in general, they usually mean ESMTP. ESMTP uses TLS, which is done after the `EHLO` command by sending `STARTTLS`.
  * This initializes the SSL-protected SMTP connection, and from this moment on, the entire connection is encrypted, and therefore more or less secure.
    * Now AUTH PLAIN extension for authentication can also be used safely
