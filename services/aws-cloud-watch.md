# ☁️ AWS CloudWatch Metrics 

CloudWatch Metrics are numerical time-series data points collected and stored by AWS to monitor the performance and health of resources and applications.

# Strong Answer

“CloudWatch metrics are system-level metrics automatically collected by AWS, such as Lambda errors or latency. Custom metrics are application-specific metrics that we push using putMetricData to track business events like payment success or order count. Custom metrics are especially useful for meaningful alerting and product insights.”

👉 In simple terms:

> Metrics = numbers over time (e.g., CPU usage, error count, request count)

## 🔍 Examples of Default AWS Metrics

## AWS automatically provides these:
- Lambda:
    - Invocations
    - Errors
    - Duration
    - Throttles
- EC2:
    - CPUUtilization
- API Gateway:
    - Latency
    - 4XX / 5XX errors

## 🎯 Use of CloudWatch Metrics

## 1. Monitoring system health
 - Is your Lambda failing?
 - Is latency increasing?

## 2. Alerting (MOST IMPORTANT USE)
- Create alarms:
 - Error rate > threshold → send alert (SNS)

## 3. Performance tracking
 - Track response time trends
 - Detect bottlenecks

## 4. Auto scaling (in EC2-based systems)
- Scale up when CPU increases


### 👉 Interview one-liner:

> “CloudWatch metrics help monitor system performance and enable alerting and automated actions based on thresholds.”


# 🧩 What are CloudWatch Custom Metrics?'
Custom Metrics are user-defined metrics that you explicitly send to CloudWatch to track application-specific or business-level data that AWS does not provide by default.


## 🔍 Examples of Custom Metrics
- OrdersPlaced
- PaymentSuccess / PaymentFailure
- UserSignupCount
- CartAbandonmentRate
- Third-party API response time

# 🎯 Use of Custom Metrics

## 1. 📦 Business-level monitoring (VERY IMPORTANT)

AWS doesn’t know your business.

So you track:
- “Are payments failing?”
- “How many orders per minute?”

## 2. 🚨 Better alerting

Instead of:
 - Lambda Errors ❌

Use:

- PaymentFailure > threshold ✅

👉 Much more meaningful


## 3. 📊 Product insights
- Feature usage tracking
- A/B testing data


## 4. ⚡ Fine-grained observability
- Track specific flows (checkout, login, etc.)

## 🧠 Example (Node.js Logic)

```js

if (paymentSuccess) {
  sendMetric("PaymentSuccess", 1);
} else {
  sendMetric("PaymentFailure", 1);
}

```

## Example How TO PUT Custom MEtrics Using AWS NOde JS SDK

```js
import { CloudWatchClient, PutMetricDataCommand } from "@aws-sdk/client-cloudwatch";

const cw = new CloudWatchClient({ region: "ap-south-1" });

export async function putMetric() {
  const command = new PutMetricDataCommand({
    Namespace: "MyApp/Metrics",
    MetricData: [
      {
        MetricName: "LoginSuccess",
        Value: 1,
        Unit: "Count",
        Dimensions: [
          { Name: "Service", Value: "Auth" }
        ]
      }
    ]
  });

  await cw.send(command);
  console.log("Metric sent");
}

// call
putMetric();

```

## Example How to Get Custom Metrics Using AWS NodeJS SDK


```js
import { CloudWatchClient, GetMetricStatisticsCommand } from "@aws-sdk/client-cloudwatch";

const cw = new CloudWatchClient({ region: "ap-south-1" });

export async function getMetrics() {
  const command = new GetMetricStatisticsCommand({
    Namespace: "MyApp/Metrics",
    MetricName: "LoginSuccess",
    Dimensions: [
      { Name: "Service", Value: "Auth" }
    ],
    StartTime: new Date(Date.now() - 60 * 60 * 1000), // last 1 hour
    EndTime: new Date(),
    Period: 300, // 5 minutes
    Statistics: ["Sum"]
  });

  const data = await cw.send(command);
  console.log(data.Datapoints);
}

// call
getMetrics();
```

## Example How to PUT custom Metrics Using AWS CLI

