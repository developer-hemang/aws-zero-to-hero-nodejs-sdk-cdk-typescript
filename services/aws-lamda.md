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




# 🔥 1. What is AWS Lambda (Simple but Strong Definition)

AWS Lambda is a serverless compute service that lets you run code without managing servers.
You just upload your code → AWS handles execution, scaling, patching, availability.

👉 You pay only for execution time (ms level billing).


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


# Lamda + S3 Event Notification Set Up Step By Step

When a file is uploaded to S3, it automatically triggers a Lambda function.

### Architecture Flow

S3 (file upload) → Event Notification → Lambda → Process file

### Step-by-Step Setup
## 1. Create S3 Bucket
- Go to S3 → Create bucket
- Example: my-upload-bucket

## 2. Create Lambda Function
- Go to Lambda → Create function

### Example (Node.js)

```js
exports.handler = async (event) => {
  console.log("S3 Event:", JSON.stringify(event));

  const record = event.Records[0];
  const bucket = record.s3.bucket.name;
  const key = record.s3.object.key;

  console.log(`File uploaded: ${key} in bucket: ${bucket}`);
};

```

## 3. Add Permission (Important)
- When you connect via S3 UI → permission is auto-added
- Otherwise, allow S3 to invoke Lambda


## 4. Configure S3 Event Notification
- Go to S3 bucket → Properties
- Scroll to Event notifications
- Click Create event notification

### Configure:
- Event type → PUT (Object Created)
- Destination → Lambda function
- Select your Lambda

## 5. Test
- Upload a file to S3
- Go to CloudWatch Logs
- You’ll see file details printed


## Important Points (Interview)
- Invocation is asynchronous
- Lambda retries on failure (3 attempts)
- Can configure DLQ (SQS/SNS)
- Make function idempotent (to avoid duplicates)


## Real-Life Example

> User uploads image → Lambda resizes image → stores processed file

# what is Event Mapping In Lamda 

Event Source Mapping is a Lambda feature that automatically polls records/messages from a service and invokes the Lambda function.

👉 Lambda continuously checks the source service for new records and processes them.

## Simple Understanding

For some services, AWS services themselves directly trigger Lambda (like S3 or SNS).

But services like:

- SQS
- Kinesis
- DynamoDB Streams
- Kafka

do not directly push events to Lambda.


### 👉 Instead, Lambda creates an Event Source Mapping that:

- Polls the service
- Reads records/messages
- Sends batches to Lambda

## How It Works

Flow:

- SQS / Stream → Event Source Mapping → Lambda


### Services That Work with Event Source Mapping
- Queue-Based
  - Amazon SQS

- Stream-Based
  - DynamoDB Streams
  - Kinesis Data Streams
  - Amazon MSK (Managed Kafka)
  - Self-managed Apache Kafka


  # Real-Life Example (SQS + Lambda)

## Scenario

E-commerce website:
- Orders are placed
- Orders go into SQS queue
- Lambda processes orders asynchronously


### Flow
- Application sends order message to SQS
- Lambda Event Source Mapping polls queue
- Lambda receives batch of messages
- Processes orders


## Why Use Event Source Mapping?
- Automatic polling
- Scalable processing
- Batch processing support
- Retry handling
- Decoupled architecture


# Important Features
## 1. Batch Processing

Lambda can process multiple records together.

Example

```bash
Batch Size = 10
```

👉 Lambda receives 10 messages in one invocation

## 2. Scaling

Lambda automatically scales based on:

- Queue size
- Stream throughpu

## 3. Retry Behavior
- For SQS:
  - Failed messages return to queue
  - Retry depends on visibility timeout
- For Streams:
  - Lambda retries until records expire or succeed

## 4. Checkpointing (Streams)

For Kinesis/DynamoDB Streams:
- Lambda tracks last processed record
- Prevents data loss


## Important Difference

| Direct Trigger       | Event Source Mapping  |
| -------------------- | --------------------- |
| Service pushes event | Lambda polls service  |
| Example: S3, SNS     | Example: SQS, Kinesis |


