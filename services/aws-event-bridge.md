# ☁️ What is EventBridge?


Amazon EventBridge is a serverless event bus service that allows different applications and AWS services to communicate with each other using events, enabling loosely coupled and event-driven architectures.

# 🧠 Why do we need EventBridge?

## ❌ Problem without EventBridge

Services are tightly coupled:

```bash
Service A → directly calls → Service B
```

Problems:

- Hard to scale
- Hard to maintain
- Changes break system

## ✅ With EventBridge

```js
Service A → EventBridge → Service B / C / D
````

Benefits:
- Decoupling
- Scalability
- Flexibility

# 🔁 How EventBridge Works

```js
Event Source → Event Bus → Rule → Target
```