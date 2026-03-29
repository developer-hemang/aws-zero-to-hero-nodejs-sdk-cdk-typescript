# :one: What is AWS SQS?

Aws SQS is a fully managed distributed message queuing service provided by Amazon Web Services that allows different component of an application to communicate with each other asynchronously by sending and receiving messages through a queue

In a typical distributed system, multiple services need to interact with each other. If these services communicate directly, the system becomes tightly coupled. This means if one service fails, becomes slow, or is unavailable, it can affect other services that depend on it.

AWS SQS solves this problem by acting as an intermediate message storage layer between services.

Instead of communicating directly:

- One service sends a message to a queue
- Another service retrieves the message from the queue and processes it

Because of this mechanism, services become independent and loosely coupled, improving system reliability and scalability.

In simple words:

AWS SQS is a service that temporarily stores messages in a queue so that different services of an application can communicate with each other asynchronously and reliably.


# :two: Why AWS SQS is Needed

In modern systems such as microservices architectures, applications often consist of multiple services performing different tasks. For example:

- Order Service
- Payment Service
- Email Service
- Notification Service
- Analytics Service

If these services communicate synchronously, the request chain can become slow and fragile.

Example without SQS:

```js
Client → Order API → Payment Service → Email Service → Analytics Service
```

Problems with this approach:

- If any service becomes slow, the entire request slows down.
- If one service fails, the whole process fails.
- The API must wait for all tasks to finish.

Using SQS, tasks can be handled asynchronously.

```js 
Client → Order API → SQS Queue → Worker Services
```

Now the API only needs to:

1 Receive the request 
2 Send the task to the queue 
3 Return response immediately

Workers will process tasks later.

benifits 

- Faster API response
- Better reliability
- Easy scaling
- Independent services


# :three: Real World Analogy

🧑‍🍳 A good way to understand SQS is by comparing it to a restaurant order system.

🤔 Imagine the following process:

1. A customer places an order with the waiter.
2. The waiter writes the order and places it on a kitchen order board.
3. The chef picks orders from the board one by one and prepares them.

In this scenario:

| System Component | Real World Example  |
| ---------------- | ------------------- |
| Producer         | Waiter              |
| Queue            | Kitchen order board |
| Consumer         | Chef                |


😁 Important point:

The waiter does not wait in the kitchen for the chef to cook the meal. Instead, the order is placed in the queue and processed when the chef is available.

This is exactly how SQS works in distributed systems.

# :four: Core Components of AWS SQS

There are three primary components involved when working with SQS.

## Producer
 
 A producer is the application or service that creates and sends messages to the queue.

Example:

- Node.js API
- Payment service
- Order service

The producer's responsibility is only to push messages to the queue.

## Queue

Queue is the temporary storage location where messages are held untill they are processed.

AWS manages the queue infrastructure automatically, including:

- storage
- replication
- reliability
- scaling

Messages remain in the queue until a consumer retrieves and deletes them.

## Consumer

A consumer is a service that retrieves messages from the queue and processes them.

Examples:

- Email sending worker
- Image processing service
- Notification service

Consumers continuously poll the queue for new messages.

# :five: Types of AWS SQS Queues

### AWS provides two types of queues.

---

## :one: Standard Queue

Standard Queue is a default and most commaonly used type of SQS queue. it is designed for high throughput and maximum scalablity.

Standard queues provide at-least-once message delivery, which means a message might be delivered more than once in rare situations.


Because of the distributed nature of the system, strict ordering of messages is not guaranteed, although most of the time messages are delivered in the correct order.


Characteristics of Standard Queue:

- Nearly unlimited throughput
- Messages may occasionally be delivered more than once
- Message ordering is not guaranteed
- Extremely scalable and fast

Standard queues are suitable for most applications where absolute ordering is not critical.


## :two: FIFO Queue (First-In-First-Out)

A FIFO queue guarantees that messages are processed exactly in the order they were sent.
This type of queue ensures that each message is processed exactly once, preventing duplicate processing.

FIFO queues are typically used in applications where order matters, such as financial transactions or inventory updates.

Characteristics of FIFO Queue:

- Guaranteed message ordering
- Exactly-once processing
- Lower throughput compared to Standard Queue
- Queue name must end with .fifo

Example queue name:

```js
order-processing.fifo
```

# :six: SQS Message

A message is the unit of data that is sent from a producer to a queue.
A message can contain any type of information needed by the consumer to perform its task.

Example message:

```js
{
  "orderId": 101,
  "userId": 25,
  "email": "user@example.com",
  "type": "ORDER_CONFIRMATION"
}
```

Message size limit:

