# How Certificate Transparency Logs Work

**1. Certificate Request**

* A website owner asks a trusted Certificate Authority (CA) for an SSL/TLS certificate.
* The CA verifies domain ownership and checks business credentials.
* After validation, the CA creates a **pre‑certificate** a preliminary version with all key details.

**2. Logging the Pre‑certificate**

* The CA submits the pre‑certificate to multiple independent CT logs.
* These logs are distributed globally to avoid a single point of control.
* Each log entry is permanent and tamper‑proof, forming a secure public ledger.

**3. Getting a Signed Certificate Timestamp (SCT)**

* Each log issues an SCT a cryptographic timestamp proving the pre‑certificate was logged at a specific time.
* The SCT acts like a digital receipt that's embedded into the final certificate.
* The timestamp is secure and cannot be forged or altered.

**4. Browser Verification**

* When a user visits the website, the browser checks the certificate’s SCTs against public CT logs in real time.
* If everything matches, the browser shows the padlock and establishes a secure connection.
* If the SCTs are missing or inconsistent, the browser warns the user.

**5. Ongoing Monitoring by the Community**

* Security researchers, domain owners, and browser vendors scan CT logs for suspicious certificates.
* They look for unauthorized certificates, policy violations, or malicious activity.
* Alerts are issued quickly when something seems off, ensuring swift corrective action.
