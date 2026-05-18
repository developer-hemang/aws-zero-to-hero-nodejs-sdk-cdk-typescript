# What is AWS Serverless API Gateway ?

AWS API Gateway is a fully managed AWS service used to create, publish, secure, monitor, and manage APIs at any scale.


It acts as a front door for applications to access backend services like:

- AWS Lambda
- EC2
- ECS / Kubernetes
- Any HTTP backend
- AWS services directly

You can think of API Gateway as:

A service that receives client requests, processes them, applies security/rules, and forwards them to backend services.


### API Gateway handles:

- Handle security Authentication & Authorization
- Rate limiting
- Routing
- Request validation
- Monitoring
- Logging
- Versioning
- Handle different environments (dev,test,prod..).
- can create API keys, handle request throtting.
- Swagger / Open API import to quickly defined APIs.
- Transform and validate requests and responses.
- Can Generate SDK and API specifications.
- Cache API responses.

Then forwards request to correct backend.


# API Gateway - Integration High Level

- Lamda Function 
   - invoke Lamda Function
   - Easy Way to expose REST API backed by AWS Lamda

- HTTP
   - Expose HTTP Endpoints in the backend 
   - Example internal HTTP API on premise, Application.
   - WHy ? Add rate limiting, caching , user authentications API keys, etc...

- AWS Services 
   - Expose any AWS API through the API Gateway ?
   - Example start an AWS step function workflow post a message to SQS
   - Why? add authentication , deploy publicly, rate control


# API Gateway Endpoints Types

### Edge-Optimized (default): For global clients
-  request are routed through the CloudFront Edge locations (improves latency)
-  The API Gateway still lives in the only one region

### Regional
-  for clients within the same region
-  could manually combine with CloudFront (more control over the caching strategies and distribution)

### Private: 
-  Can only be accessed from your vpc using an interface VPC endpoint (ENI)
-  Use a resource policy to define access


# API Gateway - Security

### User Authentication through
-  IAM Roles (useful for internal applications)
-  Cognito (identity for external users - Example mobile users)
-  Custom Authorizer (your own logic generally Lamda function)

### Custom Domain Names HTTPS security through itegration with AWS Certificate Manager (ACM)
-  if using Edge-Optimized endpoint , then the certificate must be in us-east-1
-  if using Regional endpoint the certificate must be in the API-Gateway region
-  Must setup CNAME or A-alias record in Route 53


# Why Use API Gateway?

### Without API Gateway:

- Clients directly access backend
- Security becomes difficult
- No centralized control
- Hard to monitor APIs

### With API Gateway:

- Single entry point
- Better security
- Monitoring
- Traffic management
- Scalability


# Main Features of API Gateway

### 1. API Creation

You can create:
- REST APIs
- HTTP APIs
- WebSocket APIs


### 2. Authentication & Authorization

Supports:
- IAM authentication
- Cognito User Pools
- Lambda Authorizers
- JWT authentication
- API Keys


### 3. Rate Limiting / Throttling

Protect backend from overload.

Example:

```bash
100 requests/sec
```

If exceeded:

```bash
429 Too Many Requests
```

### 4. Request Validation

API Gateway can validate:
- Headers
- Query parameters
- JSON body

Before request reaches backend.


### 5. Monitoring

Integrated with:
- CloudWatch
- X-Ray

You can monitor:
- Latency
- Errors
- Request count

### 6. Caching

Example:

```bash
GET /products
```
Frequently used responses can be served faster.


### 7. Security

Supports:
- HTTPS
- WAF
- Resource policies
- CORS
- Authentication


# Types of API Gateway APIs

## 1. REST API

Oldest and most feature-rich.

Supports:
- API keys
- Usage plans
- Request transformation
- Advanced security

Best for:

- Enterprise APIs
Example:

```bash
GET /users
POST /orders
```


# 2. HTTP API

Lightweight and cheaper.

Faster than REST API.

Supports:
- Lambda
- JWT auth
- OIDC
- OAuth

Best for:
- Modern serverless apps

Cost is lower than REST API.

3. WebSocket API

For real-time communication.
Examples:
- Chat app
- Live notifications
- Multiplayer games


## API Gateway Flow

```bash
Mobile App
   |
POST /login
   |
API Gateway
   |
Lambda Function
   |
DynamoDB
```

Flow:
1. Client sends request
2. API Gateway receives it
3. Validates/authenticates
4. Sends to Lambda
5. Lambda processes
6. Response returned through API Gateway



# Important Concepts

## 1. Resources

Represents URL paths.
Example:

```bash
/users
/orders
/products
```

## 2. Methods

HTTP methods:
- GET
- POST
- PUT
- DELETE
- PATCH
- OPTIONS

## 3. Stages

A named environment where a specific deployment of an API is published.

Stages allow you to maintain different environments for the same API.

You don’t directly release new code to customers.
Each environment is a separate API Gateway stage.


## Each stage can:
- Use different Lambda aliases
- Use different configs
- Have different throttling
- Have different logs


Example 

```bash
dev
qa
prod
```

URL:

```js
https://abc.execute-api.us-east-1.amazonaws.com/prod
```

## Why API Gateway Stages Are Important?

Stages help with:

| Use                      | Explanation             |
| ------------------------ | ----------------------- |
| Environment separation   | Dev vs Prod             |
| Safe deployments         | Test before release     |
| Configuration management | Different settings      |
| Monitoring               | Separate logs/metrics   |
| CI/CD                    | Automated deployments   |
| Rollback                 | Deploy previous version |


## API Gateway Stage Variables

Stage variables are like enviroment variables for API Gateway, Stage variables are key-value configs stored at stage level.
- Use them to change often changing configuration values
- They can be used in:
   - Lambda function ARN
   - HTTP Endpoint
   - Parameter mapping templates


- Use cases:
   -  Configure HTTP endpoints your stages talk to (dev, test, prod)
   -  Pass configuration parameters to AWS Lambda through mapping templates
- Stage variables are passed to the "context" object in aws Lambda 
- Format: ${stageVariables:variableName}


## 4. Deployment

