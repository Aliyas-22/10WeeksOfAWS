# Day 17 — Messaging & Streaming on AWS

**Project:** CloudAdhar AWS Zero To Hero — Week 9
**Region:** ap-south-1 (Mumbai)
**Services covered:** Amazon SQS, Amazon SNS, Amazon EventBridge, EventBridge Scheduler, Amazon Kinesis Data Streams, Amazon Data Firehose, Amazon S3

This lab builds a complete order-processing and clickstream pipeline using AWS's core messaging and streaming services — entirely through the AWS Management Console, with AWS CloudShell used only to produce test records for Kinesis.

---

## Architecture

![Architecture Diagram](./Daigram/Day-17.drawio.png)

```text
SQS Standard queue -> visibility timeout -> DLQ -> redrive
SQS FIFO queue -> message group -> deduplication

SNS topic
  +-> Standard orders queue: receives every message
  +-> Priority orders queue: receives only priority=HIGH

Custom application event
  -> EventBridge custom bus
  -> amount > 5000 rule
  -> Priority SQS queue

One-time schedule
  -> EventBridge Scheduler
  -> Payment reminder in SQS

Clickstream producer
  -> Kinesis Data Streams
  -> Amazon Data Firehose
  -> Private S3 bucket
```

---

## Part A — Amazon SQS

### 1. Create and configure the Standard queue

A Standard queue was created for order messages, with a Dead-Letter Queue (DLQ) attached after 3 failed receive attempts.

![Message sent to Standard queue](./Screenshots/sended-msg-to-queue-standerd.png)

![Standard orders queue overview](./Screenshots/orders-standerd.png)

### 2. Test visibility timeout — poll without deleting

The same message was polled repeatedly without deleting it, letting the receive count climb past the configured maximum of 3.

![Polling shows receive count building up](./Screenshots/poll-for-three-msg.png)

### 3. Message moves to the Dead-Letter Queue

After exceeding the maximum receive count, SQS automatically moved the message to the DLQ.

![Message received in DLQ](./Screenshots/dlq-queue-0-recived.png)

### 4. Redrive the message back to the source queue

Using **Start DLQ redrive → Redrive to source queue(s)**, the failed message was moved back to the Standard queue.

![DLQ redrive successful](./Screenshots/seuccesffulyy-redrive.png)

### 5. Purge queue before the next test

Queues were purged between tests to keep results predictable.

![Purge queue action](./Screenshots/purge-queue.png)

### 6. FIFO queue — ordering within a message group

Two messages (`Payment received`, `Order shipped`) were sent to the same message group (`order-O-2001`) to prove FIFO ordering is preserved per group.

![Sending FIFO messages](./Screenshots/fifo-message-send.png)

![First FIFO message polled — Payment received](./Screenshots/payment-recieved-msg.png)

**Result:** the first message in the group was always returned before the second, confirming order is guaranteed within a message group.

---

## Part B — Amazon SNS

### 7. Create the SNS topic

A Standard SNS topic was created to fan out order events to multiple subscriber queues.

![SNS topic created](./Screenshots/sns-topic.png)

### 8. Subscribe both SQS queues to the topic

The Standard queue and the Priority queue were both subscribed to the topic (raw message delivery disabled).

![SNS subscriptions](./Screenshots/sns-subcription.png)

![Topic-to-queue mapping](./Screenshots/topic-queue.png)

### 9. Add a HIGH-priority filter policy

The Priority queue's subscription was given a filter policy: `{"priority": ["HIGH"]}`. The Standard queue subscription was left unfiltered so it receives every message.

### 10. Test — NORMAL order (goes to Standard queue only)

![SNS publish — NORMAL priority test](./Screenshots/sns-string-normal.png)

![Message delivered to Standard queue via SNS](./Screenshots/thrugh-sns-standerd-msg.png)

### 11. Test — HIGH order (goes to Standard + Priority queue)

![SNS publish — HIGH priority test](./Screenshots/sns-string-value-high.png)

![Priority queue receives filtered HIGH message](./Screenshots/recieve-msg-priority-high.png)

