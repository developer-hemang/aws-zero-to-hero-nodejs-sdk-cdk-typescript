# What is Serverless?
Serverless is a way of building applications where you don’t manage servers.
👉 You only write code, and the cloud (AWS) runs it for you.

Simple Definition

>> Serverless means you focus on writing business logic, while the cloud provider automatically handles servers, scaling, and infrastructure.

### Serverless does NOT mean no servers.

Serverless does NOT mean no servers.

👉 It means:
- You don’t manage servers
- AWS manages infrastructure
- You only write code

<details>
<summary> # 🔥 1. What is AWS Lambda (Simple but Strong Definition)</summary>

AWS Lambda is a serverless compute service that lets you run code without managing servers.
You just upload your code → AWS handles execution, scaling, patching, availability.

👉 You pay only for execution time (ms level billing).

</details>
# Why Lamda 

- Vertual functions - no servers to manage!
- limited by time - short executions
- Run On Demand
- Scalling is automated



# 🧠 Real-Life Example

Think of Zomato order processing:

User places order → event generated
Lambda triggers → validates order → pushes to restaurant
No need to run a server 24x7 just for occasional orders

👉 Lambda runs only when needed



# Resource Based Policy in Lamda Function

A resource-based policy in Lambda is a policy attached directly to the Lambda function that defines who (which service or account) is allowed to invoke the function.


## Why Do We Need It?

By default, no external service or account can invoke your Lambda.

### 👉 So if you want:
- S3 to trigger Lambda
- API Gateway to call Lambda
- Another AWS account to access your Lambda
You must explicitly allow it using a resource-based policy.


### How It Works
- The policy is attached to the Lambda function (resource)
- It defines:
    - Principal → Who can access (e.g., s3.amazonaws.com)
    - Action → What they can do (e.g., lambda:InvokeFunction)
    - Resource → Which Lambda function
    - Condition (optional) → Extra restriction


### Example (S3 Invoking Lambda)


```json

{
  "Effect": "Allow",
  "Principal": {
    "Service": "s3.amazonaws.com"
  },
  "Action": "lambda:InvokeFunction",
  "Resource": "arn:aws:lambda:region:account-id:function:my-function",
  "Condition": {
    "ArnLike": {
      "AWS:SourceArn": "arn:aws:s3:::my-bucket"
    }
  }
}

```

👉 Meaning:

Only this S3 bucket can invoke this Lambda


Real-Life Analogy

Think of Lambda as your house:

IAM Role → What you (Lambda) can do (outgoing permissions)
Resource Policy → Who is allowed to enter your house (incoming access)


### Common Use Cases
- Allow S3 → Lambda
- Allow API Gateway → Lambda
- Allow EventBridge → Lambda
- Cross-account access (very important in interviews)


> “Resource-based policies control who can invoke the Lambda function, while IAM execution roles control what the Lambda function can access.”



# ⚡ 2. What is Invocation?

Lambda invocation means triggering or calling an AWS Lambda function to execute its code.

There are 3 types:

# 1. Synchronous Invocation (Request-Response)

In synchronous invocation, the caller sends a request to Lambda and waits for the response.

> Request goes → Lambda runs → Response comes back → then process continues


# How to Invoke (Sync)

You can perform synchronous invocation using:

- AWS CLI
- AWS SDK (Application code)
- API Gateway (HTTP/REST APIs)
- Application Load Balancer (ALB)


# Key Characteristics
- The result is returned immediately after execution
- The caller waits until Lambda finishes processing
- The response includes:
- Execution result (payload)
- Status code
- Error details (if any)


## Error Handling
- Handled by the client (caller)
- Common strategies:
- Retries
- Exponential backoff
- Timeout handling

## Services That Use Synchronous Invocation
User-Triggered (Direct Invocation)
These services typically wait for a response:

- API Gateway
- Application Load Balancer (ALB)
- CloudFront (Lambda@Edge)
- S3 Batch Operations


## Service-to-Service Invocation
AWS services invoking Lambda synchronously:

- Amazon Cognito (e.g., triggers during authentication flows)
- AWS Step Functions (waits for task completion)


Flow:

```js
Client → API Gateway → Lambda → Response → Client
```

### Key Points:
- Caller waits
- Timeout matters (max 15 mins)
- Used in real-time systems

## 🧠 Real-Life Example

Login API:
- User enters credentials
- Lambda verifies from DB
- Returns success/failure instantly

> there is a defined response structure, but it depends on how Lambda is invoked. Don’t assume one universal format

## 1. Direct Invocation (CLI / SDK) – Actual Lambda Response
When you invoke Lambda synchronously using AWS CLI or SDK, the response looks like this:

```js
{
  "StatusCode": 200,
  "ExecutedVersion": "$LATEST",
  "Payload": {
    "message": "Success",
    "data": {}
  },
  "FunctionError": "Unhandled"   // only present if error occurs
}
```

