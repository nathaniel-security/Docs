# AWS NACL vs Security Group: The Practical Guide (With Console Walkthroughs)

Security Groups are stateful bouncers on the instance/ENI. NACLs are stateless speed bumps at the subnet edge. That is the core idea. The rest is details, and you are here for the details.



## TL;DR

| Feature               | Security Group                    | NACL                                         |
| --------------------- | --------------------------------- | -------------------------------------------- |
| Level                 | Instance/ENI                      | Subnet                                       |
| Stateful or Stateless | Stateful                          | Stateless                                    |
| Rule Types            | Allow only                        | Allow and Deny                               |
| Applies To            | ENIs/Instances                    | Entire Subnet                                |
| Rule Evaluation       | All rules are evaluated as a set  | Rules are processed by number (lowest first) |
| Defaults              | Inbound denied, outbound allowed  | Default NACL allows all; you can change it   |
| Typical Use           | App-level firewalling             | Broad subnet filtering and quarantine        |
| Logging               | VPC Flow Logs (via ENI or Subnet) | VPC Flow Logs (via Subnet)                   |
| Console Path          | EC2 -> Security Groups            | VPC -> Network ACLs                          |

If you only remember one thing:&#x20;

* SG = stateful allow-lists on ENIs;&#x20;
* NACL = stateless allow+deny lists on subnets with ordered rules.

### Why you should care

* Reality check:&#x20;
  * SGs do not support deny.&#x20;
    * NACLs do.&#x20;
  *   SGs track connections;&#x20;

      * NACLs do not.&#x20;

      NACL rule order matters

      * SG rule order does not.
* Real world: Subnet-wide quarantine, hot IP blocks, blast-radius control, and keeping your SGs clean all hinge on knowing which lever to pull.
* Performance note: NACLs are checked per packet; SGs are connection-aware. If you are pushing very high packet rates, the difference shows up in behaviour and troubleshooting.

### See it yourself: AWS Console Walkthroughs

#### Security Groups

1. Open AWS Console -> EC2 -> Security Groups.
2. Click a group. Inspect Inbound rules and Outbound rules tabs.
3. Click the References/Attached resources tab to see ENIs/instances using this SG.
4. Try to add a Deny. You cannot. SGs are allow-only.
5. Housekeeping: Filter for groups with zero attached network interfaces. Those are candidates for cleanup.

#### Network ACLs

1. Open AWS Console -> VPC -> Network ACLs.
2. Pick a NACL. Inspect Inbound rules and Outbound rules. Note the rule numbers and the implicit deny at the end.
3. Check the Subnet associations tab to see which subnets inherit these rules.
4. Create a Deny rule for a test source IP and give it a small rule number (for example 100) so it wins.
5. Housekeeping: Filter for NACLs with zero subnet associations. Those are usually safe to archive or delete after review.

Pro tip: Make one small change at a time, then test. NACL mistakes are easy to make and hard to debug if you pile on changes. (been here)

### Hands-on mini lab (15-30 min)

Goal: Experience stateful vs stateless behavior and allow vs deny in a safe sandbox.

#### Setup

* One VPC with two public subnets: web-subnet and admin-subnet.
* An EC2 instance in web-subnet (call it web1).
* An EC2 instance in admin-subnet (call it admin1).
* An Internet Gateway and route tables so both are reachable.

#### Security Groups

* SG-web: inbound allow TCP 80 from 0.0.0.0/0, outbound allow all.
* SG-admin: inbound allow TCP 22 only from your public IP, outbound allow all.

#### NACLs

* NACL-web: allow inbound 80 from 0.0.0.0/0, allow ephemeral response ports outbound. No explicit deny.
* NACL-admin: add a deny inbound rule for your home IP block as a test (for example, rule 100 deny 203.0.113.0/24), then allow the rest.

#### Tests

1. From your laptop, curl http://web1-public-ip. Should work.
2. From your laptop, ssh admin1. If your IP matches the deny in NACL-admin, you should be blocked regardless of SG.
3. From admin1, curl http://web1-private-ip. This should work if SGs allow it. If you add a NACL deny between the subnets, it will fail even if SGs allow.
