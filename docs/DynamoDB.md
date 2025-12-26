!!! example "Topics according to exam"
    For AWS Developer exam: the main thing to know is:

    - Calculate RCU WCU
    - Dax
    - LSI and GSI 
    - Query and Scan

    For AWS Solution Architect exam: the main thing to know is:
  
    - DAX
    - There is way more question on RDS

**<span style="color: #0057FF;">Keys: </span>**

Primary Key must be there:

* Simple Primary Key (Partition Key only): Also called hash key must be unique 
* Composite Primary Key (Partition Key only + sort key) → combined must be unique
Here the combination must be unique 
Item by same partition key are sorted together sorted by *sort key* 

Example: UserID (partition) + OrderDate (sort) in an Orders table 
The partition key determines which partition stores the data. 

*Items with the same partition key are stored together*

**<span style="color: #0057FF;">LSI: </span>**
  
At table creation can not be added later  
Uses **same partition key** but different sort key   
5 limit LSI  
**Strongly consistent** 
!!! note "Example"
    - exam loves to ask if you can use LSI after table is created
    **Table:** Partition Key \= `UserID`, Sort Key \= `OrderDate`  
    **LSI:** Partition Key \= `UserID`, Sort Key \= `OrderStatus`

    Now you can query all orders for a user sorted by status, not just by date.

**<span style="color: #0057FF;">GSI: </span>**
Can be created anytime up to 20 GSI  
Can have a different partition key AND different sort key  
**Eventually consistent** reads only (no strongly consistent)
!!! note "Example"
    **Table:** Partition Key \= `UserID`, Sort Key \= `OrderDate`  
    **GSI:** Partition Key \= `OrderStatus`, Sort Key \= `OrderDate`  
    **Exam Tip:** GSI = "G" for "Global flexibility" - different keys, added anytime, separate throughput