Key Fields:
- StatusCode → HTTP status of invocation (200 = success)
- Payload → Your Lambda function's return value
- FunctionError → Present only if function throws error
- ExecutedVersion → Version of Lambda executed

### 👉 Important:

> Your actual business response is inside Payload, not at top level.

##  2. API Gateway → Lambda (Most Common in Interviews)

Here, you must return a specific structure from Lambda:

```js

{
  "statusCode": 200,
  "headers": {
    "Content-Type": "application/json"
  },
  "body": "{\"message\": \"Success\"}"
}

```

### Important Rules:
- statusCode → Required
- body → Must be stringified JSON
- headers → Optional

👉 If you don’t follow this format:

API Gateway will return 502 Bad Gateway


## 3. ALB → Lambda Response Structure

Very similar to API Gateway:

```js
{
  "statusCode": 200,
  "statusDescription": "200 OK",
  "isBase64Encoded": false,
  "headers": {
    "Content-Type": "application/json"
  },
  "body": "{\"message\": \"Success\"}"
}

```

### Key Interview Insight (Very Important)

“Lambda itself does not enforce a strict response structure. The structure depends on the service invoking it. For example, API Gateway and ALB require a specific response format, while direct SDK/CLI invocation simply returns the payload.”






# 2. Lamda  Asynchronous Invocation (Event-based)

Asynchronous invocation means the caller sends an event to Lambda and does not wait for the response. Lambda processes the request in the background.


##  How It Works
- Event is sent to Lambda
- Lambda queues the event internally
- Function executes asynchronously
- Caller continues without waiting

## Services That Use Asynchronous Invocation

- Amazon S3
- Amazon SNS (Simple Notification Service)
- Amazon EventBridge (CloudWatch Events)
- CloudWatch Logs / Alarms


## Key Characteristics
- Caller does not wait for response
- High scalability (handles large workloads)
- Best for event-driven processing

👉 Example:
Processing 1000 uploaded files → no need to wait → async is ideal

## Retry Behavior (Important)

If the function fails:
- Lambda retries 2 additional times (total 3 attempts)

Retry Timing
- 1st retry → after ~1 minute
- 2nd retry → after ~2 minutes


## Duplicate Execution (Important)

Because of retries:
- Same event can be processed multiple times
- You may see duplicate logs in CloudWatch Logs

# Idempotency
### Definition
Idempotent means performing the same operation multiple times produces the same result as running it once.

### Why Important
Prevents duplicate processing due to retries

Example
- ❌ Non-idempotent: inserting duplicate orders
- ✅ Idempotent: check if order already exists before inserting


# Dead Letter Queue (DLQ)
### Definition
A DLQ stores events that failed processing even after all retries.

### Supported Targets
- Amazon SQS
- Amazon SNS

### Why Use
- Debug failed events
- Reprocess later

👉 Requires correct IAM permissions


# Why Use Asynchronous Invocation
- Faster (no waiting for response)
- Scales automatically
- Ideal for background jobs & bulk processing


# Lambda Execution Role
### Definition

Execution role is an IAM role attached to Lambda that defines what AWS resources the function can access.


# Why It Is Useful

Without an execution role, Lambda cannot access any AWS service.

It is used to:

- Read/write data from S3
- Access DynamoDB / RDS
- Send logs to CloudWatch
- Publish messages to SNS/SQS


## Example

If Lambda needs to read from S3:

```js

{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "*"
}

```

Important Interview Line

> “In asynchronous invocation, Lambda processes events in the background with automatic retries. To avoid duplicate processing due to retries, the function should be idempotent. Execution roles define what resources the Lambda function can access.”

Examples:

- S3 upload triggers Lambda
- SNS → Lambda
- EventBridge → Lambda

Flow:

```js
Event → Lambda → Queue (internal) → Execution
```

Key Points:

- Lambda retries automatically (2 times)
- Events stored temporarily
- Used for background processing

## 🧠 Real-Life Example
User uploads image:

- S3 triggers Lambda
- Lambda resizes image
- User doesn’t wait


# Lambda + EventBridge (Simple Setup)

### What it is

EventBridge triggers a Lambda function based on a schedule or an event.

## Simple Setup Steps

1. Create Lambda
- Go to AWS Lambda → Create function
- Add basic code


2. Create EventBridge Rule
- Go to EventBridge → Rules → Create rule
- Choose:
    - Schedule → e.g. rate(5 minutes)
    OR
    - Event pattern → e.g. EC2 state change


3. Select Target
- Choose Lambda function
- Select your function

### 👉 Permission is added automatically

4. Test
- Wait for schedule or trigger event
- Check logs in CloudWatch

## Example


```bash
Daily at 9 AM → EventBridge triggers Lambda → Lambda processes report
```

## Important Points
- Invocation is asynchronous
- EventBridge handles retry (up to 24 hours)
- Can configure DLQ (SQS)