```js
Maximum 256 KB
```

For larger payloads, AWS recommends using S3 + SQS combination.


# :seven: Message Attributes

Message attributes are additional metadata attached to a message.

They provide extra information about the message without modifying the message body.

### Example attributes:

```js
Priority: High
Type: Email
Source: OrderService
```
Consumers can use these attributes to make decisions during processing.

# :eight: Visibility Timeout

Visibility timeout is one of the most important concepts in SQS.

When a consumer retrieves a message from the queue, the message becomes temporarily invisible to other consumers for a specified time.

This prevents multiple consumers from processing the same message simultaneously.

### Example scenario:

1. Worker retrieves a message.
2. Visibility timeout is set to 30 seconds.
3. Other workers cannot see this message during that time.


If the worker successfully processes the message, it deletes the message from the queue.
If the worker crashes or fails before deleting the message, the visibility timeout expires and the message becomes visible again.
This ensures that messages are never lost.

# :nine: Message Retention Period

Message retention period determines how long messages remain in the queue if they are not processed.

Default retention period:

```js
4 days
```

Maximum retention period:

```js
14 days
```

If a message is not consumed within this time, it is automatically deleted.

# :one::zero: Polling

Consumers retrieve messages from SQS by polling the queue.

There are two polling methods:

## Short Polling
The consumer checks the queue frequently for messages.
This can lead to many empty responses when no messages are available.

## Long Polling
Long polling allows the consumer to wait for messages for a specified time before returning an empty response.

Example:

```js
WaitTimeSeconds = 20
```

This reduces unnecessary API calls and improves efficiency.


# :one::one: Dead Letter Queue (DLQ)

A Dead Letter Queue is a special queue used to store messages that cannot be successfully processed.

Example scenario:

1. Worker processes message
2. Processing fails
3. Message is retried multiple times
4. After exceeding retry limit → moved to DLQ

This helps developers analyze failed messages without losing them.


# :one::two: SQS Message Lifecycle

The lifecycle of a message in SQS follows several stages.


1. A producer sends a message to the queue.
2. The message is stored in the queue.
3. A consumer retrieves the message.
4. The message becomes temporarily invisible due to visibility timeout.
5. The consumer processes the message.
6. The consumer deletes the message from the queue.


If the consumer fails to delete the message, it becomes visible again for other consumers.


# :one::three:  What is an SQS Access Policy?

An SQS Access Policy is a JSON-based permission document that defines who is allowed to perform which actions on an SQS queue.

It works similarly to AWS IAM policies, but it is attached directly to the queue itself, making it a resource-based policy.

In simple terms:

> An SQS access policy controls which AWS users, roles, accounts, or services can send messages to the queue, receive messages from the queue, or manage the queue.


This policy helps enforce security and controlled access to the queue.

### For example, you may want:

- Only a specific application server to send messages.
- Only a worker service to read messages.
- Allow SNS topic to publish messages.
- Allow Lambda function to consume messages.


All of these are controlled using SQS access policies.

# :one::four: Why Access Policies Are Needed?

In real production systems, queues are often accessed by multiple services.

Example architecture:

```js
Order Service → SQS → Email Worker
                     ↓
                Analytics Worker

```


### Without access control:

- Anyone with AWS credentials could access the queue.
- Unauthorized services might read or send messages.
- Security risks increase.

### Using access policies allows you to:

- Restrict who can send messages
- Restrict who can consume messages
- Allow only specific AWS services
- Allow cross-account access


# :one::five: Types of Permissions in SQS

Common SQS actions that can be controlled through policies include:

| Action                   | Description               |
| ------------------------ | ------------------------- |
| `sqs:SendMessage`        | Send message to queue     |
| `sqs:ReceiveMessage`     | Retrieve messages         |
| `sqs:DeleteMessage`      | Delete processed messages |
| `sqs:GetQueueAttributes` | Read queue configuration  |
| `sqs:SetQueueAttributes` | Modify queue settings     |
| `sqs:PurgeQueue`         | Delete all messages       |


# :one::six:  Structure of an SQS Access Policy

An SQS access policy is written in JSON format and contains several key fields.

Example structure:

```js
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "sqs:SendMessage",
      "Resource": "QUEUE_ARN"
    }
  ]
}

```
Explanation:

| Field     | Meaning                  |
| --------- | ------------------------ |
| Version   | Policy language version  |
| Statement | List of permission rules |
| Effect    | Allow or Deny            |
| Principal | Who gets permission      |
| Action    | Allowed operations       |
| Resource  | Queue ARN                |



# Example :one: Allow Only One IAM User to send Message

