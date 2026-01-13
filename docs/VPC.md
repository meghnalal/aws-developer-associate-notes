!!! example "Topics according to exam"
    For AWS Developer exam: the main thing to know is:

    - Not many questions on the developer one

    For AWS Solution Architect exam: the main thing to know is:
    
    - Very important for Solution Architect
    - Transit Gateway
    - Virtual Private Gateway
    - NAT 
    - Direct connect 

**<span style="color: #0057FF;">CIDR (Classless Inter-Domain Routing)</span>**

IP address allocation method   
`0.0.0.0/0` → all IPs

**Base IP:**  
Represents IP contained in range: `10.0.0.0`

**Subnet Mask:**  
Defines `/8`, `/16`, `/24`, `/32`

**CIDR IP Count:**

* `/32` → 1 IP  
* `/31` → 2 IPs  
* `/30` → 4 IPs  
* `/29` → 8 IPs  
* `/28` → 16 IPs  
* `/27` → 32 IPs  
* `/26` → 64 IPs  
* `/0` → all IPs

---

**<span style="color: #0057FF;">Public vs Private IP</span>**

**Private IP:**

* Local network only  
* No internet routing  
* Need NAT for internet access  
* IP address used inside private network (home, office, LAN)  
* Can only allow certain values   
* AWS default VPC `/16`  
* Home network `/16`

**Public IP:**

* Global   
* Globally unique  
* Internet routing   
* Assigned by ISP  
* No NAT needed  
* Example: `8.8.8.8` (Google DNS)

---

**<span style="color: #0057FF;">Default VPC</span>**

* New instances are launched in default VPC   
* Default VPC has internet connectivity  
* All instances inside have public IPv4 address 

---

**<span style="color: #0057FF;">VPC (Virtual Private Cloud)</span>**

Your own isolated network within AWS where you can launch resources

**Core VPC Components:**

**CIDR Block:**

* You define an IP address range with CIDR

**Subnet:**

* These divide VPC into smaller networks - each network exists in a single Availability Zone and gets a portion of the CIDR range   
* **Public subnet** has direct internet access via Internet Gateway 

**Public subnets need three things:**

1. Internet Gateway attached to the VPC  
2. A route to the IGW (`0.0.0.0/0`)  
3. Public IP or Elastic IP on instances

**Private subnets:**

* Don't have internet access   
* Private subnets needing internet access require:  
  1. NAT Gateway in a public subnet  
  2. Appropriate routing

!!! info "Important"
    AWS reserves 5 IPs in each subnet (first 4 and last 1)  
    
    If you need 29 IP addresses: use `/27` (32 IP addresses, 32 – 5 = 27 < 29)

---

**<span style="color: #0057FF;">Route Table</span>**

* Rules where network traffic is directed   
* Each subnet must be associated with one route table    
* But one route table can be associated with multiple subnets 

Each table has:
* **Destination** - IP range to reach
* **Target** - where to send traffic

Most specific routing takes priority

**Example:**

| Destination | Target |
|-------------|--------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | igw-xxxxx |

* Traffic going to `10.0.0.0/16` stays within the VPC (local)  
* Traffic going to **any other destination** (`0.0.0.0/0` means "everything else") goes to the IGW

!!! info "Important"
    If you need internet connectivity, then IGW and NAT Gateway become essential components for your architecture.

---

**<span style="color: #0057FF;">Internet Gateway (IGW)</span>**

* Allows communication between resources in VPC and internet   
* One IGW per VPC   
* Resources in public subnet need **both IGW and route table**

---

**<span style="color: #0057FF;">Bastion Host</span>**

SSH to Public Subnet EC2 (The Bastion Host)  
Then Bastion SSH to private subnet EC2

---

**<span style="color: #0057FF;">NAT Gateway</span>**

* Enables instances in private subnet to access the internet   
* High availability NAT Gateway in multi-AZs  
* Flow: Private Subnet → NAT Gateway → Internet Gateway  
* No security group to manage  
* Deploy multiple NAT Gateways in multiple AZs for redundancy

---

**<span style="color: #0057FF;">NAT Instance (outdated but still in exam)</span>**

* EC2 instances you manage yourself in public subnet   
* Need to manage Security Groups   
* Private subnets talk to it   
* Must have Elastic IP attached (EIP)  
* Route traffic configured from private subnet 

---

**<span style="color: #0057FF;">Security</span>**

**Security Groups:**

* Stateful - return traffic automatically allowed  
* At instance level  
* Allow rules only

