**Lambda → serverless** 

Lambda → specify the amount of memory allocated to → more memory more CPU   
To increase performance connect db outside handle

**Aliases aka pointer**  
Point to a specific lambda version  
$LATEST \= latest   
When we publish lambda \== immutable therefore v1 and can not change code (each version is independent)

* Dev alias → $LATEST mutable  
* Prod alias → v1 immutable  
* Test ALIAS → v2 immutable

Alias enable Canary deployment percentage of prod traffic can go to test  
   
**Dependency & Layers:**  
Just a zip archive that contains libraries, runtime or other dependency  
Used to pull in additional code and content in form of layer 

Lambda dependency you zip and **upload straight to Lambda is less than 50MB**   
Otherwise s3 first  
AWS SDK by default installed  
With cloud formation you cant add dependency you have to do the s3 way only takes simple one if you wanna do inline

**Concurrancy:**  
If the first function is still running and you invoke it again, it creates a concurrent lambda. **This is for all your lambda function in one region \!**  
For initial burst you can have 500-3000 depending on region   
My default account limit 1000 and can be increased   
Each invocation over limit will invoke a throttle

**Throttle behaviour:**  
For sync invocation 429 HTTP status code: 429 and the message is “Request throughput limit exceeded- client retries   
For asyn retries automatically twice then DLQ up to 6 hrs

**Reserved Concurrency:**  
If you know that your lambda 1 needs **max** 200 you can reserve that so that function wont go on 429 error it will have guarantee 200 invocation  
Function A (image processing)  
Function B (order processing)  
Function C (user notifications)

If Function A suddenly gets a huge spike in traffic (say, 1,000 S3 uploads happen at once), it could consume all 1,000 concurrent executions. This means:

* Function B and C would be **throttled** (rejected)  
* Critical business operations might fail  
* You'd see `TooManyRequestsException` errors

**Provisioned Concurrency:**  
You essentially warm it up so no cold start you pay these even in idle\!

Important Exam Trap 

**\*\*Exam Trap → "Function has reserved concurrency of 50 but needs to handle spike of 200 requests" \> \> Answer: \*\*It will throttle at 50\*\* regardless of account capacity**

**If application receives spikes and having 429 what can i do:**  
**Need instant responseRequest limit increase \+ Provisioned Concurrency \- Tens of thousands to millions**  
**Processing can wait few secondsSQS buffer**  
**Unpredictable massive spikesSQS buffer**  
**Cost sensitiveSQS buffer (cheapest)**

**AutoScaling API:**  
Application Auto Scaling \= A service that automatically scales AWS resources based on demand.  
You can only auto-scale Provisioned Concurrency  
Without Auto Scaling:

* Provisioned Concurrency: 100 (fixed)  
* Normal traffic: Uses 50 ✅   
* Spike traffic: Needs 500 ❌ Only 100 warm, rest cold start

   
With Auto Scaling: 

* Minimum Provisioned: 100   
* Maximum Provisioned: 500  
*  Scale when: Utilization \> 70%

A company has a Lambda function that powers a customer-facing API. The function experiences **predictable traffic spikes every day between 9 AM and 5 PM**. Users complain about **slow response times during the start of peak hours** due to cold starts. The company wants to minimize latency while optimizing costs.

 Configure Provisioned Concurrency with Application Auto Scaling

**Env Variable:**  
You can pass dynamic variable   
Can call different variable for different env such as DEV, TEST, PROD  

**Async**:  
Lambda queues the event for processing and returns response  
Usecases :s3 sns event bridge SES  
Can configure Dead Letter Queues (DLQ) to SQS or SNS for failed events  
**\--invocation-type Event**

**Sync:**  
Wait for the function to process the event first \- error handling on client side the retries  
API Gateway, ALB, Cloudfront, Elastic Load Balancing, Amazon CloudFront , S3 Batch

**Event SourceMapping:**

S3 bucket → "Hey Lambda, new file uploaded\!" → Lambda runs **No event source** Mapping  
Don't need:

* S3  
* SNS  
* API Gateway  
* Cognito

SQS queue ← Lambda keeps asking "Any messages?" ← **Event Source Mapping** manages this

**From a stream or queue**  
Source Mapping is a Lambda resource that **reads from a stream or queue** and invokes your Lambda function with the records it retrieves.

* Kinesis Data Stream  
* SQS   
* Dynamo DB stream  
* Amazon MQ  
* Apache Kafka (MSK)


Record polled from source so **lambda is asking for record** and then getting them in batches   
Lambda is invoked Synchronously →  it will poll from kinesis → when it has data → invoke lambda with **event batch** (you can also have batches processed in parallel)

Error Handling: **default function will rerun the whole batch** (you can discard old event/ restrict retries/split error batch)

**SQS** →   
Same logic under the hood but its long pooling   
Queue batch size can be 1-10 messages   
Recommended Timeout **6x timeout of lambda**  
**Use DLQ on SQS not on Lambda** **(DLQ for lambda is only for async stuff**)