```bash

aws cloudwatch put-metric-data \
  --namespace "MyApp/Metrics" \
  --metric-name "LoginSuccess" \
  --value 1 \
  --unit Count \
  --dimensions Service=Auth,Environment=Prod

```

## Example How to GET custom Metrics Using AWS CLI

```bash
aws cloudwatch get-metric-statistics \
  --namespace "MyApp/Metrics" \
  --metric-name "LoginSuccess" \
  --dimensions Name=Service,Value=Auth \
  --start-time 2026-05-01T09:00:00Z \
  --end-time 2026-05-01T10:00:00Z \
  --period 300 \
  --statistics Sum
```
### “CloudWatch accepts metric data points up to 2 weeks in the past and 2 hours in the future”

# 🔍 1. How do you know “Is Lambda failing?”

## ✅ Using CloudWatch Metrics

AWS automatically gives you this metric:

- ```Errors```
- ```Invocations```

## 📊 Error Rate Formula

```js
Error Rate = Errors / Invocations
```

### 👉 How to check:
- Go to CloudWatch → Metrics → Lambda
- Select your function
- View:
    - Errors
    - Invocations

## 🚨 Set Alarm (Real Production Way)

Example:

- Condition:
    - Errors > 5 in 5 minutes

👉 Action:

- Trigger SNS → Email/Slack


### 💻 CLI Example

```js 
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Errors \
  --dimensions Name=FunctionName,Value=my-function \
  --start-time 2026-05-01T00:00:00Z \
  --end-time 2026-05-01T01:00:00Z \
  --period 300 \
  --statistics Sum
```

### 🧠 Logs (for root cause)
- Go to CloudWatch Logs
- Check:
    - Stack trace
    - Error message

👉 Metrics tell “something is wrong”
👉 Logs tell “what exactly went wrong”


# ⏱ 2. How do you know “Is latency increasing?”

✅ Lambda Duration Metric

AWS gives:
- Duration (in milliseconds)

👉 How to check:
- CloudWatch → Metrics → Lambda → Duration

Look for:

Average
p95 / p99 (important for senior roles)

📊 API Gateway Latency

Also check:
- Latency
- IntegrationLatency

👉 Why?
- Helps identify:
    - API issue vs Lambda issue

#### 🚨 Alarm Example
- Condition:
    - Duration > 2 seconds for 5 mins

---

# ☁️ What is CloudWatch Logs?

CloudWatch Logs is a service that collects, stores, and analyzes log data from AWS resources (like Lambda, API Gateway, EC2) and applications.

A Log Group in CloudWatch is a logical container that organizes and stores logs generated by a specific application, service, or AWS resource, such as a Lambda function or API Gateway. It acts as the top-level structure where you manage log retention, access control, and filtering.

## 🧠 Real Understanding

```bash
Application → Folder → Files
             ↑
         Log Group
```

## 🎯 Why it matters

- Keeps logs organized per service
- Allows:
  - Retention control (cost optimization)
  - IAM permissions
  - Monitoring setup

## Practicle Example OF Log Group

You have a service:

```bash
User → API Gateway → Lambda (order-service)
```

AWS automatically creates:

```bash
/aws/lambda/order-service
```

### 💻 Practical Example (Node.js Lambda)

```js
export const handler = async () => {
  console.log("Order received");
  return { statusCode: 200 };
};
```

👉 Output goes to:

```bash
CloudWatch → Log Group → /aws/lambda/order-service
```
## 🎯 Real Use
- Debug failed API calls
- Track request flow


# ☁️ 2. Log Streams

A Log Stream is a sequence of log events generated from a single source within a log group, such as one Lambda execution environment or one EC2 instance.

### 🧠 Real Meaning
Inside one log group:

```bash
Log Group (Payment Service)
 ├── Stream 1 (Lambda instance A)
 ├── Stream 2 (Lambda instance B)
```

# 🎯 Why important
- Separates logs per execution instance
- Helps debug specific failures

🎤 Interview Answer

“Log Streams represent individual sources of logs within a log group, helping isolate logs from different instances or executions for easier debugging.”

# 🎯 Real Use
  - Debug specific request failure
  - Identify which instance failed

# ☁️ 3. Metric Filters (Logs → Metrics)