## Execution Role Requirement

Lambda execution role needs permissions like:
- sqs:ReceiveMessage
- kinesis:GetRecords
- dynamodb:GetRecords



# AWS Lambda Event Object & Context Object

# 1. Event Object

Definition

The event object contains the input data sent to the Lambda function by the triggering source.

It is the first parameter passed to the Lambda handler

Syntax (Node.js)

```js
exports.handler = async (event, context) => {
    
};
```

Here:

- ```event``` → Input/request data
- ```context```  → Runtime/execution information


## How Event Object Works

The structure of the event object depends on:
- Which AWS service triggered Lambda
- What data that service sends

## Common Event Sources

| Service          | Event Data Contains     |
| ---------------- | ----------------------- |
| API Gateway      | HTTP request details    |
| S3               | Bucket & file info      |
| SQS              | Queue messages          |
| SNS              | Notification message    |
| EventBridge      | Event details           |
| DynamoDB Streams | Database change records |


Real-Life Example

## Scenario

User uploads:

```js
photo.jpg
```

to S3 bucket.

S3 sends event data to Lambda.

## Sample S3 Event Object

```json
{
  "Records": [
    {
      "eventVersion": "2.1",
      "eventSource": "aws:s3",
      "awsRegion": "ap-south-1",
      "eventTime": "2026-05-08T10:00:00.000Z",
      "eventName": "ObjectCreated:Put",
      "s3": {
        "bucket": {
          "name": "my-upload-bucket"
        },
        "object": {
          "key": "photo.jpg",
          "size": 2048
        }
      }
    }
  ]
}
```

Important Fields

| Field       | Meaning                        |
| ----------- | ------------------------------ |
| eventSource | Which service triggered Lambda |
| eventTime   | When event happened            |
| bucket.name | S3 bucket name                 |
| object.key  | Uploaded file name             |
| object.size | File size                      |


## Why Event Object Is Useful

It provides:
- Input data
- Request details
- File info
- Queue messages
- Event metadata

Without event object:
❌ Lambda would not know what to process


# 2. Context Object

## Defination

The context object contains information about the Lambda execution environment and runtime.

It is automatically provided by AWS Lambda.

Why Context Object Is Useful
Used for:

- Request tracking
- Timeout handling
- Monitoring
- Logging
- Debugging

## Important Context Properties

| Property                   | Meaning                  |
| -------------------------- | ------------------------ |
| functionName               | Lambda function name     |
| functionVersion            | Current version          |
| awsRequestId               | Unique invocation ID     |
| memoryLimitInMB            | Allocated memory         |
| getRemainingTimeInMillis() | Remaining execution time |
| invokedFunctionArn         | Lambda ARN               |


## Example Context Object

```json
{
  "functionName": "imageProcessor",
  "functionVersion": "$LATEST",
  "memoryLimitInMB": "128",
  "awsRequestId": "1234-abcd-5678",
  "invokedFunctionArn": "arn:aws:lambda:ap-south-1:123456789:function:imageProcessor"
}
```

## Event vs Context (Very Important)

| Feature               | Event Object   | Context Object       |
| --------------------- | -------------- | -------------------- |
| Purpose               | Input data     | Runtime metadata     |
| Comes From            | Trigger source | AWS Lambda           |
| Used For              | Business logic | Monitoring/debugging |
| Changes Every Request | Yes            | Yes                  |



# AWS Lambda Destinations

### Defination 

AWS Lambda Destinations is a feature that sends the result of an asynchronous Lambda invocation to another AWS service after execution completes.

It can send data:
- On successful execution
- On failed execution


Why Lambda Destinations Are Needed

Normally in asynchronous invocation:
- Caller does not wait for response
- Result is not automatically tracked

Without destinations:
- Hard to know what happened after execution
- Need custom error-handling logic

Lambda Destinations solve this problem.


### How It Works

Flow

Event Source → Lambda → Destination