**Network ACL (NACL):**

* Subnet level  
* Firewall at the subnet level  
* Stateless - must specify both inbound and outbound rules   
* Number defines priority: Rule 100 ALLOW > Rule 200 DENY   
* Great way to block specific IP addresses

---

**<span style="color: #0057FF;">VPC Peering</span>**

VPC ←→ VPC (AWS only, cloud network)

* Connects two VPCs using private IP addresses - AWS network   
* **Non-transitive:** If A peers with B, and B peers with C, A cannot reach C  
* Works across regions and accounts

!!! question "Exam Scenario"
    "Connect two VPCs in different regions for disaster recovery" → **VPC Peering**

---

**<span style="color: #0057FF;">Transit Gateway</span>**

```Hybrid ←→ VPC and VPC ←→ VPC```

* Connects multiple **VPCs and on-premises** networks   
* Ideal when connecting 3+ VPCs  
* Transit Gateway handles all routing   
* No private physical connection needed

**Examples:**

!!! info "Flow"
    * On-Premises ──VPN──→ Transit Gateway ──→ Multiple VPCs, Multiple Accounts  
    * On-Premises ──Direct Connect──→ Transit Gateway ──→ Multiple VPCs, Multiple Accounts  
    * Account A Transit Gateway → Account B VPC attachment to A → Account C VPC attachment to A

---

**<span style="color: #0057FF;">VPN Connection (Site-to-Site)</span>**

```Hybrid ←→ VPC Only```

An **encrypted connection between your on-premises network and AWS** over the internet.

* Connects on-prem network to AWS over the internet using IPSec  
* You need Virtual Private Gateway on AWS side and Customer Gateway on your side   
* Bandwidth limited  
* Dependent on internet connection quality  
* Setup time: Hours or days  
* Cheaper than Transit Gateway

!!! question "Exam Scenario"
    "Company needs to quickly connect their data center to AWS with encryption" → **Site-to-Site VPN**

**Connection Types:**

**Connect to 1 VPC - Virtual Private Gateway:**
```
On-Prem (Customer Gateway) → VPN → Virtual Private Gateway (VGW) → Single VPC
```

**Connect many VPCs - Transit Gateway:**
```
On-Prem → VPN → Transit Gateway → Many VPCs (even across accounts)
```

**Complex network security - Software VPN:**
```
On-Prem → VPN (IPSec or SD-WAN) → EC2 VPN Appliance → VPC
```

---

**<span style="color: #0057FF;">Direct Connect</span>**

```Hybrid ←→ VPC Only```

* Dedicated physical connection from on-prem to AWS   
* Weeks to set up   
* Bypasses the public internet entirely  
* Best for 1 or 2 VPCs

!!! info "Important"
    Does **not encrypt traffic by default** → Use VPN over Direct Connect for encryption

!!! question "Exam Scenario"
    "Company transfers 500TB monthly and needs consistent performance" → **Direct Connect**

**Direct Connect Gateway:** If you want to setup Direct Connect to one or more VPCs in many different regions (same account)

**Connection Types:**

**Connect to 1 VPC - Virtual Private Gateway:**
```
On-Prem Router → Direct Connect → DX Router → Private Virtual Interface (VIF) → Virtual Private Gateway (VGW) → VPC
```

**Connect many VPCs - Transit Gateway:**
```
On-Prem Router → Direct Connect → DX Router → Transit Virtual Interface (VIF) → Direct Connect Gateway → Transit Gateway → Many VPCs
```

---

**<span style="color: #0057FF;">VPN over Direct Connect</span>**

Hybrid ←→ VPC Only

In case Direct Connect fails, you can set up a backup Direct Connect or Site-to-Site VPN

!!! question "Exam Scenario"
    "Company needs dedicated connection with end-to-end encryption" → **VPN over Direct Connect**

---

**<span style="color: #0057FF;">VPC Endpoints</span>**

``` Private VPC ←→ AWS Services ```

**You can connect to AWS services using private network from your subnet**  
No need for IGW or NAT Gateway

Allows private connection to AWS services without using the internet

**Gateway Endpoints:**

* Work with S3 and DynamoDB  
* Free  
* Gateway Endpoints don't work from on-premises networks connected via VPN or Direct Connect - they only work from within the VPC  
* Gateway Endpoints don't work across VPC peering connections or **on-prem traffic**

**Interface Endpoint (PrivateLink):**

* Use ENI (Elastic Network Interface)  
* Work with many AWS services  
* Not free  
* Preferred for access from on-premises or different VPC or different region

