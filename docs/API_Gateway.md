!!! example "Topics according to exam"
    For AWS Developer exam: the main thing to know is:

    - Stages
    - Throttling 
    - Integration type
    - Especially API with Lambda 

    For AWS Solution Architect exam: the main thing to know is:
    
    - Usage Plan

**<span style="color: #0057FF;">API types:</span>**

* Rest API (full feature its old) → Needs Lambda authorizer or cognito  
* HTTP API (simpler cheaper faster) → support JWT authorize  
* Websocket API (real time 2 way communication) → Lambda authorize

API Gateway

* Stage (dev, test, prod)

* Resource (/users) URL path

* Method (GET, POST)

* Integration (Lambda, HTTP, etc.)

*Deployment:* Snapshot of your API pushed to a stage

**Cache Keys:**

* **Remember 29-second timeout** \- cannot be increased

**<span style="color: #0057FF;">API Gateway \+ Lambda:</span>**

* Lambda can run up to 15 minutes  
* But API Gateway times out at 29 seconds  
* Solution: Use asynchronous pattern (invoke Lambda async, return immediately)

**Exam Tip:** API Gateway max timeout is 29 seconds. For long-running tasks, use async Lambda or Step Functions.

**<span style="color: #0057FF;">API Gateway Errors:</span>**

*Client Errors (4XX):*

* 400 Bad Request: Invalid request (validation failed invalid JSON)  
* 403 Forbidden: Access denied (auth failure, WAF blocked, resource policy)  
* 429 Too Many Requests: Throttled (exceeded rate limit)

*Server Errors (5XX):*

* 502 Bad Gateway: Bad response from backend (Lambda error, timeout, malformed response)  
* 503 Service Unavailable: API Gateway overloaded, tmp issue  
* 504 Gateway Timeout: Integration timeout (default 29 seconds max)

**<span style="color: #0057FF;">Logging & Monitoring:</span>**

*CloudWatch Logs:*

* Execution logs: Detailed request/response info  
* Access logs: Who accessed API, when, response codes  
* Enable at stage level

*CloudWatch Metrics:*

* CacheHitCount / CacheMissCount: Cache effectiveness  
* Count: Total API requests  
* IntegrationLatency: Time between API Gateway → backend  
* Latency: Time from client request → response (includes IntegrationLatency)

*X-Ray:*

* Enable tracing for end-to-end request tracking  
* See time spent in API Gateway, Lambda, DynamoDB, etc.

Exam Tip: Know the difference between Latency and IntegrationLatency.

**<span style="color: #0057FF;">Stage Variables:</span>**

You typically have multiple environments — dev, staging, production — and want the same API configuration to behave differently in each.You deploy your API once and promote it through stages without changing any configuration.

you can promote the test stage to the prod stage. The promotion can be done by redeploying the API to the prod stage or updating a stage variable value from the stage name of test to that of prod.

**Stage variable like env variable for API Gateway**

Use cases for stage variables:

* Configure HTTP endpoints your stages talk to (dev, test, prod etc.).  
* Pass configuration parameters to AWS Lambda through mapping templates.

Stage variables are passed to the “context” object in Lambda.

Stage variables are used with Lambda aliases.

**<span style="color: #0057FF;">Integration type Mapping Template:</span>**


Mapping templates let you **transform** request and response payloads as they pass through API Gateway

**Integration types**: define how your api gateway connect to the backend 

You choose an API integration type according to the types of integration endpoint you work with : 2 option
Used when your backend is an HTTP endpoint (like an existing web server, EC2 instance, on-premises API)  
Passes the entire request to Lambda in a structured format

* **AWS\_PROXY:**API Gateway passes the entire request to Lambda as-is and returns the Lambda response directly to the client. → { "statusCode": 200, "headers": {}, "body": "string" } → otherwise 502  
* **AWS:** Lambda custom integration: API Gateway transforms requests and responses using mapping templates before and after calling Lambda.  
* **HTTP\_PROXY:** Passes requests directly to an HTTP backend and returns the response unchanged  
* **HTTP:** Connects to an HTTP backend but allows transformation via mapping templates  
* **MOCK:** Returns a response directly from API Gateway without calling any backend.

When to choose what?

**If there is Lambda:**

* Passthrough → AWS\_PROXY  
* Transform → AWS

**If its a HTTP backend:**

* Passthrough → HTTP\_PROXY  
* Transform → HTTP

**If it doesnt want to interact with backend:**

* MOCK

**Security:**

**<span style="color: #0057FF;">User authentication:</span>**

**1- IAM Permissions**

Great for users/roles within AWS \+ resource policy for cross account

* Handles **authentication** and **authorization**  
  **When to use:** Internal AWS services, EC2 calling API  

* Can be used with: HTTP API  \- REST API \- WEBSOCKET API  

!!! info "Flow"
    Client does an api call with Sig 
    v4→ API Gateway will decrypt those and check with IAM Policy→ (IAM Policy checks )→ Backend 


**2- Lambda Authorizer (Custom Authorizer)**

  *What is it?*

  * You write the authentication logic in a Lambda function. 
  * API Gateway calls your Lambda to decide if the request is allowed.  
  * Its a token based authZ

  **Mainly used for third party auth system **

  *Use Cases:*

