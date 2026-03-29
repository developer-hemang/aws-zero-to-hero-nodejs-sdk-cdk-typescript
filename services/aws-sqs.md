# :one:  What is AWS SQS?

Aws SQS is a fully managed distributed message queuing service provided by Amazon Web Services that allows different component of an application to communicate with each other asynchronously by sending and receiving messages through a queue

In a typical distributed system, multiple services need to interact with each other. If these services communicate directly, the system becomes tightly coupled. This means if one service fails, becomes slow, or is unavailable, it can affect other services that depend on it.

AWS SQS solves this problem by acting as an intermediate message storage layer between services.

Instead of communicating directly:

- One service sends a message to a queue
- Another service retrieves the message from the queue and processes it

Because of this mechanism, services become independent and loosely coupled, improving system reliability and scalability.

In simple words:

AWS SQS is a service that temporarily stores messages in a queue so that different services of an application can communicate with each other asynchronously and reliably.


# 2. Why AWS SQS is Needed

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


# 3. Real World Analogy