After Lambda execution:
- Success → Send result to configured success destination
- Failure → Send error details to configured failure destination

Important Point

Lambda Destinations work only with:

✔️ Asynchronous invocations

Examples:
- S3
- SNS
- EventBridge

❌ Not for synchronous invocations like:
- API Gateway
- ALB

Supported Destinations

| Destination | Usage                       |
| ----------- | --------------------------- |
| SQS         | Queue failed/success events |
| SNS         | Send notifications          |
| Lambda      | Trigger another Lambda      |
| EventBridge | Trigger workflows/events    |


Success Destination Example

Real-Life Scenario

## Image Processing System
- User uploads image to S3
- S3 triggers Lambda
- Lambda resizes image successfully
- Lambda sends success event to EventBridge
- EventBridge triggers another workflow

Flow
S3 → Lambda → EventBridge


Failure Destination Example

Real-Life Scenario

Payment Processing
- Payment event triggers Lambda
- Lambda fails due to DB issue
- Failed event sent to SQS
- Support/retry system processes failed events late

Flow
SNS → Lambda → SQS (Failure Destination)

# What Data Is Sent to Destination

Lambda sends:

- Original event
- Response payload
- Error details
- Request context
- Execution metadata


# Example Success Payload

```json
{
  "version": "1.0",
  "timestamp": "2026-05-08T10:00:00Z",
  "requestContext": {
    "requestId": "abc-123",
    "functionArn": "arn:aws:lambda:ap-south-1:123456789:function:imageProcessor",
    "condition": "Success"
  },
  "requestPayload": {
    "fileName": "photo.jpg"
  },
  "responsePayload": {
    "message": "Image resized successfully"
  }
}

```

# Example Failure Payload

```json
{
  "version": "1.0",
  "timestamp": "2026-05-08T10:00:00Z",
  "requestContext": {
    "requestId": "xyz-789",
    "condition": "RetriesExhausted"
  },
  "requestPayload": {
    "orderId": 1001
  },
  "responseContext": {
    "functionError": "Unhandled"
  },
  "responsePayload": {
    "errorMessage": "Database connection failed"
  }
}
```

# Benefits of Lambda Destinations

## 1. Better Error Handling
Automatically capture failed executions.


## 2. Workflow Chaining
Trigger another Lambda or EventBridge after success.


## 3. Easier Monitoring

Track success/failure events centrally.

## 4. No Custom Retry Logic
AWS handles routing automatically.


## 5. Event-Driven Architecture

Helps build loosely coupled systems.


# Success vs Failure Destinations

| Type      | When Triggered             |
| --------- | -------------------------- |
| OnSuccess | After successful execution |
| OnFailure | After all retries fail     |



# Retry Behavior

For async Lambda:

- Lambda retries failed execution
- If retries exhausted:
  - Failure destination triggered


  # Lambda Destinations vs DLQ

  | Feature           | DLQ                 | Lambda Destination         |
| ----------------- | ------------------- | -------------------------- |
| Works On          | Failure only        | Success + Failure          |
| Data Sent         | Original event only | Full execution details     |
| Supported Targets | SQS/SNS only        | SQS/SNS/Lambda/EventBridge |
| More Detailed     | No                  | Yes                        |


# When to Use DLQ vs Destination

### Use DLQ:
- Simple failure storage
### Use Destination:
- Advanced workflows
- Monitoring
- Success/failure tracking
- Chaining services


## Example Architecture

### Order Processing System

Success Flow
API → SNS → Lambda → EventBridge → Billing Service

Failure Flow
API → SNS → Lambda → SQS DLQ


# Simple Setup Steps

## 1. Open Lambda Function
Go To: 

```js
Lambda → Configuration → Destinations
```

## 2. Select Async Invocation
Choose:

```bash
Asynchronous invocation
```

## 3. Configure Destination
### On Success
- EventBridge / SNS / Lambda / SQS
### On Failure
- SQS / SNS

## 4. Save Configuration
AWS automatically handles routing.
