# AWS SNS, SQS, and DLQ — Key Concepts

This document summarizes key AWS concepts related to message delivery guarantees, dead-letter queues, and how to set up notifications for failed messages.

---

## 1. Does SNS guarantee exactly-once delivery to subscribers?

**No.**  
Amazon **SNS (Simple Notification Service)** does **not guarantee exactly-once delivery**.  
It provides **“at least once” delivery**.

### Explanation:
- SNS makes a *best-effort* attempt to deliver every message to all subscribed endpoints.
- In some cases (e.g., network retries, endpoint timeouts, internal retries), a subscriber may receive **duplicate messages**.
- Therefore, subscribers should design their systems to be **idempotent** — meaning handling the same message more than once will not cause unintended side effects.

✅ **Guarantee:** At-least-once delivery  
❌ **Not guaranteed:** Exactly-once delivery or ordered delivery (except FIFO topics)

---

## 2. What is the purpose of the Dead-letter Queue (DLQ)?

A **Dead-letter Queue (DLQ)** is a **backup queue** that stores messages that could not be successfully processed or delivered after a certain number of retries.

### Purpose:
- To **capture failed messages** for later analysis or reprocessing.
- To **avoid losing data** when a target system or consumer fails.
- To **debug issues** — for example, message format errors or endpoint timeouts.

### Where it’s used:
- **SQS**: When a consumer fails to process a message successfully after the maximum receive attempts.
- **SNS**: When an SNS subscription endpoint (like an SQS queue or Lambda function) cannot process the message.
- **EventBridge**: When the target (e.g., Lambda, API, etc.) fails repeatedly.

---

## 3. How to enable a notification to your email when messages are added to the DLQ?

You can set up **email notifications** for DLQ messages by integrating **SNS with CloudWatch**:

### Steps:

1. **Create an SNS Topic**
   - Go to **Amazon SNS** → **Create topic** (type: Standard).  
   - Give it a name, e.g., `DLQ-Alerts-Topic`.

2. **Subscribe Your Email**
   - Create a subscription for the topic with **Protocol = Email**.  
   - Enter your email address and confirm via the email AWS sends you.

3. **Create a CloudWatch Alarm on the DLQ**
   - Go to **CloudWatch → Alarms → Create alarm**.  
   - Select **SQS Metrics → ApproximateNumberOfMessagesVisible** for your DLQ.  
   - Set the condition (e.g., “≥ 1 message for 1 datapoint”).  
   - For the **Alarm action**, choose “Send a notification to an SNS topic”.  
   - Select your `DLQ-Alerts-Topic`.

4. **Result:**  
   When a message appears in the DLQ, CloudWatch triggers the alarm → SNS sends an **email notification**.

---

### ✅ **Summary**

| Question | Answer |
|-----------|---------|
| Does SNS guarantee exactly-once delivery? | ❌ No — SNS guarantees **at-least-once** delivery (duplicates possible). |
| Purpose of DLQ | To capture **failed or undeliverable messages** for analysis or reprocessing. |
| How to notify via email when DLQ gets messages | Set up a **CloudWatch alarm** on the DLQ metric → **Trigger SNS topic** → **Send email alert**. |

---