Metric Filters are a feature in CloudWatch Logs that scan log data for specific patterns and convert matching occurrences into numerical metrics, which can then be used for monitoring and alerting.

### 🧠 Real Meaning

```js
{ "level": "error", "message": "Payment failed" }
```

Logs:

```js
{ "level": "error", "message": "Payment failed" }
```
Converted into:

```bash
ErrorCount = 1
```

### 🎯 Why needed
- Logs are hard to monitor ❌
- Metrics are easy to alert on ✅

### 🎤 Interview Answer
Metric filters allow us to extract meaningful signals from logs by converting specific patterns into metrics, enabling alerting on application-level events like failures.”

# Practical USE CASES OF Metrics Filters

🧠 Scenario

Your logs:

```js
{ "level": "error", "message": "Payment failed" }
```

## ⚙️ You create filter:

```js
{ $.level = "error" }
```

👉 CloudWatch creates metric:

```js
PaymentErrors = count
```

## 🚀 Real Use Case

```js
If PaymentErrors > 10 → trigger alert
```


💻 Practical CLI

```bash
aws logs put-metric-filter \
  --log-group-name "/aws/lambda/payment-service" \
  --filter-name "PaymentErrorFilter" \
  --filter-pattern '"Payment failed"' \
  --metric-transformations \
    metricName=PaymentErrors,metricNamespace=MyApp,metricValue=1

```


# ☁️ 4. CloudWatch Metrics

CloudWatch Metrics are time-based numerical data points that represent the performance and health of AWS resources and applications, collected either automatically by AWS or custom-defined by developers.

## 🧠 Real Meaning
Examples:
- Errors over time
- Latency trends
- Request count

## 🎯 Why important
- Core of monitoring system
- Drives alarms

### 🎤 Interview Answer
“Metrics are time-series data used to monitor system behavior, such as error rates or latency, and are essential for triggering alerts and detecting anomalies.”


# ☁️ 4. CloudWatch Metrics (Practical)

## 🧠 Scenario

You track business metric:
- Orders placed

### 💻 Node.js Example

```js
await cloudwatch.send(new PutMetricDataCommand({
  Namespace: "MyApp",
  MetricData: [{
    MetricName: "OrdersPlaced",
    Value: 1,
    Unit: "Count"
  }]
}));

```


### 🎯 Real Use
- Track business KPIs
- Build dashboards


# ☁️ 5. CloudWatch Alarms

## ✅ Definition
A CloudWatch Alarm is a monitoring mechanism that evaluates a metric against a defined threshold over time and triggers actions when the condition is met.

## 🧠 Real Meaning
```bash
If Errors > 5 → Send alert
```

# 🎯 Why important
- Enables proactive monitoring
- Prevents downtime impact


## 🎤 Interview Answer

CloudWatch Alarms continuously evaluate metrics and trigger automated actions like notifications or remediation when thresholds are breached

# ☁️ CloudWatch Alarms (VERY IMPORTANT – Practical + Targets)

✅ Scenario

Metric:

```bash
Lambda Errors
```

### 🚨 Alarm Condition

```bash
Errors > 5 in 5 minutes
```

## 🔥 Alarm Targets

### CloudWatch Alarm can trigger:

## 1. SNS (Most Common)

```bash
Alarm → SNS → Email / Slack / SMS
```

## 2. Lambda Function

```js
Alarm → Lambda → Auto-fix issue
```

Example:
- Restart process
- Clear cache

## 3. Auto Scaling
- Alarm → Scale EC2 instances

## 4. Systems Manager (SSM)
Run automation scripts

# 5. EC2 Actions
- Stop / terminate / reboot instance

### 🧠 Real Production Flow

```bash
Errors ↑
 → Alarm triggered
 → SNS
 → Slack alert to DevOps
```

## 💻 CLI Example

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "HighLambdaErrors" \
  --metric-name Errors \
  --namespace AWS/Lambda \
  --threshold 5 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --period 300 \
  --statistic Sum \
  --alarm-actions arn:aws:sns:ap-south-1:123456789012:my-topic