* OAuth 2.0 / OAuth 1.0 tokens  
* SAML assertions  
* 3rd party authentication (Auth0, Okta)  
* Custom authentication logic (check database, special rules)  
* API keys from external system 
* Can be used with: HTTP API  \- REST API \- WEBSOCKET API

!!! info "Flow"
    Client → authenticate with 3rd party system gives a token→ we pass token to API Gateway either through header or req parameter → API Gateway talks to Lambda AuthZ retrieves info on Context \+ token → verify with the 3rd party that validity of token → will return IAM Principal \+ IAM Policy → just once and it will be cached

**3-Cognition user Pools**:

* Database of users high level
* Cognito fully managed lifecycle
* AuthN= Cognition Users Pool
* AuthZ= API Gateway Methods
* You manage your own userpool   
* No need to write custom code  
* AuthZ in the backend  
* *JWT Authorizer (HTTP API only)*
* Validates JWT tokens from any OpenID Connect (OIDC) or OAuth 2.0 provider Okta

!!! danger "Important"
    HTTP API(only here) 

!!! info "Flow"
    Client → authenticate through Cognito user Pools → retriever token → pass the token to API Gateway with the call → API gateway will evaluate the token **JWT(ISSUED BY COGNITO)** with internal cognition connection → if token correct you get backend access 

**<span style="color: #0057FF;">Chacing:</span>**

* Caching allows you to cache the endpoint’s response.  
* Caching can reduce the number of calls to the backend and improve the latency of requests to the API.  
* The default TTL is 300 seconds (min 0, max 3600).  
* Caches are defined per stage.  
* Cache capacity: 0.5GB to 237GB  
* Can encrypt cached data  
* You can flush the entire cache (invalidate it) immediately if required- header: Cache-Control: max-age=0 

**API Gateway Cache Keys**

When caching is enabled, API Gateway needs to decide: "Have I seen this exact request before?"

By default, it only looks at the **URL path**. But often that's not enough.

Two request same endpoint with default cashing (path only):

* GET /products?category=electronics   
* GET /products?category=clothing

You get electronics data which is wrong 

Hence why Cache keys:

* Query string parameters  → Cache key: /products|electronics|price  
* Headers → for language Accept \-Language   
* Path parameters → /users/456/orders/abc

**<span style="color: #0057FF;">API Throttling:</span>**

* *10,000 requests per second (RPS)* \- steady state  
* *5,000 burst* \- token bucket algorithm  
* Exceeded → *429 Too Many Requests →exponential backoff and retry* 
* 5000 not a hard limit AWS can increase it based on your use case

!!! question "Example Question"
    - **"API returns 429 errors during traffic spikes..."** → Increase burst limit or steady-state rate

    - **"Need to limit free tier users to 100 requests per day..."** → Create usage plan with quota, assign API keys

    - **"One client is overwhelming the API affecting others..."** → Implement usage plans with per-client throttling

    - **"Backend database can only handle 50 requests per second..."** → Set method-level throttling to protect backend

    - **"API returns 429 errors during traffic spikes..."** → Increase burst limit or steady-state rate

    - **"Need to limit free tier users to 100 requests per day..."** → Create usage plan with quota, assign API keys

    - **"One client is overwhelming the API affecting others..."** → Implement usage plans with per-client throttling

    - **"Backend database can only handle 50 requests per second..."** → Set method-level throttling to protect backend

**<span style="color: #0057FF;">Usage Plan:</span>**

A usage plan specifies who can access one or more deployed API stages and methods — and how much and how fast they can access them.

The plan uses API keys to identify API clients and meters access to the associated API stages for each key.

It also lets you configure throttling limits and quota limits that are enforced on individual client API keys.

You can use API keys together with [usage plans](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-api-usage-plans.html) or [Lambda authorizers](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html) to control access to your APIs.

**Cors:**

Cross-Origin Resource Sharing. It's a browser securit**y feature that blocks web pages from calling APIs on different domains**.

OPTIONS method handles preflight

With proxy integration, Lambda must return CORS headers too

* `Access-Control-Allow-Origin`Which domains can call (use `*` for any)  
* `Access-Control-Allow-Methods`Which HTTP methods allowed  
* `Access-Control-Allow-Headers`Which headers client can send  
* `Access-Control-Max-Age`How long to cache preflight response

!!! question "Common Exam Scenario"
    - **"Serverless REST API for mobile app"** → API Gateway \+ Lambda (proxy integration) \+ DynamoDB
    - **"Reduce Lambda invocations for repeated request"** → → Enable caching
    - **"API only accessible from specific VPC"** → Private endpoint \+ resource policy
    - **"Real-time chat application"** → WebSocket API
    - **"Gradually roll out new API version"** → Canary deployment
    - **"OAuth token authentication"** → Lambda authorizer

**Assign a Security Group to your API Gateway** \- API Gateway does not use security groups but uses resource policies, which are JSON policy documents that you attach to an API to control whether a specified principal (typically an IAM user or role) can invoke the API. 
You can restrict IP address using this, the downside being, an IP address can be changed by the accessing user. So, this is not an optimal solution for the current use case.

