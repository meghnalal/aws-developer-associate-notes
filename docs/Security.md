!!! example "Topics according to exam"
    For AWS Developer exam: the main thing to know is:

    - ACM
    - WAF
    
    For AWS Solution Architect exam: the main thing to know is:
    
    - Macie
    - WAF
    - Shield
    - Guard Duty
    - Inspector

**<span style="color: #0057FF;">ACM (Certificate Manager)</span>**

* Provision, manage and deploy TLS certificates  
* Integration with: Application Load Balancer, CloudFront, API Gateway  
* Cannot use ACM with EC2 (need load balancer in front)  
* Automatic renewal  
* You can generate certificate outside and import but no automatic renewal

!!! info "Flow"
    API Edge-Optimized with CloudFront → link certificate → us-east-1 region API Gateway → link certificate → same region as API stage

---

**<span style="color: #0057FF;">WAF (Web Application Firewall)</span>**

Layer 7 HTTP  

User → WAF (checks request) → CloudFront/ALB/API Gateway → Your App  
          ↓  
     Blocks bad requests here  

**WAF CAN'T WORK WITH:**

* Network Load Balancer  
* Classic Load Balancer  
* EC2 instances directly

**WAF can inspect:**

* HTTP headers (User-Agent, Referer, etc.)  
* HTTP body content  
* Query strings  
* URI paths  
* Cookies  
* HTTP methods (GET, POST, etc.)

**Examples:** 

* Block requests with `' OR 1=1--` in query string (SQL injection)  
* Block if body size > 8KB  
* Protect web app from SQL injection  
* Block all traffic except from US and Canada

Because WAF works at layer 7, it needs ALB in front of it. Since ALB doesn't have static IP, you can put Global Accelerator in front.  
Then add the WebACL in the same region where ALB is located.

---

**<span style="color: #0057FF;">AWS Shield</span>**

Protects from DDoS attacks:

* Many requests at same time  
* Protects at layer 3/4  
* Protects against sophisticated attacks on EC2, ELB, CloudFront, Route 53

**AWS Shield Advanced:** 

* Advanced DDoS mitigation service  
* 24/7 DDoS Response Team (DRT)  
* Automatically creates WAF rules for you

---

**<span style="color: #0057FF;">AWS Firewall Manager</span>**

Manage rules across all accounts in AWS Organization  
You set security policy (all firewalls managed in 1 place)  
Firewall automatically applies rules to new resources without you doing anything  
Used with WAF for comprehensive protection

**Manages:**

* WAF  
* Shield  
* Security groups  
* Network Firewall  
* Route 53 Resolver DNS Firewall

---

**<span style="color: #0057FF;">Amazon GuardDuty</span>**

Uses Machine Learning algorithms, anomaly detection, 3rd party data  
Intelligent threat discovery to protect your AWS Account

!!! info "Flow"
     VPC Flow Logs
     CloudTrail Logs     → GuardDuty → EventBridge → SNS/Lambda
     DNS Logs


---

**<span style="color: #0057FF;">Amazon Inspector</span>**

Automated security assessment service

* EC2 instances  
* Container images (ECR)  
* Lambda functions  
* Risk score shows vulnerability severity

---

**<span style="color: #0057FF;">Amazon Macie</span>**

Managed data security and data privacy service  
Uses machine learning to discover and protect sensitive data  
Identifies and alerts on sensitive data (PII, credentials, etc.)

!!! info "Flow"
     S3 → analyze → Macie → EventBridge (notify)

---

**<span style="color: #0057FF;">Scenario: DDoS Attack Mitigation</span>**

After investigation, the team suspects the application is being targeted by distributed denial-of-service (DDoS) attacks coming from a wide range of IP addresses. The team needs a solution that provides DDoS mitigation, detailed logs for audit purposes, and requires minimal changes to the existing architecture.

**Solution:**  
Subscribe to AWS Shield Advanced to gain proactive DDoS protection. Engage the AWS DDoS Response Team (DRT) to analyze traffic patterns and apply mitigations. Use the built-in logging and reporting to maintain an audit trail of detected events.

**Alternative approach:**  
Placing Amazon CloudFront in front of the ALB and using AWS WAF for traffic filtering can improve latency and offer some DDoS protection, especially at Layer 7. However, this introduces architectural changes that may require reworking DNS.