```

## 🎯 Real Use
- Production alerting
- Auto-remediation



# ☁️ 6. Composite Alarms

## ✅ Definition
Composite Alarms are advanced alarms that combine multiple alarms using logical conditions (AND/OR) to produce a single, more meaningful alert.

### 🧠 Real Meaning

```bash
Alert only if:
Errors HIGH AND Latency HIGH
```

### 🎯 Why important
- Reduces alert noise
- Avoids false positives


### 🎤 Interview Answer

> “Composite alarms help reduce alert fatigue by combining multiple conditions, ensuring alerts are triggered only when there is a real issue.”

# ☁️ 7. Subscription Filters


✅ Definition

Subscription Filters allow you to stream log data in real time from CloudWatch Logs to other AWS services like Lambda, Kinesis, or OpenSearch for further processing or analysis.


## 🧠 Real Meaning

```bash
Logs → Lambda → Slack Alert
```

## 🎯 Why important
- Real-time processing
- Central logging systems


## 💻 Example Use Case
- Detect security issue
- Send alert immediately

## 🎯 Real Use
- Real-time log processing
- Central logging system

## 🎤 Interview Answer

> “Subscription filters enable real-time log streaming to downstream systems for processing, alerting, or analytics.”


# ☁️ 8. CloudWatch Logs Insights

### ✅ Definition

CloudWatch Logs Insights is an interactive query service that allows you to search, analyze, and visualize log data using a powerful query language.


### 🧠 Real Meaning

Like SQL for logs:

```bash
filter error logs
sort latest
```
### 🎯 Why important
- Fast debugging
- Analyze large logs


### 🎤 Interview Answer

> “Logs Insights allows efficient querying of large-scale logs using a structured query language, making debugging faster and more effective.”

# Logs Insights (Practical)

### 🧠 Scenario

Find all failed payments:

```sql
fields @timestamp, message
| filter message like /Payment failed/
| sort @timestamp desc
```

### 🎯 Real Use
Debug production issues fast


# ☁️ 9. CloudWatch Live Tail

### ✅ Definition

CloudWatch Live Tail provides real-time streaming of log events as they are generated, allowing developers to observe system behavior instantly.

# 🎯 Why important
- Debug live deployments
- Monitor production issues

# 🎤 Interview Answer

> “Live Tail helps in real-time debugging by streaming logs as they are generated, especially useful during deployments or incident handling.”

## 🧠 Scenario
You deploy new version:

```bash
Watch logs live → detect error immediately
```
## 🎯 Real Use
Deployment monitoring
Live debugging

# ☁️ 10. CloudWatch Agent vs Logs Agent

## ✅ CloudWatch Agent

The CloudWatch Agent is a unified monitoring agent that you install on servers (such as EC2 instances, on-prem machines, or virtual machines) to collect system-level metrics and application logs and send them to Amazon CloudWatch for monitoring and analysis.

## ❌ Logs Agent (Old)

Older version used only for logs; now replaced by unified agent.

## 🧠 Why do we need CloudWatch Agent?

Out of the box, AWS gives limited metrics:

### ❌ By default (EC2):
- CPU utilization ✅
- Network traffic ✅
- Disk I/O ✅
- Memory usage ❌
- Disk usage ❌
- Custom

### ✅ CloudWatch Agent solves this
It allows you to collect:
- Memory usage (RAM)
- Disk usage (% full)
- Swap usage
- Custom application logs (e.g., /var/log/app.log)


## 🎯 Real Use Case
Scenario:

You have a Node.js app running on EC2

👉 You want:
- RAM usage alert
- Disk almost full alert
- App logs in CloudWatch

👉 Solution:

- Install CloudWatch Agent
- Configure metrics + logs
- Create alarms

## 🎤 Interview Answer

> “The CloudWatch Agent is a unified tool for collecting both system metrics and logs, while the older Logs Agent was limited and is now deprecated.”


# ☁️ 11. CloudWatch Synthetics

### ✅ Definition

CloudWatch Synthetics allows you to create canaries that simulate real user interactions with your application to continuously test availability and performance.

### 🧠 Real Meaning

```bash
Every 5 mins → Call API → Check response
```

### 🎯 Why important
Detect issues before users
Monitor uptime

### 🧠 Scenario

```bash
Every 5 min:
 → Call API
 → Check response = 200
```

### 🎯 Real Use
- Detect downtime before users
- SLA monitoring