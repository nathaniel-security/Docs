# Amazon Macie Lab: Spotting Sensitive Data Like a Boss

<a href="https://github.com/nathaniel-security/Amazon_Macie_labs" class="button primary">Link to Github Project</a>

#### **What’s This Lab All About?**

**Amazon\_Macie\_labs** is a purpose-built playground to test out AWS Macie’s swagger when it comes to sniffing out sensitive info. Inside, there's a nifty `main.py` that spins up a `credit.csv` file stuffed with fake PII think names, SSNs, credit card numbers all randomly generated



<figure><img src="../.gitbook/assets/ChatGPT Image Aug 7, 2025, 10_55_15 PM.png" alt="" width="188"><figcaption></figcaption></figure>

#### **How It Rolls**

* Fire off:

```
python src/main.py
```

* Upload that CSV to an S3 bucket.
* Head over to AWS Macie in the console. Make sure it’s enabled and properly configured to scan your bucket. If needed, spin up a new discovery job targeting it.
* Once Macie wraps up scanning, dive into the findings. You should see clear detection of those fake credit card nuggets and PII—proof that Macie is sharper than your average watchdog

#### **Why This Matters?**

* **Real‑world testing, zero risk**: You’re working with mock data, but replicating how Macie behaves in serious scenarios.
* **Quick feedback loop**: Generate → upload → scan → review = rapid validation.
* **Perfect for proof-of-concepts or demos**: Show stakeholders that Macie isn’t just hype it actually flags sensitive info
