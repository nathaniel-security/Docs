---
hidden: true
---

# True Cybersecurity Governance: Where Financial Risk Modeling Meets Attack Graphs

This blog deviates from my usual technical content to explore concepts that fundamentally expanded my thinking about business and organizational security. While primarily focused on governance, this piece bridges the gap between technical implementation and strategic risk management.

### Background and Context

A couple of years ago, I got the opportunity to work with a couple of hedge fund managers few of of whom were with Swiss Bank. During that time, I learnt a lot about risk assessment and capital allocation, along with capital deallocation, and specifically the risk assessment aspect. While I don't come from a finance background, these principles have proven invaluable when applied to cybersecurity contexts. Personally, where is it worth allocating funds towards what project, and to what point does it not make sense to invest resources or reallocate assets towards a resource or project?

During a recent discussion with a few security industry professionals, revealed striking parallels between financial risk modelling and cybersecurity risk management. While the intersection of finance and security has always intrigued me, the conversations illuminated specific domains where these disciplines converge powerfully.

Technical perspective: Imagine combining all technical knowledge with automation to analyze attack vectors comprehensively, understanding not just how attackers compromise systems, but also calculating the probability and impact of each method. This probabilistic approach transforms security from reactive defense to predictive risk management.

### Cyber Risk Modelling Language

This framework embodies what I consider true cybersecurity governance, fully aligned with the ‘Govern’ function in modern cyber standards. My experience spans technical implementation and management across multiple organisations, but this approach crystallised scattered concepts into a coherent risk management structure.

Building Comprehensive Threat Models

Let’s construct a threat model layer by layer:

#### Application Layer:

Consider an application with an endpoint vulnerable to SQL injection. This forms our initial threat surface. (theoretical example of implementation to prove a point)

#### Infrastructure Layer:

The application runs on a server with port 80 exposed(http), directly serving requests.

Load Balancing Layer: The ALB listener is configured with path-based rules so that only `/user/verification` is forwarded to the backend. With the app instances in private subnets and security groups allowing traffic only from the ALB, `/user/internal_check` is not reachable from the internet(assume the security groups are configured correctly), even though it exists on the backend service.

Security Layer: A Web Application Firewall (WAF) sits before the ALB. Traffic flow becomes: `Internet → Domain → WAF → ALB → Application`, with strict endpoint restrictions.

The threats in this architecture include:

* Discovery of new vulnerabilities in the exposed user/verification endpoint
* Insider threats from employees accessing user/internal\_check directly (bypassing WAF/ALB protections)
* Compromised employee credentials, workstations, or identities leading to internal exploitation
* If traffic between the ALB and application is plain HTTP instead of TLS, any compromise of the underlying network plane or misconfigured intermediaries could expose sensitive data in transit.

Consider the business impact: if this application processes credit card data worth $10 million (accounting for competitive advantage loss and regulatory penalties), the primary risk becomes insider data exfiltration or sale.

Graph-Based Attack Modeling

Visualize security as a graph where nodes represent system components and edges represent compromise paths. Each connection serves dual purposes: functional relationships and potential attack vectors. For example:

* Web server compromise → Database credential access → Data exfiltration
* Each path carries probability and impact metrics.

Risk calculations incorporate:

* Historical compromise data from your organization
* Industry-standard breach statistics
* Potential loss scenarios weighted by likelihood.

Technical Deep Dive into Risk Quantification

Foundational Definitions

* Threat: The potential exploitation of a weakness
* Risk: The probability of threat realization multiplied by impact
* Vulnerability: A system weakness enabling exploitation

Mathematical Risk Framework

For our SQL injection scenario, here’s the probability calculation:

Annual Loss Expectancy (ALE) = Single Loss Expectancy (SLE) × Annual Rate of Occurrence (ARO)

Or using Factor Analysis of Information Risk (FAIR): Risk = Loss Event Frequency (LEF) × Loss Magnitude (LM)

Breaking this down:

