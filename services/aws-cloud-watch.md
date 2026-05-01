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

## Example How to PUT custom Metrics Using AWS CLI

```bash

aws cloudwatch put-metric-data \
  --namespace "MyApp/Metrics" \
  --metric-name "LoginSuccess" \
  --value 1 \
  --unit Count \
  --dimensions Service=Auth,Environment=Prod

```

## Example How to PUT custom Metrics Using AWS CLI

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

# 🚨 Alarm Example
- Condition:
    - Duration > 2 seconds for 5 mins


# “CloudWatch accepts metric data points up to 2 weeks in the past and 2 hours in the future”


