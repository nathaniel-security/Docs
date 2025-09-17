# Presence-Aware Infrastructure: My Lab Knows When I'm Home (and Saves Me \$$$)

## Building the Ultimate R\&D Cloud Lab: How I Achieved Real-Time Cost Control and Zero-Waste AWS Experimentation <a href="#building-the-ultimate-rd-cloud-lab-how-i-achieved" id="building-the-ultimate-rd-cloud-lab-how-i-achieved"></a>

**TL;DR**: Built a sophisticated AWS R\&D environment with real-time cost tracking ($0.01/day for visibility), automated shutdown systems based on sleep detection, and seamless hybrid cloud integration. The result? Cost visibility + intelligent automation = zero waste and maximum experimentation velocity.

### The R\&D Cost Problem Every Technical Leader Knows <a href="#the-rd-cost-problem-every-technical-leader-knows" id="the-rd-cost-problem-every-technical-leader-knows"></a>

Let's be honest,I traditional R\&D labs suffer from two critical flaws that'll make your wallet weep:

* **No cost visibility** until the monthly AWS bill lands like a financial haymaker
* **Always-on infrastructure** burning cash 24/7, even when you're sleeping

I decided to solve this properly. No more monthly surprises, no more "oops, I left that GPU instance running for three weeks" moments. (I unfortunately know the pain)

### The Solution: Real-Time Cost Intelligence That Actually Works <a href="#the-solution-real-time-cost-intelligence-that-actu" id="the-solution-real-time-cost-intelligence-that-actu"></a>

### Daily Cost Tracking: The $0.01 Investment That Saves Thousands

Here's the game-changer: **I know exactly what my experiments cost the day after I run them**. Using the AWS Cost Explorer API, my system delivers:

* **Immediate feedback loop** on experimental costs
* **Daily spending attribution** to specific tests and infrastructure changes
* **Proactive budget management** instead of monthly financial surprises

The monitoring itself costs roughly $0.01 per day, an acceptable overhead that pays for itself within hours of preventing a single oversized experiment.

### Why This Beats Every Alternative

**Traditional Setup**: Spin up instances → Run experiments → Hope for the best → Get shocked by the monthly bill

**My Smart R\&D Lab**: Daily cost reports → Instant experiment attribution → Sleep-based automation → 60-80% cost savings

The difference? **Real-time awareness creates behavioural change**. When you see the daily cost impact of your experiments, you naturally optimise.

### The Architecture: Hybrid Intelligence Done Right <a href="#the-architecture-hybrid-intelligence-done-right" id="the-architecture-hybrid-intelligence-done-right"></a>

### Multi-Location Home DC Foundation

My setup starts with a distributed home data centre (please, i really, really want to call them a DC even tho they are not) across 4 physical locations:

* **ARM hardware** for lightweight processing
* **IoT and industrial systems** for specialised testing
* **High-performance CPU/RAM** for compute-intensive workloads
* **GPU hardware** for GPU workloads

### AWS Cloud Integration via WireGuard S2S VPN

The home infrastructure connects securely to AWS through WireGuard site-to-site VPN, enabling:

* **Secure data extraction** from home lab to AWS for processing
* **Private workload testing** without public exposure
* **Hybrid analytics** process home-generated data using AWS services

### VPC Architecture: Security Without Complexity

**Public Subnet**:

* Bastion host (auto-managed based on presence)
* NAT Gateway for private subnet access
* Security groups locked to trusted IPs only

**Private Subnet**:

* Development and testing instances
* Database resources
* No direct internet access, everything routes through NAT

### The Automation That Changes Everything <a href="#the-automation-that-changes-everything" id="the-automation-that-changes-everything"></a>

### Presence Detection: Your Lab Knows When You're Home

The system detects my presence through **WiFi network monitoring** when my phone connects to the home WiFi, the automation triggers. This approach is:

* **Zero battery impact** (uses existing WiFi connection)
* **Instant detection** (phone connects within seconds of arriving)
* **100% reliable** (can't fake being home unless you're actually there)

### Sleep Detection: Smart Resource Management

Here's where it gets sophisticated. The system monitors network activity patterns to detect sleep cycles:

* **Network activity monitoring** of phone and laptop usage
* **Pattern recognition** when activity drops below the working baseline
* **Time-based logic** (only between 20:00-04:00)
* **Multi-device correlation** to avoid false positives
  * It uses my phone and laptop in an OR condition&#x20;
    * If any one of the devices (laptop or phone ) is active means I am awake.&#x20;
  * This works in a 30-minute interval to decide if I am awake
