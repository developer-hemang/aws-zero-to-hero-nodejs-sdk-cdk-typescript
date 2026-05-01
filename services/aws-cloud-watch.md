# ☁️ AWS CloudWatch Metrics 

CloudWatch Metrics are numerical time-series data points collected and stored by AWS to monitor the performance and health of resources and applications.

👉 In simple terms:

>> Metrics = numbers over time (e.g., CPU usage, error count, request count)

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

- 1. Monitoring system health
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

>> “CloudWatch metrics help monitor system performance and enable alerting and automated actions based on thresholds.”