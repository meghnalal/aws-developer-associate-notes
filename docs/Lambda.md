!!! example "Topics according to exam"
    For AWS Developer exam: the main thing to know is:

    - Aliases
    - Limitation CPU 
    - API Gateway + Lambda
    - Concurrancy
    - Exam loves API Gateway + Lambda + DynamoDB

    For AWS Solution Architect exam: the main thing to know is:
    
    - VPC Lambda
    - Exam loves API Gateway + Lambda + RDS

**<span style="color: #0057FF;">Lambda → serverless</span>**

Lambda → specify the amount of memory allocated to → more memory more CPU   

!!! warning "Important"
    To increase performance connect db outside handle

**<span style="color: #0057FF;">Aliases aka pointer</span>**
  
Point to a specific lambda version  
$LATEST \= latest   
When we publish lambda = immutable therefore v1 and can not change code (each version is independent)

* Dev alias → $LATEST mutable  
* Prod alias → v1 immutable  
* Test ALIAS → v2 immutable

!!! warning "Important"
    Alias enable Canary deployment percentage of prod traffic can go to test

**<span style="color: #0057FF;">Dependency & Layers:</span>**
 
Just a zip archive that contains libraries, runtime or other dependency  
Used to pull in additional code and content in form of layer 

Lambda dependency you zip and **upload straight to Lambda is less than 50MB**   
Otherwise upload s3 first  
AWS SDK by default installed  
With *cloud formation* you cant add dependency you have to do the s3 way only takes simple one if you wanna do inline

**<span style="color: #0057FF;">Concurrancy:</span>**
   
If the first function is still running and you invoke it again, it creates a concurrent lambda. *This is for all your lambda function in one region \!*  
* For initial burst you can have 500-3000 depending on region   
* By default account limit 1000 and can be increased   
* Each invocation over limit will invoke a throttle

**<span style="color: #0057FF;">Throttle behaviour:</span>**
 
* For sync invocation **429 HTTP** status code: 429 and the message is **“Request throughput limit exceeded- client retries"**   
* For asyn retries automatically twice then DLQ up to 6 hrs

**<span style="color: #0057FF;">Reserved Concurrency:</span>**

If you know that your lambda 1 needs **max** 200 you can reserve that so that function wont go on 429 error it will have guarantee 200 invocation

* Function A (image processing)

* Function B (order processing)

* Function C (user notifications)

If Function A suddenly gets a huge spike in traffic (say, 1,000 S3 uploads happen at once), it could consume all 1,000 concurrent executions. This means:

* Function B and C would be **throttled** (rejected)  
* Critical business operations might fail  
* You'd see `TooManyRequestsException` errors

**<span style="color: #0057FF;">Provisioned Concurrency:</span>**

You essentially warm it up so **no cold start** you pay these even in idle\!

!!! warning "Important Exam Trap"
    Exam Trap → "Function has reserved concurrency of 50 but needs to handle spike of 200 requests"
    
    - Answer:It will throttle at 50 regardless of account capacity

    If application receives spikes and having 429 what can I do?

    - Need instant responseRequest limit increase + Provisioned Concurrency which can be increased to  Tens of thousands to millions
    - If processing can wait few seconds then SQSs buffer (cost sensitive the cheapest)

**<span style="color: #0057FF;">AutoScaling API:</span>** 

Application Auto Scaling = A service that automatically scales AWS resources based on demand.  
You can only auto-scale **Provisioned Concurrency**
Without Auto Scaling:

* Provisioned Concurrency: 100 (fixed)

* Normal traffic: Uses 50 ✅ 

* Spike traffic: Needs 500 ❌ Only 100 warm, rest cold start

With Auto Scaling: 

* Minimum Provisioned: 100   
* Maximum Provisioned: 500  
*  Scale when: Utilization \> 70%

!!! question "Example question"

    A company has a Lambda function that powers a customer-facing API. The function experiences *predictable traffic spikes every day between 9 AM and 5 PM*. Users complain about *slow response times during the start of peak hours* due to cold starts. The company wants to minimize latency while optimizing costs.
    
    Configure Provisioned Concurrency with Application Auto Scaling

**<span style="color: #0057FF;">Env Variable:</span>** 
 
You can pass dynamic variable   
Can call different variable for different env such as DEV, TEST, PROD  

**<span style="color: #0057FF;">Async:</span>** 

Lambda queues the event for processing and returns response  
Usecases :s3 sns event bridge SES  
Can configure Dead Letter Queues (DLQ) to SQS or SNS for failed events
<span style="color: #FF1493; font-size: 16px;">--invocation-type Event</span>  

**<span style="color: #0057FF;">Sync:</span>** 
 
Wait for the function to process the event first:

* error handling on client side the retries  

* used with: API Gateway, ALB, Cloudfront, Elastic Load Balancing, Amazon CloudFront , S3 Batch

**<span style="color: #0057FF;">Event SourceMapping:</span>** 

S3 bucket → "Hey Lambda, new file uploaded\!" → Lambda runs **No event source Mapping**
Don't need with :

* S3  
* SNS  
* API Gateway  
* Cognito

SQS queue ← Lambda keeps asking "Any messages?" ← **Event Source Mapping** manages this!