!!! info "Example: Lambda + SNS"
    You have Lambda functions in a private subnet that need to call SNS to send notifications. Your options:
    
    * Add a NAT Gateway ($0.045/hour + data transfer costs)  
    * Add an Interface Endpoint for SNS ($0.01/hour + data transfer costs)  
    * Interface Endpoint is actually cheaper here!

!!! info "Example: On-Premises + S3"
    Your on-premises data center connects to AWS via Direct Connect. You want to access S3 privately without going over the internet.
    
    **With Gateway Endpoint:** ❌ Won't work. On-premises traffic can't use Gateway Endpoints.
    
    **With Interface Endpoint:** ✅ Works perfectly. On-premises systems can resolve the Interface Endpoint's private IP and access S3 over Direct Connect.

!!! info "Important: VPC Peering"
    The application in VPC-A **cannot** use VPC-B's Gateway Endpoint for S3. Each VPC needs its own Gateway Endpoint.
    
    **With Interface Endpoints:** You can configure it to be accessible from peered VPCs (though you need to set up specific configurations with Route 53 and security groups).

---

**<span style="color: #0057FF;">AWS VPN CloudHub</span>**

CloudHub allows A ↔ B ↔ C to communicate with each other through AWS, not only to the VPC.

* Used for multiple **on-premises** branch offices only  
* You already use **VGW-based VPNs**

---

**<span style="color: #0057FF;">Lambda in VPC Accessing DynamoDB</span>**

DynamoDB is a public service

**Option 1:**
```
Private subnet Lambda in VPC → Public subnet NAT → Public subnet IGW → DynamoDB
```

**Option 2 (Better):**
```
Private subnet Lambda in VPC → Deploy VPC Gateway Endpoint → DynamoDB
```

---

**<span style="color: #0057FF;">VPC Flow Logs</span>**

Capture info about IP Traffic   
Helps monitor and troubleshoot connectivity issues 

**Look at ACTION field:**

**Incoming request:**

* Outbound/Inbound REJECT → NACL or Security Group  
* Inbound ACCEPT, Outbound REJECT → NACL  
* Outbound ACCEPT, Inbound REJECT → NACL

**Flow Log Destinations:**

* VPC Logs → CloudWatch Logs → CloudWatch Insights  
* VPC Logs → CloudWatch Logs → CloudWatch Alarm  
* VPC Logs → S3 Bucket → Amazon Athena

---

**<span style="color: #0057FF;">VPN Connection Components</span>**

**Virtual Private Gateway (VGW):**

* The **VPN endpoint on the AWS side**  
* Used for Site-to-Site VPN

**Customer Gateway (CGW):**

* The **VPN endpoint on your side (customer side)**

---

**<span style="color: #0057FF;">Connectivity Summary</span>**

**On-Premises to AWS (Hybrid Connectivity):**

* Site-to-Site VPN  
* AWS Direct Connect (DX)  
  * Types of Virtual Interfaces:  
    * Private VIF: Connects to VGW (for single VPC)  
    * Transit VIF: Connects to Transit Gateway (for multiple VPCs)  
    * Public VIF: For AWS public services (S3, DynamoDB, etc.)  
* Direct Connect + VPN (Hybrid)  
* AWS VPN CloudHub

**VPC to VPC Connectivity (Within AWS):**

* VPC Peering  
* Transit Gateway  
* AWS PrivateLink (VPC Endpoint Services)

---

**<span style="color: #0057FF;">Egress-Only Internet Gateway</span>**

* Only for IPv6  
* Like NAT Gateway but for IPv6

---

**<span style="color: #0057FF;">Minimizing Egress Traffic Network Cost</span>**

**Egress traffic:** Outbound traffic from AWS to outside  
**Ingress traffic:** From outside to AWS (typically free)

* Keep as much traffic within AWS as possible  
* AWS charges data transfer out of AWS (egress) differently depending on whether your traffic stays in the same region or crosses regions  
* Same region transfers are cheaper

---

**<span style="color: #0057FF;">Network Protection on AWS</span>**

We've seen:

* Network Access Control Lists (NACLs)  
* Amazon VPC Security Groups  
* AWS WAF (protect against malicious requests)  
* AWS Shield & AWS Shield Advanced  
* AWS Firewall Manager (to manage them across accounts)

**Want to protect entire VPC in a sophisticated way?**

→ **AWS Network Firewall**

* Supports 1000s of rules  
* Advanced protection for entire VPC