**Loss Event Frequency (LEF)** = How often a loss occurs

* Threat Event Frequency (TEF): How often threats attempt to act (e.g., 1000 SQL injection attempts/month)
* Vulnerability: Probability the threat succeeds when attempted (e.g., 0.1% success rate)
* LEF = TEF × Vulnerability = 1000 × 0.001 = 1 successful breach/month

**Loss Magnitude (LM)** = Financial impact when loss occurs

* Primary Loss: Direct costs from the asset itself
  * Asset Value: Worth of compromised data (e.g., $10M customer database)
  * Loss Factor: Percentage lost in breach (e.g., 40% of data exposed)
  * Primary Loss = $10M × 0.4 = $4M
* Secondary Loss: Cascading effects
  * Reputational Damage: Lost customers, reduced sales (e.g., $2M)
  * Regulatory Fines: GDPR/CCPA penalties (e.g., $1M)
  * Operational Disruption: Downtime, recovery costs (e.g., $500K)
  * Secondary Loss = $2M + $1M + $0.5M = $3.5M

**Total Risk** = LEF × LM = 1 breach/month × $7.5M = $7.5M/month or $90M annual risk

Specific calculation: P(insider\_breach) = P(employee\_compromised) × P(exploit\_successful) × P(detection\_evasion)

Using data:

* Organizations average 6.3 malicious insider events annually
* 56% of organizations experienced insider incidents
* P(exploit\_successful) = 0.70 (given known SQL injection with limited DB access)
* P(detection\_evasion) = 0.30 (with standard monitoring)
* Cost per incident: $715,366

Based on 2025 data: Annual Rate of Occurrence (ARO) = 6.3 incidents/year (average per organization) Single Loss Expectancy (SLE) = $715,366 per incident Annual Loss Expectancy (ALE) = $715,366 × 6.3 = $4,506,806

Monte Carlo simulation with 10,000 iterations using log-normal distribution (typical for cyber losses):

A log-normal distribution is used because cyber losses are inherently skewed - most incidents cause moderate damage, but rare catastrophic breaches can result in extreme losses (think Equifax at $1.4B or Target at $292M). Unlike a normal distribution, which is symmetric. This matches empirical breach data where the logarithm of loss amounts follows a normal distribution.

* 95th percentile loss: $8,500,000
* 99th percentile loss: $15,000,000
* Mean annual loss: $4.5M (aligned with industry average of $17.4M\* for organizations experiencing multiple incidents)
* Requires historical breach data and vulnerability lifecycle models for accurate projections
* This modeled mean is below some published figures (e.g., studies that report \~$17.4M annualized loss for large organizations experiencing multiple severe incidents).

This quantitative approach combines hard data with qualitative assessments for comprehensive cyber risk modeling.

For the Tech folks(an analogy), Chaos Engineering for Security

Netflix’s Chaos Monkey randomly terminates instances to test resilience. Apply this concept to security through controlled compromise simulation:

Borrowing the spirit of chaos engineering, we instead simulate attacks…

* Model different attack vectors weighted by probability.
* Test defense-in-depth effectiveness
* Identify cascade failure points.
* Quantify business impact across various breach scenarios.

This approach reveals how seemingly minor vulnerabilities can cascade into major incidents when combined with specific conditions that traditional vulnerability scanning misses.

### Secure Controls Framework (SCF)

SCF provides vendor-agnostic security standardization, essentially enterprise-grade system hardening that scales.

Technical Implementation Perspective

For penetration testers: SCF addresses the exact vulnerabilities you exploit during privilege escalation. Without proper hardening implementation, these become your attack vectors:

* Weak service configurations
* Excessive permissions
* Missing security patches
* Default credentials
* Unencrypted communications

#### The Scaling Challenge

Managing a single server is straightforward. Managing 100+ servers requires systematic standardization. Consider the Shark Tank parallel: when entrepreneurs pitch handcrafted products, investors immediately ask, “How will you scale?” The same principle applies to security manual configurations that work for 5 servers, but fail catastrophically at 500. Standardization transforms security into industrial-grade protection.