**From a stream or queue**  
Source Mapping is a Lambda resource that **reads from a stream or queue** and invokes your Lambda function with the records it retrieves.

* Kinesis Data Stream  
* SQS
* Dynamo DB stream  
* Amazon MQ  
* Apache Kafka (MSK)

Record polled from source so *lambda is asking for record* and then getting them in batches.

**Kinesis stream**          
Lambda is invoked Synchronously →  it will poll from kinesis → when it has data → invoke lambda with *event batch* (you can also have batches processed in parallel)

Error Handling: **default function will rerun the whole batch** (you can discard old event/ restrict retries/split error batch)

**SQS**

* Same logic under the hood but its long pooling

* Queue batch size can be 1-10 messages

* Recommended Timeout *6x timeout of lambda*

*Use DLQ on SQS not on Lambda* **(DLQ for lambda is only for async stuff**)

SQS \[msg1,msg2,msg3\] ← Event Source Mapping (Polls every few second) Batches → Lambda function

**<span style="color: #0057FF;">Lambda Layers:</span>** 

With layers, you can use libraries in your function without needing to include them in your deployment package  
A function can use up to 5 layers at a time.

**<span style="color: #0057FF;">Lambda@Edge:</span>** 

Lambda functions that run at CloudFront edge locations (closer to users)

Instead of running in one region, your code runs at 200+ locations worldwide.  

Regular Lambda: User (Australia) → Lambda (us-east-1) → Long distance = High latency 😕    
Lambda@Edge: User (Australia) → Edge Location (Sydney) → Lambda runs HERE → Close = Low latency **😊** 

Lambda@Edge Limitation:

* No Layers  
* No VPC  
* No env Variable

**<span style="color: #0057FF;">Lambda VPC:</span>** 

* Private subnet ID.  
* Security Group ID (with required access).

If you need access to the internet, you will **need to create a NAT in your VPC**  
AWSLambdaVPCAccessExecutionRole

**<span style="color: #0057FF;">AWS SAM:</span>** 

A shortcut for writing CloudFormation templates for serverless apps.  
Instead of writing 100+ lines of CloudFormation, you write 20 lines of SAM.

Transform: AWS::Serverless-2016-10-31

AWS Developer loves SAM 

**Limitation:**

* Request more memory 128Mb - 10240 Mb (1MB increment)  
* Max 15 min (900s), default 3s  
* Env Variable only 4KB   
* Disk capacity /tmp folder: 512-10GB (ephemeral)-   
* *Ephemeral means:* Temporary - it only lasts for the lifetime of the execution environment- 10 lambda each has its separate /tmp but survive with env  
* Concurrency 1000 (can be increase)

Deployment:

* Lambda deployment size .zip file \= 50MB  
* Uncompressed (code  \+ dependency) \= 250 Mb also on S3  
* Can use /tmp to load file at start up  
* Env Variable only 4KB 

!!! warning "Important Exam Question"
    To increase performance connect db outside handler

**<span style="color: #0057FF;">Monitor:</span>**
 
Lambda needs permission to grant cloudwatch logs:

* AWS LambdaBasicExecutionRole \- write logs to cloud watch  
* AWSXRayDeamonWriteAccess \- upload trace data to xray
* Without SDK (Basic Tracing) Basic segment info  
* With SDK (Detailed Tracing) With SDK (Detailed Tracing)

**504 Error**:

* 50 ms to 29 seconds api gateway

Best Practices:

**<span style="color: #0057FF;">What is a Custom Runtime?</span>**

A *Custom Runtime* allows you to run languages that AWS doesn't natively support (like C++, Rust, PHP, COBOL, etc.) by implementing the *Lambda Runtime API*.

The Custom Runtime:

1. Receives invocation events from Lambda  
2. Passes them to your code  
3. Returns responses back to Lambda

* Separate the Lambda handler (entry point) from your core logic.

* Take advantage of Execution Context reuse to improve the performance of your function

* Use AWS Lambda Environment Variables to pass operational parameters to your function.

**<span style="color: #0057FF;">Lambda deployments only support:</span>**

* **Canary** (e.g., 10% → wait → 90%)

* **Linear** (e.g., 10% every X minutes)

* **All-at-once**

!!! note "Notes"
    **API Gateway + Lambda:** Build REST/HTTP APIs  
    **S3 \+ Lambda:** Process files on upload  
    **DynamoDB Streams \+ Lambda:** React to table changes  
    **EventBridge \+ Lambda:** Scheduled/event-driven automation  
    **SQS \+ Lambda:** Asynchronous processing with retries  
    **Step Functions \+ Lambda:** Orchestrate complex workflows

!!! question "Questions to keep in mind"
    - "Authenticate users at the edge before accessing S3 content" → Lambda@Edge (Viewer Request)
    - "Add security headers to all CloudFront responses" → Lambda@Edge (Origin Response) or CloudFront Functions
    - "A/B testing for users globally with low latency" → Lambda@Edge (Viewer Request)
    - "Simple URL redirect with lowest latency" → CloudFront Functions (simpler, faster)
    - "Access DynamoDB at the edge" → Lambda@Edge (can make network calls)
    - The type of concurrency has no bearing on the Lambda function's ability to process the compute-heavy workflows. So both these options are incorrect.