!!! question "Questions to keep in mind"
    "Need to query by a different attribute after table is created" → Use GSI  
    "Need to query same partition but different sort order" → Use LSI (if at creation time) 
    "Running out of capacity on queries" → GSI has separate throughput; LSI shares with table 
    "Need strongly consistent reads on secondary index" → Only LSI supports this  
    "Sparse index" (only some items have the attribute) → Both LSI and GSI support this (items without the index key won't appear in the index)  
    "Write throttling on GSI" → Can throttle writes to the main table (GSI writes must complete)

**<span style="color: #0057FF;">Capacity: </span>**

Provisioned capacity:

**Strongly consistent**

  *1 RCU gives you 1 eventually consistent reads of 4KB each per second*

`If you need to read 10/s each item 8KB strongly consistent`   
`For strongly consistent 1=4KB so 2 for 8KB`   
`I need to read 10 Item therefore 2x10item = 20RCU`

* Eventually consistent:

  *1 RCU gives you 2 eventually consistent reads of 4KB each per second*

`Do the same calculation but half it`        
`I need to read 10 Item therefore 2x10item = 20RCU /2 =10RCU`  
`Eventually consistent reads same calculation but half it` 

!!! question "Examples"
    - Read 50 items per second, each item is 2KB, strongly consistent.      
    1RCU is enough so 50 x 1RCU = =50RCUs 

    - Read 50 items per second, each item is 2KB, eventually consistent.  
    1RCU is enough so 50 x 1RCU = =50RCUs/2 = 25RCUs 

    - Read 20 items per second, each is 6KB, strongly consistent.  
    2RCUs = 20x2= 40RCus`  

**On-Demand:**

* Pay per request 
* Good for unpredictable workloads

**DynamoDB Stream:**  
* 24hrs Retention

**<span style="color: #0057FF;">Operation: </span>**

**Write Operations**

* `PutItem` - Create or replace entire item- doesn't exist create a new one or overrides  
* `UpdateItem` - Update specific attributes if it exist otherwise create item  
* `DeleteItem` - Remove item  
* `BatchWriteItem` - Up to 25 items (max 16MB total)

**Read Operations**

* `GetItem` - Read single item by primary key  
* `Query` - Read items with same partition key (can filter by sort key)  
* `Scan` - Read entire table (expensive, avoid in production!)  
* `BatchGetItem` - Up to 100 items (max 16MB)

**<span style="color: #0057FF;">Query vs Scan: </span>**
 
**Query:**

* Specify Partition key 
* Return sorted result  
* Efficient- only reads matching items` 
* Should prefer this

**Scan:**

* Reads every Item in table  
* Expensive  
* Parallel scan  
* Avoid in prod

**<span style="color: #0057FF;">DAX: </span>**

* Fully managed caching service  
* Fast   
* Good for read heavy workloads

**<span style="color: #0057FF;">Global Table: </span>**

* Multi region  
* Less then 1s replication  
* Active-active set up

**Point in Time recovery TTL:**

* Restore to any point in last 35 days

**Backup:**

* Manual backups  
* Can restore the same to different region

**<span style="color: #0057FF;">Conditional writes: </span>**

* Do a write operation if xyz  
* This prevents race conditions where multiple users/processes might try to update the same item simultaneously.

The operation throws a `ConditionalCheckFailedException`and:

* No changes are made to the item  
* You still consume Write Capacity Units (WCUs)! 
* Your application should catch this exception and handle it (retry, notify user, etc.)

**<span style="color: #0057FF;">Security: </span>**

**Encryption**

* Encryption at rest (enabled by default)  
* Encryption in transit (HTTPS)  
* AWS owned, AWS managed, or Customer managed KMS keys

**`TransactWriteItems`** and **`BatchWriteItem:`**

* Writes directly to DynamoDB tables  
* Works on regular tables, global tables  
* Can write to multiple tables in one transaction  
* Does NOT write to DynamoDB Streams  
* Has nothing to do with consuming streams

**`TransactWriteItems:`**

* All-or-nothing  
* ACID  
* Conditional Logic  
* 2WCU  
* Put, Update, Delete, ConditionCheck

**`​​BatchWriteItem`**

* Some can succeed, others can fail  
* Not acid  
* Put, Delete

**<span style="color: #0057FF;">DynamoDB stream data: </span>**

Stream the data in an Amazon DynamoDB table. Enable DynamoDB Streams, and configure an AWS Lambda function with `AmazonDynamoDBFullAccess` permissions to perform anonymization on newly written items

DynamoDB Streams processes changes to already written data, meaning unanonymized PII would be stored in DynamoDB before anonymization

**<span style="color: #0057FF;">Hot Partition:</span>**

Basically when one partition key is being updated too often so bad key partition key choosing

* Implement error retries and exponential backoff.

* Refactor your application to distribute your read and write operations as evenly as possible across your table.

**<span style="color: #0057FF;">Optimistic locking:</span>**

**What is Optimistic Locking?**

Optimistic locking *assumes conflicts are rare* and only checks for conflicts at write time.- *Conditional updates* \- "prevent update if attribute has certain value"

**Why NOT Pessimistic Locking?**

*DynamoDB doesn't support native locks*

**<span style="color: #0057FF;">Limit:</span>**

Item in a Amazon DynamoDB table is 400 KB

!!! question "Common Exam Scenarios"
    - Need shared data across regions? → Global Tables
    - Need caching for reads? → DAX
    - Need to trigger actions on changes? → DynamoDB Streams + Lambda  
    - Need to query by different attributes? → GSI  
    - Scan taking too long? → Use Query with proper keys or GSI  
    - Running out of capacity? → Switch to On-Demand or increase provisioned capacity  
    - Need ACID transactions? → Use TransactWriteItems/TransactGetItems  
    - You need to add an index to query by a new attribute on an existing table. What should you use? →  GSI - because LSI can only be created at table creation time! 
    - Different partition key? → GSI 
    - Same partition key, different sort order? → LSI 
 
!!! danger "Important info on Backups"

    - Use the DynamoDB on-demand backup capability to write to Amazon S3 and download locally
    *Use the DynamoDB on-demand backup capability to write to Amazon S3 and download locally*
    - This option is not feasible for the given use-case. DynamoDB has two built-in backup methods (On-demand, Point-in-time recovery) that write to Amazon S3, but you will not have access to the S3 buckets that are used for these backups.  
    **Use Hive with Amazon EMR to export your data to an S3 bucket and download locally**