Suppose you want only a specific IAM user to send messages.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowSendMessage",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/backend-api"
      },
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:ap-south-1:123456789012:email-queue"
    }
  ]
}
```

Explanation:

- Only the IAM user backend-api can send messages.
- Other users cannot send messages.


# Example :two: Allow Workers to Receive Messages

Suppose you have worker servers that process queue jobs.

Policy example:

```js
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowWorkersToConsume",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/email-worker-role"
      },
      "Action": [
        "sqs:ReceiveMessage",
        "sqs:DeleteMessage",
        "sqs:GetQueueAttributes"
      ],
      "Resource": "arn:aws:sqs:ap-south-1:123456789012:email-queue"
    }
  ]
}

```
Now:

- Worker role can receive and delete messages.
- Others cannot consume messages.


# Example :three: Allow SNS Topic to Send Messages to SQS

A very common architecture is:

```js
SNS Topic → SQS Queue
```

This allows multiple services to publish events.


```js
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowSNSPublish",
      "Effect": "Allow",
      "Principal": {
        "Service": "sns.amazonaws.com"
      },
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:ap-south-1:123456789012:email-queue",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": "arn:aws:sns:ap-south-1:123456789012:order-topic"
        }
      }
    }
  ]
}

```

# :one::seven: We will implement a simple architecture:

```js
Client → Node API (Producer) → AWS SQS → Worker Service (Consumer)
```

Example Use Case:


```js
User Signup → Queue → Email Worker sends welcome email
```


# Project Structure (Production Grade)

### A clean architecture keeps SQS logic separated from business logic.

```js
sqs-nodejs-example
│
├── src
│   ├── config
│   │   └── aws.js
│   │
│   ├── queues
│   │   ├── producer.js
│   │   └── consumer.js
│   │
│   ├── services
│   │   └── emailService.js
│   │
│   ├── workers
│   │   └── emailWorker.js
│   │
│   ├── routes
│   │   └── userRoutes.js
│   │
│   ├── controllers
│   │   └── userController.js
│   │
│   ├── utils
│   │   └── logger.js
│   │
│   └── app.js
│
├── worker.js
├── package.json
└── .env
```

## This structure follows common Node.js production patterns:

- Config
- Services
- Controllers
- Workers
- Queues



## Install Required Packages & AWS SDK

```js
npm init -y
npm install express dotenv @aws-sdk/client-sqs
```

## Environment Variables

`.env`

```js
PORT=3000
AWS_REGION=ap-south-1
AWS_ACCESS_KEY=your_access_key
AWS_SECRET_KEY=your_secret_key

SQS_QUEUE_URL=https://sqs.ap-south-1.amazonaws.com/123456789012/email-queue

```
## AWS SQS Configuration

``src/config/aws.js``

```js
import { SQSClient } from "@aws-sdk/client-sqs";
import dotenv from "dotenv";

dotenv.config();

export const sqsClient = new SQSClient({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY,
    secretAccessKey: process.env.AWS_SECRET_KEY
  }
});

```

This creates a reusable SQS client instance.

## Logger Utility
Production apps should never rely only on ``console.log``.

``src/utils/logger.js`` 

```js
export const logger = {
  info: (message, data = null) => {
    console.log(`[INFO] ${message}`, data || "");
  },

  error: (message, error = null) => {
    console.error(`[ERROR] ${message}`, error || "");
  }
};
```

In real production systems, you'd use:

```js
winston
pino
```

# SQS Producer (Send Messages)

``src/queues/producer.js``


```js
import { SendMessageCommand } from "@aws-sdk/client-sqs";
import { sqsClient } from "../config/aws.js";
import { logger } from "../utils/logger.js";

const QUEUE_URL = process.env.SQS_QUEUE_URL;

export const sendMessageToQueue = async (payload) => {
  try {

    const params = {
      QueueUrl: QUEUE_URL,
      MessageBody: JSON.stringify(payload)
    };

    const command = new SendMessageCommand(params);

    const response = await sqsClient.send(command);

    logger.info("Message sent to SQS", response.MessageId);

  } catch (error) {
    logger.error("Failed to send message to SQS", error);
    throw error;
  }
};
```

This function acts as a generic queue publisher.

## Business Service Example

``src/services/emailService.js``

```js
import { logger } from "../utils/logger.js";

export const sendWelcomeEmail = async (email, name) => {

  logger.info(`Sending welcome email to ${email}`);

  // simulate email sending
  await new Promise((resolve) => setTimeout(resolve, 2000));

  logger.info(`Email sent successfully to ${email}`);

};

```

In production this could use:

```js
AWS SES
SendGrid
Mailgun
```