* If any of my servers in any of these locations are talking to the server in AWS, it (in the 2 VPC) it keeps the AWS environment active, since I might be running an experiment&#x20;
  * I am yet to implement this function&#x20;
  * The idea is that the AWS env is meant for testing ideas, or if there is an active connection to them means I might be running some kind of test or taking logs, or something&#x20;
  * The plan to do this is similar to how I built my WireGuard tracking system&#x20;
    * It basically checks \`wg show\` and sees the networks transfer data

### Database-Driven Configuration

Similar to Windows Registry for key-value management, I use a database to store:

* Instance lists for automated shutdown
* Sleep detection thresholds
* Cost alert parameters
* Environment-specific configurations

This enables **runtime parameter adjustments** without code changes.

### Why n8n Crushes Lambda for R\&D Automation 🚀 <a href="#why-n8n-crushes-lambda-for-rd-automation" id="why-n8n-crushes-lambda-for-rd-automation"></a>

**Lambda Reality Check**: Per-invocation pricing + cold starts + debugging nightmares

**n8n Advantages**:

* **Fixed cost** (it's self-hosted, you can't beat that AWS) vs per-execution pricing (99% cheaper for frequent workflows)
* **Visual workflow debugging** that actually helps troubleshoot
* **No cold starts** for consistent performance
* **Webhook reliability** without the serverless complexity

For daily cost notifications and automation workflows, n8n delivers enterprise-grade functionality at a fraction of Lambda's cost.

### The Business Impact: Numbers That Matter 📊 <a href="#the-business-impact-numbers-that-matter" id="the-business-impact-numbers-that-matter"></a>

### Cost Optimisation Results

| Metric                 | Traditional Setup | Smart R\&D Lab       | Savings             |
| ---------------------- | ----------------- | -------------------- | ------------------- |
| 24/7 running instances | Always on         | Sleep-based shutdown | **60-80%**          |
| Cost visibility        | Monthly surprises | Daily tracking       | **Immediate**       |
| Management overhead    | Manual processes  | Automated workflows  | **25+ hours/month** |
| Workflow costs         | Lambda functions  | n8n automation       | **99% cheaper**     |

### Technical Advantages

**Hybrid Intelligence**: Home lab handles persistent workloads, AWS scales for experiments

**Presence-Based Resource Management**: Infrastructure mirrors your actual work patterns

**Real-Time Cost Attribution**: Every experiment's financial impact is immediately visible

**Secure Network Integration**: Private connectivity without compromising security

### Implementation Insights: What Actually Matters <a href="#implementation-insights-what-actually-matters" id="implementation-insights-what-actually-matters"></a>

### Cost Monitoring Setup

1. **Enable Cost Explorer API** with minimal permissions
2. **Deploy n8n workflow** for daily cost tracking
3. **Configure Discord webhooks** for instant notifications
4. **Set up attribution logic** to track experiment impacts

### Infrastructure Automation

1. **Deploy VPC** with public/private subnet architecture
2. **Configure the bastion host** with presence-based auto-startup
3. **Set up sleep detection** automation with database configuration

### Security Implementation

* **IP-restricted access** from home DC locations
* **Audit-ready connections** for compliance requirements(yeah, what better than saying I have audit logs for my homelab, maybe even get an ISO27001, cause why not)

### Key Takeaways for Technical Leaders <a href="#key-takeaways-for-technical-leaders" id="key-takeaways-for-technical-leaders"></a>

**Cost Visibility = Control**: The $0.01/day investment in cost monitoring prevents $1000+ monthly surprises

**Automation Prevents Waste**: Sleep-based shutdown saves 60-80% on compute costs without impacting productivity

**Hybrid Approach Maximises Value**: Home lab for persistence, AWS for scalability, VPN for security

**n8n > Lambda for Workflow Automation**: Fixed pricing, visual debugging, and no cold starts win for R\&D workflows

**Presence Detection Eliminates Manual Management**: Your infrastructure should adapt to your work patterns, not the other way around

### The Bottom Line 🏆 <a href="#the-bottom-line" id="the-bottom-line"></a>

This architecture transforms R\&D experimentation from a cost centre into a controlled, measurable, and highly automated technical asset.

The system knows when I'm home, when I'm sleeping, and exactly what each experiment costs. Infrastructure starts up as I walk in the door and shuts down when I'm inactive, all while maintaining enterprise-grade security and providing immediate cost feedback.

**Result**: Maximum experimentation velocity with zero waste and complete cost transparency.

Your R\&D environment should serve your curiosity, not drain your budget. This setup delivers both.

***

### Enterprise Application: Smart Cost Control for Development Teams 🏢 <a href="#enterprise-application-smart-cost-control-for-deve" id="enterprise-application-smart-cost-control-for-deve"></a>

**For businesses with development teams**, this system automatically shuts down infrastructure when developers aren't actively using it. The beauty lies in the **database-driven override mechanism**—unless a developer explicitly changes the shutdown prevention value to `1` In the database, the system will power down non-critical services when the team isn't in the office.

This creates a **default-to-savings** approach while giving developers full control when they need 24/7 uptime for critical testing or deployment cycles. No more forgotten instances burning budget overnight, yet no friction when teams genuinely need persistent infrastructure.

***

_Want to build something similar? Start with cost monitoring, add presence detection, and let automation handle the rest. The future of R\&D infrastructure is intelligent, hybrid, and cost-aware._
