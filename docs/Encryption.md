**<span style="color: #0057FF;">Encryption Types</span>**

**Encryption in flight** 

* TLS/SSL - data encrypted before sending  
* Ensures protection from man-in-the-middle attacks

**Server-side encryption at rest**

* Data encrypted after being received by server   
* And decrypted before being sent  
* Encryption/decryption key must be managed somewhere

**Client-side encryption** 

* Encrypted and decrypted at client side  
* Server can't decrypt data 

---

**<span style="color: #0057FF;">AWS KMS (Key Management Service)</span>**

* Encryption for AWS services - it's most likely KMS  
* AWS manages keys for us   
* Enable audit with CloudTrail

**Symmetric Keys:** 

* Symmetric KMS Keys: A **single key** is used for **both encryption and decryption**  
* Key never leaves KMS unencrypted - must call the KMS API to use it   
* When you need to encrypt or decrypt, you send data to KMS

!!! info "Example: Database encryption"

**Asymmetric Keys:** 

* It has 2 Keys: Public (Encrypt) - exportable  
* Private (Decrypt) - kept in KMS  
* Use case: encryption outside AWS by users who can't use KMS API, Digital signing

Supports **RSA** and **ECC** key types.

!!! info "Example: IoT Devices Sending Data"
    You have thousands of IoT sensors in factories worldwide sending encrypted telemetry data to AWS. Each device has the public key embedded and encrypts data locally before transmission.
    
    Devices encrypt data using the public key (which is safe to distribute). Even if device is attacked, they can't decrypt the data.

---

**<span style="color: #0057FF;">Types of KMS Keys</span>**

* **AWS Owned:** SSE-S3, SSE-SQS, SSE-DDB  
* **AWS Managed Key:** free (aws/service-name)  
* **Customer managed key in KMS:** $1/month - KMS stores the CMK, and receives data from clients, which it encrypts and sends back  
* **Customer managed key imported:** $1/month

**Key Rotation:**

* **AWS Managed Key** - rotates every 1 year  
* **Customer managed key in KMS:** must be enabled - automatic or on-demand  
* **Imported Key:** only manual rotation possible

---

**<span style="color: #0057FF;">Copy Snapshot Across Accounts</span>**

* In the source account, modify your **KMS key policy to allow the destination account** to use it  
* Create a snapshot and share the snapshot with the destination account (add permission of destination account)  
* Copy snapshot and re-encrypt (decrypting and encrypting again) with your own KMS keys  
* Decrypt the snapshot using the source account's KMS key (this is why you needed to grant KMS permissions)  
* The KMS API calls being made:  
  * `kms:Decrypt` - To decrypt the encrypted data blocks   
  * `kms:DescribeKey` - To get key metadata   
  * `kms:CreateGrant` - To create temporary credentials if needed  
* Re-encrypt the data using the destination account's KMS key

!!! warning "Important"
    **They CAN'T have the same key** - KMS keys cannot be shared or transferred between accounts. Each KMS key belongs to exactly one AWS account.
    
    Even though you grant the destination account *permission to use* your key temporarily, they can't:
    
    * Own your key  
    * Keep using it long-term  
    * Control it (rotate, delete, modify policies)

---

**<span style="color: #0057FF;">KMS Multi-Region Keys</span>**

**Use cases:** global client-side encryption, encryption on Global DynamoDB, Global Aurora  
One primary key in one region → the other region gets the replica key   
Encrypt in one region, decrypt in another

---

**<span style="color: #0057FF;">Normal 4KB File Encryption/Decryption</span>**

Secret <4KB → Encrypt API → KMS → checks permission, if they pass → performs encryption → sends back the encrypted secret

Then to decrypt → Decrypt API → KMS will know what key was used for encryption → checks permission if can decrypt → sends back value in plain text

!!! warning "Important"
    Anything above 4KB: Envelope encryption → GenerateDataKey API (consider using DEK caching)
    
    GenerateDataKey without plaintext   
    Generate for decryption to use at some point   
    To use it we must decrypt later on, which is an extra step

**KMS Limit:** Throttling exception → exponential backoff - can request quota increase

---

**<span style="color: #0057FF;">AWS Encryption SDK</span>**

Client-side encryption - supports envelope encryption, KMS, keyrings  
Encrypt file payloads or logs

The **AWS Database Encryption SDK** is a client-side encryption library that provides encryption and signing of your data before it's sent to a database service like Amazon DynamoDB. 

**Option 1: Just encryption at rest**

DynamoDB Encryption at Rest (Server-Side)

