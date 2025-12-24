  
**EC2 →**  
**UserData:**

* EC2 user data script runs with root user  
* Only runs at first boot by default

**Security Group:**

* Control how traffic is allowed in or out instance  
* Security group : **only ALLOW**  
* Security group : can reference IP or Security group

If application not accessible (time out) then security group issue  
Connection refused \- application error or not launched

**Purchasing Option:**

* On-Demand: pay by hr \- short and unpredictable workload   
* Reserved Instances: 1-3 year commitment 75% discount  
* Spot Instances: Bid on unused capacity up to 90% \- for batch jobs → you define max spot price if the one picked its more → 2min period to shut it down  
* Dedicated Hosts: Physical server dedicated u- (purchasing option on-demand or reserved)  
* Saving Plan: Flexible pricing model (1 to 3 year)  
* Dedicate instance: runs on a hardware for me \- may share hardware with others instances same account 

**Security group:**

* Virtual firewalls controlling inbound/outbound traffic  
* Stateful \-   
  * If an inbound rule allows a client to connect, then the response traffic from EC2 → client is automatically allowed back out.  
  * If an outbound rule allows EC2 to connect to something, then the response from that target → EC2 is automatically allowed back in.  
* Only ALLOW no DENY rule

NACLs are stateless means If an inbound rule allows a client to connect, then the response traffic from EC2 → client is automatically not allowed back out.

**Storage:**  
EBS (data persist) lost in instance store(epheral storage)  
These volumes persist after instance termination  
EBS- locked to AZ  
You can reattach them to other instances

* Attached one instance at time  
* Exist independently of instance lifecycle  
* Ephemeral storage (lost when instance is stops)  
* EBS snapshot can be copied across region  
* EBS volumes support both in-flight encryption and encryption at rest using KMS

Instance Store (Ephemeral Storage):  
THIS is what you lose when EC2 stops/terminates

Monitoring:

* CloudWatch metrics (CPU, Network, Disk, Status checks)  
* Default: 5-minute intervals  
* Detailed monitoring: 1-minute intervals (extra cost) \- aws ec2 monitor-instances \--instance-ids i-1234567890abcdef0  
* Custom metrics can be sent via CloudWatch agent  
* High-resolution metric every 1 second \- api PutMetricData with high res cloud watch alarm for every 10s

**Limits:**

* 20 On Demand Instances per region (can be increased)  
* Spot Instances interrupted with 2 min warning   
* Cant change instance type whilst running needs to be stopped

Networking 

Each instance gets a private IP in your VPC. 

Elastic IPs are static public IPs that persist across stops. 

Elastic Network Interfaces (ENIs) can be attached/detached and moved between instances.

 Enhanced Networking provides higher bandwidth and lower latency. 

Placement Groups optimize for different needs (where you want instances to be): 

* Cluster (low latency)- packs in one AZ \- for big data job  
* Spread (high availability)- across different hardware max 7 Per az  
* Partition (distributed applications).- used for cassandra kafka 

**Scaling policy Autoscaling group**

AMI

* When the new AMI is copied from Region A into Region B, it automatically creates a snapshot in Region B because AMIs  
* Copying an Amazon Machine Image (AMI) backed by an encrypted snapshot cannot result in an unencrypted target snapshot  
* You can share an Amazon Machine Image (AMI) with another AWS account  
* You can copy an Amazon Machine Image (AMI) across AWS Regions

**Auto Scaling policies (target tracking, step scaling, scheduled)**

Dynamic Scaling:

* Target Tracking \- Maintains metric → average CPU to stay 40%  → scales to keep performance and keep doing it when performance goes up and down  
* Step Scaling \- scales by different amount when threshold crossed   
* Simple \- scales by fixed amount when threshold crossed → with a cooldown 5 min 300s- application not be able to react quickly

**Schedule Scaling:**

* Schedule: Anticipated scaling based on usage pattern \- like increase capacity to 10 at 5pm

Application Load Balancer:

Client 12.34.56.78 → ALB → (Load Balancer IP) → EC2

* To see the client IP X-Forwarded-For  
* To see the client Port X-Forwarded-Port/X-Forwarded-Proto

Network Load Balancer:

* Forward TCP \+UDP to your instance   
* Million request/s  
* One static IP per AZ  
* Can talk to EC2/ IP /ALB

www → TCP+Rule → NLB →TCP EC2

Gateway Load Balancer:

* Deploy Scale manages 3rd party network virtual appliances  
* Talk to EC2 or IP addresses  
* GENEVE protocol 6081

Cross Zone Load balancing: balances all EC2 in all zones

**EC2 Hibernate (for stop):**

* Instance boot faster  
* When services takes time to initiate  
* Root volume EBS  
* Basically RAM gets written to EBS root volume so cpu resume where it started 

recover Amazon EC2 instances

* ✅ Use EBS-backed instances

* ✅ Enable DeleteOnTermination \= false for important volumes

* ✅ Take regular EBS snapshots

* ✅ Create AMIs before major changes

* ✅ Use Elastic IP if IP stability matters

* ✅ Enable EC2 Auto Recovery (CloudWatch)

Automatic recovery is triggered only by a System Status Check failure, not an Instance Status Check failure.

After recovery, the **EC2 instance keeps the same instance ID, private IP address, Elastic IP, and attached EBS volume**s.

For EC2 automatic recovery:

* ✅ Elastic IP addresses are retained  
* ❌ Auto-assigned public IPv4 addresses are NOT retained

public IPv4 address: Elastic IP

"Which component is changing the private IP address of EC2 instance E1 into a public IP address so it can communicate with the internet?"

VPC1 → IGW →S1 → E1 → NAT

So E1 sits with IGW \+NAT \= so public subnet S1

**Internet Gateway (I1)**