SQS \[msg1,msg2,msg3\] ← Event Source Mapping (Polls every few second) Batches → Lambda function

**Lambda Layers**  
With layers, you can use libraries in your function without needing to include them in your deployment package  
A function can use up to 5 layers at a time.

## **Lambda@Edge**

## Lambda functions that run at CloudFront edge locations (closer to users)

Instead of running in one region, your code runs at 200+ locations worldwide.  
Regular Lambda: User (Australia) → Lambda (us-east-1)   
Long distance \= High latency 😕  
   
Lambda@Edge: User (Australia) → Edge Location (Sydney) → Lambda runs HERE   
Close \= Low latency **😊**  
Lambda@Edge Limitation:

* No Layers  
* No VPC  
* No env Variable

**Lambda VPC**

* Private subnet ID.  
* Security Group ID (with required access).

If you need access to the internet, you will **need to create a NAT in your VPC**  
AWSLambdaVPCAccessExecutionRole

**AWS SAM**  
A shortcut for writing CloudFormation templates for serverless apps.  
Instead of writing 100+ lines of CloudFormation, you write 20 lines of SAM.

Transform: AWS::Serverless-2016-10-31

**Limitation:**   
Request more memory 128Mb \- 10240 Mb (1MB increment)  
Max 15 min (900s), default 3s  
Env Variable only 4KB   
Disk capacity /tmp folder: 512-10GB (ephemeral)-   
**Ephemeral means:** Temporary \- it only lasts for the lifetime of the execution environment- 10 lambda each has its separate /tmp but survive with env  
Concurrency 1000 (can be increase)

Deployment:  
Lambda deployment size .zip file \= 50MB  
Uncompressed (code  \+ dependency) \= 250 Mb also on S3  
Can use /tmp to load file at start up  
Env Variable only 4KB 

To increase performance connect db outside handle

**Monitor**  
Lambda needs permission to grant cloudwatch logs:  
AWS LambdaBasicExecutionRole \- write logs to cloud watch  
AWSXRayDeamonWriteAccess \- upload trace data to xray

Without SDK (Basic Tracing) Basic segment info  
With SDK (Detailed Tracing) With SDK (Detailed Tracing)

504:  
50 ms to 29 seconds api gateway

Best Practices:

**What is a Custom Runtime?**

A **Custom Runtime** allows you to run languages that AWS doesn't natively support (like C++, Rust, PHP, COBOL, etc.) by implementing the **Lambda Runtime API**.

The Custom Runtime:

1. Receives invocation events from Lambda  
2. Passes them to your code  
3. Returns responses back to Lambda

 – Separate the Lambda handler (entry point) from your core logic.

 – Take advantage of Execution Context reuse to improve the performance of your function

 – Use AWS Lambda Environment Variables to pass operational parameters to your function.

 – Control the dependencies in your function’s deployment package.

 – Minimize your deployment package size to its runtime necessities.

 – Reduce the time it takes Lambda to unpack deployment packages

 – Minimize the complexity of your dependencies

 – Avoid using recursive code

Lambda deployments only support:

### **4\. Traffic-Shifting (Blue/Green for Lambda)**

* **Canary** (e.g., 10% → wait → 90%)

* **Linear** (e.g., 10% every X minutes)

* **All-at-once**

**API Gateway \+ Lambda:** Build REST/HTTP APIs  
**S3 \+ Lambda:** Process files on upload  
**DynamoDB Streams \+ Lambda:** React to table changes  
**EventBridge \+ Lambda:** Scheduled/event-driven automation  
**SQS \+ Lambda:** Asynchronous processing with retries  
**Step Functions \+ Lambda:** Orchestrate complex workflows

**"Authenticate users at the edge before accessing S3 content" → Lambda@Edge (Viewer Request)**

**"Add security headers to all CloudFront responses" → Lambda@Edge (Origin Response) or CloudFront Functions**

**"A/B testing for users globally with low latency" → Lambda@Edge (Viewer Request)**

**"Simple URL redirect with lowest latency" → CloudFront Functions (simpler, faster)**

**"Access DynamoDB at the edge" → Lambda@Edge (can make network calls)**

**The type of concurrency has no bearing on the Lambda function's ability to process the compute-heavy workflows. So both these options are incorrect.**

**Past questions:**

You developed a shell script which uses AWS CLI to create a new Lambda function. However, you received an **InvalidParameterValueException** after running the script.

You provided an IAM role in the CreateFunction API which AWS Lambda is unable to assume.

Concurrent executions \= requests per second × duration per request

You are developing a Lambda function which processes event notifications from Amazon S3. It is expected that the function will have:

*  50 requests per second  
* 100 seconds to complete each request

What should you do to prevent any issues when the function has been deployed and becomes operational?

Request for AWS to increase the limit of your concurrent executions

For Lambda functions that process Kinesis or DynamoDB streams, the number of shards is the unit of concurrency. If your stream has 100 active shards, there will be at most 100 Lambda function invocations running concurrently. This is because Lambda processes each shard’s events in sequence.