**Result:** the filter policy correctly routed only `priority=HIGH` messages to the Priority queue, while every message reached the Standard queue.

---

## Part C — Amazon EventBridge

### 12. Custom event bus and high-value order rule

A custom event bus was created, along with a rule matching events where `detail.amount > 5000`, targeting the Priority SQS queue.

![EventBridge rule and event pattern](./Screenshots/event-bridge.png)

### 13. Send test events

A negative test (`amount: 2500`) was sent and correctly produced no message in the Priority queue. A positive test (`amount: 7500`) matched the rule and delivered the full event envelope to the Priority queue.

![Event pattern match verification](./Screenshots/recursie-readable.png)

**Result:** only events above the ₹5000 threshold reached the queue, confirming the rule's numeric filter worked as expected.

---

## Part D — EventBridge Scheduler

### 14. Create a one-time payment reminder schedule

A one-time schedule was created to send a payment reminder payload directly to the Priority SQS queue, 5–10 minutes in the future.

![Schedule created confirmation](./Screenshots/scheduled-created.png)

![One-time schedule pattern and trigger time](./Screenshots/schedulled-pattern-time.png)

### 15. Verify the reminder fires on schedule

At the scheduled time, the payload appeared in the Priority queue and the one-time schedule automatically deleted itself.

![Schedule trigger delivering payload to queue](./Screenshots/schedule-trigger.png)

---

## Part E — Kinesis Data Streams → Firehose → S3

### 16. Produce clickstream records into Kinesis

Test records were sent from AWS CloudShell using the same partition key, proving records with the same key land on the same shard.

![Kinesis put-record from CloudShell](./Screenshots/amazon-kinesis-put-record.png)

![Kinesis stream active and receiving records](./Screenshots/streams-record.png)

### 17. View records in the Kinesis Data Viewer

![Data viewer showing streamed records](./Screenshots/stream-data.png)

### 18. Firehose delivers records to S3

Amazon Data Firehose was configured with Kinesis as its source and a private S3 bucket as its destination (SSE-S3 encrypted, uncompressed, no newline delimiter).

![Delivered record inside S3 bucket](./Screenshots/bucket-record.png)

**Result:** clickstream JSON events flowed end-to-end from Kinesis → Firehose → S3 and were verified by inspecting the delivered object directly.

---

## Part F — Amazon MQ & Amazon MSK (Selection Only)

Both services were explored at the console level only — no broker or cluster was created.

- **Amazon MQ** — best fit when migrating an existing app that already depends on RabbitMQ or ActiveMQ protocols.
- **Amazon MSK** — best fit when an application needs native Apache Kafka APIs, consumer groups, and offset management.

---

## Result Summary

| Test | Expected Result | Outcome |
|---|---|---|
| Standard SQS receive without deletion | Message becomes visible again | ✅ |
| DLQ test | Repeated failure moves message to DLQ | ✅ |
| DLQ redrive | Message returns to source queue | ✅ |
| FIFO test | Same-group messages retain order | ✅ |
| SNS NORMAL | Standard queue only | ✅ |
| SNS HIGH | Standard and Priority queues | ✅ |
| EventBridge amount 2500 | No priority message | ✅ |
| EventBridge amount 7500 | One priority message | ✅ |
| Scheduler | Payment reminder appears at selected time | ✅ |
| Kinesis | Same partition key reaches same shard | ✅ |
| Firehose | New Kinesis records appear in S3 | ✅ |

---

## Cleanup

All resources were deleted after the lab in this order: Firehose stream → Kinesis stream → S3 bucket (emptied, then deleted) → EventBridge rule → custom event bus → (Scheduler auto-deleted) → SNS subscriptions & topic → all four SQS queues → unused IAM execution roles.

---

## References

- [Amazon SQS Developer Guide](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
- [SQS Dead-Letter Queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
- [SNS Subscription Filtering](https://docs.aws.amazon.com/sns/latest/dg/sns-message-filtering.html)
- [EventBridge Event Patterns](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-event-patterns.html)
- [EventBridge Scheduler](https://docs.aws.amazon.com/scheduler/latest/UserGuide/what-is-scheduler.html)
- [Amazon Data Firehose](https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html)
