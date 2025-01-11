# Spotting Spoofing Securing Your Email with DMARC, DKIM, and SPF

<figure><img src="https://images.unsplash.com/photo-1603791440384-56cd371ee9a7?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwxMHx8ZW1haWwlMjBzZWN1cml0eXxlbnwwfHx8fDE3MzU5ODY5MDF8MA&#x26;ixlib=rb-4.0.3&#x26;q=85" alt=""><figcaption></figcaption></figure>

* In today's digital landscape, securing your email domain is more important than ever. **Email spoofing**, where attackers impersonate a trusted domain to send fraudulent emails, can have serious consequences, from phishing attacks to spreading malware.
* The best way to combat email spoofing is by properly configuring and monitoring your domain's email authentication mechanisms: **DMARC**, **DKIM**, and **SPF**.
* In this post, we’ll walk you through how to check if your domain’s email records are correctly set up and how to assess the likelihood of spoofing. We’ll also provide an overview of these mechanisms and explain how they work together to protect your domain.

#### **Key Email Authentication Records:**

* **DMARC (Domain-based Message Authentication, Reporting & Conformance)**:
  * Provides policy and reporting mechanisms for **DKIM** and **SPF**.
  * Specifies how emails that fail DKIM or SPF checks should be handled.
  * Allows feedback on the results of authentication checks.
* **DKIM (DomainKeys Identified Mail)**:
  * Uses cryptographic signatures to verify the authenticity of email messages.
  * Adds a **digital signature** to the email header, ensuring the content has not been tampered with.
* **SPF (Sender Policy Framework)**:
  * Allows domain owners to specify which IP addresses can send emails on behalf of their domain.
  * The recipient's email server checks the **SPF record** to verify the sender’s IP.

#### **How the Code Works to Check Email Records**

The code checks your domain’s **DMARC**, **DKIM**, and **SPF** records to assess whether the domain is **vulnerable to spoofing** and if the email configuration is strong.

**1. DMARC Record Check:**

* **If DMARC is missing or empty:**
  * Domain is **highly vulnerable** to spoofing.
* **If DMARC record is invalid** (doesn’t contain `v=DMARC1`):
  * Domain is **vulnerable** to spoofing.
* **If DMARC policy is `p=reject`**:
  * Strong protection against spoofing.
  * **Spoofing is unlikely**.
* **If DMARC policy is `p=quarantine`:**
  * Moderate protection against spoofing.
    * **Spoofing is possible but harder**.
* **If DMARC policy is `p=none`:**
  * **Highly vulnerable** to spoofing.
    * **Spoofing is very likely**.

**2. DKIM Record Check:**

* **If DKIM is missing**:
  * The domain is more vulnerable to spoofing.

**3. SPF Record Check:**

* **If SPF is missing or invalid**:
  * Domain is **vulnerable** to spoofing.
* **If SPF record is invalid** (doesn’t contain `v=spf1`):
  * Domain is considered **vulnerable**.
* **If SPF policy is `-all`:**
  * Strict policy, only authorized senders can send email. **Spoofing is unlikely**.
* **If SPF policy is `~all`:**
  * Soft fail policy.
  * Unauthorised senders are allowed but marked as suspicious. **Spoofing is possible but harder**.
* **If SPF policy is `?all`:**
  * Neutral policy, any sender is allowed.
  * **Spoofing is possible**.
* **If SPF policy is `+all`:**
  * Open policy, anyone can send email.
  * **Highly vulnerable** to spoofing.

***

#### **Conclusion:**

Securing your email domain is a fundamental step in protecting against phishing and spoofing attacks. By ensuring that your **DMARC**, **DKIM**, and **SPF** records are properly configured, you can greatly reduce the likelihood of email spoofing and improve the trustworthiness of your email communications.

**Ready to protect your domain?** Use the steps outlined above to check and improve your email security today!