**What happens:**
- Your Python app sends data in **plaintext** to DynamoDB   
- AWS encrypts the **entire table** on their storage disks   
- **ALL** attributes are encrypted together as a block 

**Visual Flow:**
```
Python App → [Plaintext JSON] → Network (TLS) → DynamoDB Server → Encrypts ENTIRE item → Stores on disk as encrypted blob
```

**Option 2: Client-Side Encryption (Field-Level)**

**What happens:**
- Your Python app encrypts **specific sensitive fields** before sending
- You choose which attributes to encrypt (CreditCard, SSN)
- Other fields remain searchable (Name, Email, CustomerID)

**Visual Flow:**
```
Python App → SDK encrypts CreditCard & SSN → [Mixed JSON with encrypted fields] → Network (TLS) → DynamoDB stores mixed plaintext/ciphertext
```

**Advantage:** Search something without decrypting the entire database

**Option 3: Asymmetric KMS**
```
Python App → Encrypts with Public Key → [Encrypted data] → Network (TLS) → DynamoDB stores encrypted data → Read data → Decrypt with Private Key → Plaintext
```

Only holders of the **private key** can decrypt the data

---

**<span style="color: #0057FF;">Quick Reference Table</span>**

| Type | Where | When | AWS Services |
|------|-------|------|--------------|
| **At Rest** | Data stored on disk | Always encrypted on storage | S3, EBS, RDS, DynamoDB |
| **In Transit** | Data moving over network | During transmission | TLS/SSL, VPN, HTTPS |
| **Client-Side** | Before leaving application | Before sending to AWS | Encryption SDK, S3 client-side |

**Key Points:**

KMS keys **NEVER leave AWS** unencrypted   
Max 4KB data can be encrypted directly with KMS    
For larger data: Use **envelope encryption**   
KMS integrates with CloudTrail (audit all key usage)    
Multi-region keys available for disaster recovery (MRKs) - same keys deployed across AWS Regions

---

**<span style="color: #0057FF;">Envelope Encryption (Important for Exam)</span>**

**Why?** KMS keys can only encrypt data up to 4KB in size. So if you need to encrypt a 10GB file, 500MB database backup, or even a 100KB image, you can't use the KMS key directly. That's where **GenerateDataKey** comes in and you do it locally.

**Encryption:**

1. You make an API call to KMS:
```bash
aws kms generate-data-key \
  --key-id arn:aws:kms:us-east-1:123456789:key/your-kms-key-id \
  --key-spec AES_256
```

KMS generates a random 256-bit symmetric key (the "data key")

2. API returns one data key in 2 formats:   
   * **Plaintext:** to use now  
   * **Encrypted with KMS (CiphertextBlob):** best practice so you can store it alongside encrypted data or as S3 Object Metadata

3. Encrypt your large file with the Plaintext Key:
```
Large files + PlainText_key → Encrypted File
```

4. Delete the plaintext from memory  
5. Store your Encrypted File with the Encrypted data key (CiphertextBlob)

**Decryption:**

* Retrieve CiphertextBlob + Encrypted_File   
* Call KMS Decrypt(CiphertextBlob)  
* Receive plaintext key  
* Decrypt file: Encrypted_File + Plaintext_Key → File  
* DELETE Plaintext key from memory again!  

!!! info "Important"
    This is a very common pattern with DynamoDB (and many other AWS services).

---

**<span style="color: #0057FF;">S3 Encryption</span>**

**SSE-S3:**

* AWS fully managed   
* Header: `x-amz-server-side-encryption: AES256`

**SSE-KMS:**

* AWS KMS manages Customer Managed Keys  
* When you upload an object, S3 calls **KMS.GenerateDataKey to get data key**  
* S3 encrypts the object using that key  
* S3 stores the encrypted data key with the object  
* To download, S3 calls KMS.Decrypt to unwrap the key  
* Header: `x-amz-server-side-encryption: aws:kms`  
* To specify an AWS managed key: `x-amz-server-side-encryption: aws:kms`

To specify a customer managed key:

  * `x-amz-server-side-encryption: aws:kms`  
  * `x-amz-server-side-encryption-aws-kms-key-id: arn:aws:kms:region:account:key/xxxx`  
* Audit logs in **CloudTrail**  
* Supports **multi-region KMS keys** (for DR)

**SSE-C:**

* You provide the encryption key in the request header  
* AWS encrypts and decrypts but doesn't store the key  
* Must apply key with every GET request  
* Headers:
  * `x-amz-server-side-encryption-customer-algorithm: AES256`  
  * `x-amz-server-side-encryption-customer-key: <Base64-encoded key>`  
  * `x-amz-server-side-encryption-customer-key-MD5: <Base64-encoded MD5 hash>`

