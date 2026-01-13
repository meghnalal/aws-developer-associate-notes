!!! example "Topics according to exam"
    For AWS Developer exam: the main thing to know is:

    - Not many questions on the developer one

    For AWS Solution Architect exam: the main thing to know is:
    
    - This is a heavy topic for AWS solution architect 
**<span style="color: #0057FF;">AWS Snowball</span>**

* Helps migrate petabytes of data   
* Highly secure portable device   
* **Offline** devices to perform data migration (if it takes more than 1 week to transfer data over network, use Snowball)

**How it works:**

1. You request a Snowball device through the AWS Console  
2. AWS ships the physical device to **your location** (your data center, office, etc.)  
3. You connect it to your local network and copy your data onto the device  
4. You ship the device back to **an AWS facility/data center**  
5. AWS technicians connect the device and automatically upload your data directly into your S3 bucket  
6. AWS securely erases the device after confirming successful transfer  
7. You get a notification when the data is available in S3

**Flow:**
```
Client → AWS Snowball → Ship → AWS Snowball → Import/Export → Amazon S3
```

!!! info "Example"
    Transferring 100TB over 1Gbps takes 12 days

**Snowball Types:**

* **Snowball Edge Storage Optimized:** 80TB  
* **Snowball Edge Compute Optimized:** 42TB (more RAM and CPU)

**Use Cases:**

* **Data Migration:** Moving petabytes of data to AWS when network transfer is impractical or too expensive. Think large-scale cloud migrations, data center shutdowns, or initial data loads  
* **Edge Computing:** Running compute workloads in remote locations with limited connectivity. The device can run EC2 instances and Lambda functions locally

**Snowball vs Snowball Edge:**

* **Snowball:** Just storage transfer  
* **Snowball Edge:** Process data whilst it's being created on edge location for processing data, ML, or transcoding

!!! warning "Important: Snowball into Glacier"
    Snowball **CANNOT** import to Glacier directly
    
    You need to use S3 first with S3 lifecycle policy:
```
    Snowball → S3 → Amazon Glacier
```

---

**<span style="color: #0057FF;">File Storage (EFS + FSx)</span>**

**EFS (Elastic File System):**

* AWS's own simple file storage for Linux  
* Serverless

---

**<span style="color: #0057FF;">Amazon FSx (File Storage Types)</span>**

**FSx for Windows File Server:**

* Protocol: SMB  
* OS Support: Windows + Linux (via SMB)  
* Use Case: Windows workloads, Active Directory integration, Microsoft apps

**FSx for NetApp ONTAP:**

* Protocol: NFS, SMB, iSCSI  
* OS Support: Windows + Linux (native both)  
* Use Case: NetApp users, multi-protocol needs

**FSx for Lustre:**

* Protocol: Lustre  
* OS Support: Linux  
* Use Case: HPC (High Performance Computing), ML, big data processing, parallel processing  
* Pay for what you use  
* Provides the ability to both process the 'hot data' in a parallel and distributed fashion as well as easily store the 'cold data' on Amazon S3  
* **Only one that supports direct import**

!!! warning "Important"
    For the FSx options that **don't support direct import**, the data **must land somewhere else first**, and then be copied into FSx.

**FSx for OpenZFS:**

* Protocol: NFS  
* OS Support: Linux  
* Use Case: High-performance storage for Linux workloads like databases (MySQL, Oracle)  
* Ongoing cost  
* High IOPS

---

**<span style="color: #0057FF;">AWS Storage Cloud Native</span>**

* **Block:** EBS, EC2 Instance Store  
* **File:** EFS, FSx  
* **Object:** S3, Amazon Glacier

---

**<span style="color: #0057FF;">Services to Connect with On-Premises</span>**

* File Gateway  
* Volume Gateway  
* Tape Gateway

---

**<span style="color: #0057FF;">AWS Storage Gateway</span>**

Bridge between on-premises and cloud data

It's a **hybrid cloud storage** service that connects your on-premises environment to AWS storage (S3, EBS, Glacier).

**Why use it?**

* Disaster recovery  
* Backup   
* On-prem cache for low latency

**Types:**

**S3 File Gateway (File Storage):**

* Files stored as object storage  
* Makes S3 look like a normal file share (NFS or SMB)  
* Your on-premises servers see it as a network drive

**Use Cases:**

* Store files locally but also in cloud  
* Migrate file shares to cloud

**How Caching Works:**

1. **On-premises server requests "file_x"**  
2. **File Gateway checks:** "Do I have this in my local cache?"
   * If YES → Return it immediately (fast!)  
   * If NO → Fetch it from S3 (slower)  
3. **File Gateway downloads file_x from S3**  
4. **Stores it in local cache** for future requests  
5. **Returns file to the server**

When cache fills up, **least recently used (LRU)** files are removed

Can transition to S3 Glacier using lifecycle policy

**Volume Gateway (Block Storage):**

On-prem block storage that gets backed by S3/EBS

**Stored Volumes:**

* Entire dataset stored on-premises  
* Flow: On-prem iSCSI disk → Volume Gateway (cache ALL data) → S3 (EBS snapshot)  
* Asynchronously backed up to AWS (EBS snapshots in S3)  
* **Key point:** Primary data is ON-PREMISES, backup in cloud

**Cached Volumes:**

* Entire dataset stored in S3  
* Only frequently accessed data cached on-premises  
* Flow: On-prem iSCSI disk → Volume Gateway (cache hot data) → S3  
* **Key point:** Primary data is IN CLOUD, cache on-premises

!!! info "Exam Keywords"
    Block storage, iSCSI, EBS snapshots, volumes
    
    Uses **iSCSI protocol** (block-level)

**Tape Gateway:**

* Replace physical tapes  
* Company uses tape backups and wants to move to cloud  
* Long-term archival

---

**<span style="color: #0057FF;">AWS Transfer Family</span>**

Lets you transfer files to/from AWS (S3 or EFS) using **old-school file transfer protocols** that companies already use.

**Problem it solves:** Many companies have legacy systems that use FTP, SFTP, or FTPS to transfer files

---

**<span style="color: #0057FF;">AWS DataSync</span>**

AWS DataSync = Automated data transfer and synchronization service

It's designed to **move large amounts of data** quickly and securely between on-premises and AWS, or between AWS services.

**Think of it as:** "Moving truck for data"

**Problem it solves:** You need to migrate TBs/PBs of data to AWS or sync data regularly

**What Can DataSync Connect?**

**Sources (FROM):**

* On-premises file servers (NFS, SMB)  
* On-premises object storage  
* AWS storage services  
* Other cloud providers

**Destinations (TO):**

* **Amazon S3** (any storage class)  
* **Amazon EFS** (Elastic File System)  
* **Amazon FSx for Windows File Server**  
* **Amazon FSx for Lustre**  
* **Amazon FSx for OpenZFS**  
* **Amazon FSx for NetApp ONTAP**