SCF provides 33 comprehensive security domains:

* Incident Response (IR): Automated detection and response procedures
* Access Control (AC): Identity and privilege management at scale
* System Security (SC): Hardening configurations across platforms
* Risk Management (RM): Continuous risk assessment processes
* Cloud Security (CS): Multi-cloud security orchestration

While ISO 27001 defines management system requirements and ISO 27002 provides implementation guidance with 93 controls, SCF offers more comprehensive coverage with 1,300+ prescriptive controls mapped to 100+ frameworks, making it particularly valuable for complex compliance environments and automation initiatives.

### Open Security Controls Assessment Language (OSCAL)

The Multi-Compliance Challenge at Scale

Consider the governance nightmare at enterprise scale: A company needs ISO 27001, SOC 2, PCI-DSS, and HIPAA compliance. Both ISO 27001 and SOC 2 require annual penetration testing. You don’t run two separate pentests; you run one and map the results to both frameworks. Simple enough for one company.

Now scale this to AWS’s reality: hundreds of subsidiaries (AWS, Twitch, Whole Foods, Ring, Zoox), each with different regulatory requirements across 200+ countries. The CISO must coordinate:

* Thousands of overlapping controls across frameworks
* Evidence collection from thousands of systems
* Audit reports for multiple regulators simultaneously
* Real-time risk assessment across this entire ecosystem

Combine this with the Cyber Risk Modeling Language, where each control failure cascades through the risk graph, and you see why manual compliance becomes impossible. A single pentest report might satisfy 15 different control requirements across 8 frameworks—but tracking this manually across AWS’s scale would require an army of compliance officers.

NIST’s OSCAL is fundamentally a standardization language for compliance data—it doesn’t automate compliance itself, but creates a common language that automation tools can understand (for the AI folks, think of it as an MCP server that is a medium to talk to multiple systems in a “common” language).

OSCAL Technical Architecture

OSCAL provides:

* Data model specification: A complete information architecture using the Metaschema framework, not just formats
* Multiple serializations: XML/JSON representations that are fully interoperable without data loss
* Semantic interoperability: Ensures consistent interpretation across different tools and platforms
* Control lifecycle management: Models for catalogs, profiles, implementation, and assessment phases

Benefits include:

* Automated mapping across \~10 major compliance frameworks currently (with expansion ongoing)
* Enables continuous compliance monitoring when integrated with automation tools
* Risk-to-control traceability matrices
* API-driven compliance reporting

OSCAL is a comprehensive data model specification with a Metaschema-based information architecture that enables “compliance as code.” While not an automation engine itself, it defines semantic models for the entire control lifecycle from catalog definition through assessment with lossless conversion between XML/JSON serializations, allowing any compliant tool to process and exchange control information consistently.

Comprehensive Technical Implementation Strategy

From an engineering perspective, this governance framework enables:

1. Automated Threat Modeling:

* Graph databases tracking system relationships
* Attack path analysis using Markov chains over attack graphs, Bayesian networks, or specialized AND/OR graph algorithms with CVSS-based probabilistic scoring
* Real-time risk scoring based on threat intelligence feeds

2. Continuous Risk Quantification:

* Stream processing of security events
* Bayesian inference for threat probability updates

3. Control Validation Automation:

* Infrastructure-as-code security policies
* Continuous compliance scanning
* Automated remediation workflows

4. Integrated GRC Platform:

* OSCAL-based control libraries
* Automated evidence collection
* Real-time compliance dashboards

5. Chaos Security Engineering:

* Controlled breach simulations
* Blue team/red team automation
* Resilience scoring metrics

### Conclusion

The convergence of financial risk modeling, automation, and security engineering creates a governance framework that scales with modern infrastructure complexity. This approach transforms security from a cost center to a risk-aware business enabler, quantifying, automating, and continuously improving organizational resilience.