**Client-side encryption:**

* You encrypt data yourself before upload  
* Header: `x-amz-meta-encrypted: true`

---

**<span style="color: #0057FF;">DynamoDB Encryption</span>**

**AWS Owned:** SSE default encryption  

**AWS Managed KMS Key:**

* A KMS key automatically created and managed by AWS for DynamoDB  
* Rotates every 3 years

**Customer Managed:** Stored in our account but created and managed by me  

**Client-side encryption:** DynamoDB Encryption Client (legacy) - now there is AWS Database Encryption SDK

**DynamoDB Global Tables Encryption:**

**The problem they solve:** Normally, if you encrypt data in US-East with a key from US-East, you can't decrypt it in Europe without copying that key (which creates security headaches).

Same key exists in multi-region so data can be decrypted  
You encrypt client-side for this! With client-side encryption

**Global Aurora:**

You can do the same with Aurora using AWS Encryption SDK

---

**<span style="color: #0057FF;">Secrets Manager vs Systems Manager Parameter Store</span>**

**Secrets Manager:**

* Automatic rotation for RDS, Redshift, DocumentDB  
* Built-in Lambda for rotation  
* Versioning  
* **Cross-account** access with replica  
* Encrypted with KMS

**Systems Manager Parameter Store** (Config or Flag or simple secrets):

* Hierarchical naming: `/dev/db/password`, `/prod/db/password`  
* Integration with CloudFormation, ECS, Lambda  
* Versioning  
* Three types: `String`, `StringList`, `SecureString`  
⚠️ Manual rotation (no built-in)  
GetParameters or GetParametersByPath API  
Can have TTL as expiration date

**When to use Systems Manager Parameter Store:**

* App settings, feature flags, non-critical values  
* Good for parameters that don't often change  
* Tight AWS native integration

---

**<span style="color: #0057FF;">Common Exam Scenarios</span>**

!!! info "Scenario 1: Application in Auto Scaling group processing SQS data"
    An application hosted in an Auto Scaling group of On-Demand EC2 instances is used to process data polled from an SQS queue, and the generated output is stored in an S3 bucket. To enhance security, you were tasked to ensure that all objects in the S3 bucket are encrypted at rest using server-side encryption with AWS KMS keys.

**Solution for SSE-KMS:**

Headers required:
* `x-amz-server-side-encryption: aws:kms`
* `x-amz-server-side-encryption-aws-kms-key-id: <key-arn>`

!!! info "Important"
    If you have just `x-amz-server-side-encryption`, it doesn't know what encryption to pick!

**For SSE-C (Server-Side Encryption with Customer-Provided Keys)**, you need **three headers** with every request:

1. `x-amz-server-side-encryption-customer-algorithm` — Must be set to `AES256`  
2. `x-amz-server-side-encryption-customer-key` — The base64-encoded 256-bit encryption key  
3. `x-amz-server-side-encryption-customer-key-MD5` — The base64-encoded MD5 digest of the key (used for integrity verification)

**For SSE-S3:**

* `x-amz-server-side-encryption: AES256`

---

**<span style="color: #0057FF;">Common Exam Scenarios </span>**

!!! question "Common Exam Scenarios"
    **Scenario 1: Data must be encrypted before leaving the application** → Client-side encryption (Encryption SDK, S3 client-side)
    
    **Scenario 2: Need audit trail of all encryption key usage** → KMS with CloudTrail (not S3-managed keys)
    
    **Scenario 3: Encrypt large files efficiently** → Envelope encryption pattern
    
    **Scenario 4: Automatically rotate database passwords** → AWS Secrets Manager
    
    **Scenario 5: Lowest cost encryption for S3** → SSE-S3 (S3-managed keys) - free
    
    **Scenario 6: Encrypt specific DynamoDB attributes** → DynamoDB Encryption Client (client-side)
    
    **Scenario 7: Cannot encrypt existing RDS database** → Create snapshot → Copy with encryption → Restore
    
    **Scenario 8: Share encrypted S3 object with another AWS account** → Use KMS key with cross-account policy + bucket policy
    
    **Scenario 9: HIPAA/PCI compliance requirements** → Customer-managed KMS keys (for audit and control)
    
    **Scenario 10: High throughput S3 uploads hitting KMS limits** → Use S3 bucket keys (reduces KMS calls by 99%) - don't encrypt each file in S3 as 1 file = 1 encryption, so do it at bucket level to reduce so many calls