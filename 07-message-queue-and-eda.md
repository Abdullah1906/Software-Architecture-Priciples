# Chapter ০৭: Message Queue ও Event-Driven Architecture

## 1. Introduction

আধুনিক software system-এ একাধিক service-এর মধ্যে communication, background processing, scalability এবং reliability নিশ্চিত করতে **Message Queue** ও **Event-Driven Architecture (EDA)** গুরুত্বপূর্ণ ভূমিকা রাখে।

এই গাইডে আমরা শিখব:

- Message Queue কী এবং কেন ব্যবহার করা হয়
- Event-Driven Architecture কীভাবে কাজ করে
- Synchronous ও Asynchronous communication-এর পার্থক্য
- RabbitMQ কী এবং কীভাবে কাজ করে
- ASP.NET Core Web API-তে বাস্তব ব্যবহার
- Production-grade architecture ও best practices

**Technology Stack:**
- ASP.NET Core Web API (.NET 8/9)
- RabbitMQ
- Docker Compose
- SQL Server
- Background Worker

---

## 2. Message Queue কী?

Message Queue হলো এমন একটি communication mechanism, যার মাধ্যমে একটি application অন্য application বা service-এর কাছে message পাঠাতে পারে, আর receiver সেটি পরে process করতে পারে।

সহজ উদাহরণ:

ধরুন, আপনার একটি Bus Booking System আছে।

একজন customer ticket book করলেন। Booking সফল হওয়ার পর তাঁকে email পাঠাতে হবে।

যদি booking API নিজেই email পাঠানোর জন্য অপেক্ষা করে, তাহলে response দিতে বেশি সময় লাগতে পারে।

কিন্তু Message Queue ব্যবহার করলে:

1. Booking API booking সংরক্ষণ করবে।
2. Email পাঠানোর জন্য একটি message queue-তে message পাঠাবে।
3. API customer-কে দ্রুত response দেবে।
4. Background Worker queue থেকে message নিয়ে email পাঠাবে।

অর্থাৎ, booking সম্পন্ন করা এবং email পাঠানো আলাদা কাজ হিসেবে পরিচালিত হবে।

### Architecture

```text
Customer
   |
   v
Booking API
   |
   +------> SQL Server
   |         Booking Saved
   |
   +------> RabbitMQ Queue
                 |
                 v
           Email Worker
                 |
                 v
             Email Sent
```

**মূল ধারণা:** Producer message পাঠায়, Queue message ধরে রাখে এবং Consumer message process করে।

---

## 3. Message Queue কেন ব্যবহার করা হয়?

### 3.1 Decoupling

একটি service-কে অন্য service-এর অভ্যন্তরীণ implementation সম্পর্কে জানতে হয় না।

উদাহরণ: Booking API শুধু email notification-এর message প্রকাশ করবে। Email কীভাবে পাঠানো হবে, সেটি Email Worker পরিচালনা করবে।

### 3.2 Asynchronous Processing

API-কে প্রতিটি background কাজ শেষ হওয়া পর্যন্ত অপেক্ষা করতে হয় না।

উদাহরণ:
- Email notification
- SMS notification
- PDF invoice generation
- Report generation
- Image processing

### 3.3 Scalability

একই ধরনের কাজ একাধিক consumer দিয়ে process করা যায়।

```text
             RabbitMQ Queue
              /     |     \
             v      v      v
          Worker1 Worker2 Worker3
```

কাজের চাপ বেড়ে গেলে consumer instance বাড়ানো যায়। তবে throughput নির্ভর করবে database, queue configuration, message size এবং consumer capacity-এর ওপর।

### 3.4 Reliability

Consumer সাময়িকভাবে বন্ধ থাকলেও durable queue ও persistent message configuration সঠিকভাবে ব্যবহার করা হলে message সংরক্ষিত থাকতে পারে।

তবে queue, broker, database এবং deployment-এর সঠিক durability configuration ছাড়া স্বয়ংক্রিয়ভাবে সব message নিরাপদ থাকবে—এমন নিশ্চয়তা নেই।

### 3.5 Load Buffering

হঠাৎ অনেক request এলে queue কাজগুলো সাময়িকভাবে জমা রাখতে পারে। Consumer ধীরে ধীরে সেগুলো process করবে।

---

## 4. Event-Driven Architecture (EDA) কী?

**Event-Driven Architecture** হলো এমন একটি software architecture যেখানে system-এর কোনো গুরুত্বপূর্ণ ঘটনা (Event) ঘটলে তার notification প্রকাশ করা হয় এবং অন্য component সেই event-এর ভিত্তিতে কাজ করে।

উদাহরণ:

```text
Booking Confirmed
        |
        v
BookingConfirmed Event
        |
        v
   Event Broker
     /       \
    v         v
Email       Reporting
Service      Service
```

Booking service জানে না কোন কোন service event-টি consume করবে।

Email service email পাঠাতে পারে, আর Reporting service booking statistics update করতে পারে।

দুটি service-ই একই event থেকে নিজস্ব কাজ সম্পন্ন করে।

### Event-এর উদাহরণ

- `BookingCreated`
- `BookingConfirmed`
- `PaymentCompleted`
- `OrderShipped`
- `UserRegistered`
- `InvoiceGenerated`

Event সাধারণত অতীতে ঘটে যাওয়া কোনো ঘটনার তথ্য প্রকাশ করে।

উদাহরণ: `PaymentCompleted` অর্থ payment ইতোমধ্যে সম্পন্ন হয়েছে।

---

## 5. Message Queue বনাম Event-Driven Architecture

দুটি ধারণা সম্পর্কিত, কিন্তু এক নয়।

| বিষয় | Message Queue | Event-Driven Architecture |
|---|---|---|
| মূল উদ্দেশ্য | Message আদান-প্রদান ও কাজের queue পরিচালনা | Event-এর মাধ্যমে component-গুলোর মধ্যে communication |
| Scope | Communication mechanism | সম্পূর্ণ architectural pattern |
| Communication | Command, task বা event বহন করতে পারে | সাধারণত event publication ও consumption |
| Consumer | Queue থেকে message গ্রহণ করে | Event broker, queue বা অন্য transport ব্যবহার করতে পারে |
| Example | Email processing queue | Booking event থেকে email ও reporting update |

**মনে রাখুন:** RabbitMQ একটি messaging broker। এটি Message Queue এবং Event-Driven Architecture—দুই ক্ষেত্রেই ব্যবহার করা যায়।

EDA তৈরি করতে RabbitMQ বাধ্যতামূলক নয়। Kafka, cloud messaging service বা অন্য event transport-ও ব্যবহার করা যায়।

---

## 6. Synchronous বনাম Asynchronous Communication

### Synchronous

```text
Client
  |
  v
Booking API
  |
  v
Email Service
  |
  v
Response to Client
```

এখানে API email service-এর response-এর জন্য অপেক্ষা করতে পারে।

সমস্যা:
- Email service ধীর হলে API-ও ধীর হতে পারে।
- Email service unavailable হলে booking request ব্যর্থ হতে পারে, যদি dependency-টি বাধ্যতামূলক হয়।

### Asynchronous

```text
Client
  |
  v
Booking API
  |
  +------> Database
  |
  +------> Message Broker
                |
                v
           Email Worker

API ------> Response to Client
```

এখানে email background-এ process হয়।

**গুরুত্বপূর্ণ:** Asynchronous processing মানে কাজটি অবশ্যই সফল হবে এমন নয়। Consumer failure, message rejection বা broker outage-এর জন্য error handling প্রয়োজন।

---

## 7. RabbitMQ কী?

RabbitMQ হলো একটি message broker। এটি producer-এর কাছ থেকে message গ্রহণ করে exchange ও routing rules অনুযায়ী queue-তে পাঠায়, তারপর consumer-দের message গ্রহণ করার সুযোগ দেয়।

### RabbitMQ-এর প্রধান Components

**1. Producer**

যে application message প্রকাশ করে।

উদাহরণ: Booking API।

**2. Exchange**

Producer-এর message গ্রহণ করে routing rules অনুযায়ী এক বা একাধিক queue-তে পাঠায়।

**3. Queue**

Consumer-এর process করার জন্য message ধরে রাখে।

**4. Binding**

Exchange এবং queue-এর মধ্যে routing সম্পর্ক নির্ধারণ করে।

**5. Consumer**

Queue থেকে message নিয়ে process করে।

**6. Acknowledgment (ACK)**

Consumer সফলভাবে message process করার পর broker-কে জানায় যে message-টি acknowledge করা হয়েছে।

### RabbitMQ Message Flow

```text
Booking API (Producer)
          |
          v
       Exchange
          |
      Routing Key
          |
          v
      Email Queue
          |
          v
     Email Consumer
          |
     Process Message
          |
          v
         ACK
```

**নোট:** Queue-তে message পাঠানোর জন্য exchange ও routing configuration ব্যবহার করা হয়। Default exchange-এর মাধ্যমে সরাসরি queue-তেও message route করা সম্ভব।

---

## 8. বাস্তব উদাহরণ: Bus Booking System

ধরা যাক, আমাদের system-এ নিম্নলিখিত কাজগুলো আছে:

- Booking সংরক্ষণ
- Email notification
- Ticket PDF তৈরি
- Reporting update

প্রথমে Booking API booking সংরক্ষণ করবে। তারপর `BookingConfirmed` event প্রকাশ করবে।

```text
                   Booking API
                       |
                       v
                    SQL Server
                       |
                       v
                 Event Publisher
                       |
                       v
                    RabbitMQ
                  /     |      \
                 v      v       v
              Email   Ticket   Reporting
              Worker  Worker    Worker
```

### ধাপে ধাপে workflow

1. Customer booking request পাঠায়।
2. API seat availability ও booking rules যাচাই করে।
3. Database transaction-এ booking সংরক্ষণ করা হয়।
4. Booking confirmation event প্রকাশ করা হয়।
5. Email worker confirmation email পাঠায়।
6. Ticket worker PDF ticket তৈরি করে।
7. Reporting worker booking statistics update করে।

### একটি গুরুত্বপূর্ণ Production Design সিদ্ধান্ত

Database-এ booking save হওয়ার পর broker-এ event পাঠানোর আগেই API crash করতে পারে।

ফলে booking database-এ থাকবে, কিন্তু event প্রকাশিত হবে না।

এই সমস্যার একটি সমাধান হলো **Transactional Outbox Pattern**।

```text
Single SQL Transaction
   |
   +----> Save Booking
   |
   +----> Save Outbox Event
                |
                v
          Outbox Publisher
                |
                v
             RabbitMQ
```

Booking এবং Outbox event একই database transaction-এ সংরক্ষিত হয়। একটি background publisher pending event broker-এ পাঠায়।

এই পদ্ধতি database update এবং event publication-এর মধ্যকার failure window কমায়। তবে publisher retry করলে duplicate event প্রকাশিত হতে পারে, তাই consumer idempotent হওয়া প্রয়োজন।

---

## 9. .NET-এ Message Queue ব্যবহারের ধারণা

ASP.NET Core-এ সাধারণত দুটি পৃথক application component রাখা যায়:

- **Booking API:** HTTP request গ্রহণ ও business logic পরিচালনা করে।
- **Email Worker:** RabbitMQ থেকে message গ্রহণ করে background processing করে।

### Event DTO

```csharp
public sealed record BookingConfirmedEvent(
    Guid EventId,
    Guid BookingId,
    string CustomerEmail,
    DateTime OccurredAtUtc
);
```

এটি event-এর একটি সাধারণ data contract।

Production-এ event contract versioning, validation এবং backward compatibility নিশ্চিত করা উচিত।

### Producer-এর ধারণাগত উদাহরণ

```csharp
public async Task PublishAsync(
    BookingConfirmedEvent message,
    CancellationToken cancellationToken)
{
    var body = JsonSerializer.SerializeToUtf8Bytes(message);

    var properties = new BasicProperties
    {
        ContentType = "application/json",
        DeliveryMode = DeliveryModes.Persistent,
        MessageId = message.EventId.ToString()
    };

    await channel.BasicPublishAsync(
        exchange: "booking.events",
        routingKey: "booking.confirmed",
        mandatory: true,
        basicProperties: properties,
        body: body,
        cancellationToken: cancellationToken
    );
}
```

এটি RabbitMQ.Client-এর API version অনুযায়ী সামঞ্জস্য করতে হবে। Exchange ও channel আগে declare/configure করা থাকতে হবে।

`mandatory: true` unroutable message সম্পর্কে publisher-কে return notification পাওয়ার সুযোগ দেয়। এটি একাই broker-এ message durableভাবে সংরক্ষিত হওয়ার পূর্ণ নিশ্চয়তা নয়।

Production publisher-এ publisher confirms, return handling, connection recovery এবং retry strategy-ও দরকার।

### Consumer-এর ধারণাগত উদাহরণ

```csharp
public async Task ProcessAsync(
    BookingConfirmedEvent message,
    CancellationToken cancellationToken)
{
    // Validate event
    // Check duplicate processing
    // Send confirmation email
    // Persist processing status
}
```

Consumer-এর processing সফল হলে message ACK করতে হবে।

Processing ব্যর্থ হলে policy অনুযায়ী retry বা Dead-Letter Queue (DLQ)-তে পাঠাতে হবে।

**গুরুত্বপূর্ণ:** Production consumer-এ manual acknowledgment ব্যবহার করুন এবং business processing সফল হওয়ার পর ACK দিন। ACK-এর আগে consumer crash করলে message আবার deliver হতে পারে; তাই duplicate handling আবশ্যক।

---

## 10. Production-Grade Architecture

একটি production system-এ শুধু queue তৈরি করাই যথেষ্ট নয়।

### Recommended Architecture

```text
                    Client
                       |
                       v
                 ASP.NET Core API
                       |
                       v
                   SQL Server
                  /          \
                 v            v
             Business       Outbox
              Data          Events
                              |
                              v
                       Outbox Publisher
                              |
                              v
                         RabbitMQ
                              |
                   +----------+----------+
                   |          |          |
                   v          v          v
                Email       Ticket    Reporting
                Worker      Worker     Worker
                   |          |          |
                   v          v          v
                Provider   File Store   Read Model
```

### গুরুত্বপূর্ণ Best Practices

**1. Durable Queue**

Broker restart-এর পর queue definition টিকে থাকার জন্য durable queue ব্যবহার করুন।

**2. Persistent Messages**

প্রয়োজন অনুযায়ী persistent delivery mode ব্যবহার করুন। Durable queue ও persistent message একসঙ্গে ব্যবহার করাও disk durability-এর সম্পূর্ণ নিশ্চয়তা নয়; broker confirmation এবং deployment/storage configuration গুরুত্বপূর্ণ।

**3. Publisher Confirms**

Broker message গ্রহণ করেছে কি না, তা নিশ্চিত করার জন্য publisher confirms ব্যবহার করুন।

**4. Manual ACK**

Business processing সফল হওয়ার পর message acknowledge করুন।

**5. Retry Policy**

Transient error হলে সীমিত সংখ্যক retry এবং exponential backoff ব্যবহার করুন।

**6. Dead-Letter Queue (DLQ)**

বারবার ব্যর্থ হওয়া message আলাদা queue-তে পাঠান, যাতে সেটি মূল queue-তে আটকে না থাকে।

**7. Idempotent Consumer**

একই event একাধিকবার এলে যেন duplicate email, duplicate payment বা duplicate booking না হয়।

**8. Transactional Outbox**

Database commit ও event publication-এর মধ্যকার consistency সমস্যা সমাধানে ব্যবহার করুন।

**9. Observability**

Queue depth, consumer count, processing latency, retry rate, dead-letter count এবং broker health monitor করুন।

**10. Security**

- RabbitMQ-এর default guest account production-এ ব্যবহার করবেন না।
- Least-privilege user ও virtual host ব্যবহার করুন।
- Network access সীমিত করুন।
- TLS ব্যবহার করুন যেখানে উপযুক্ত।
- Credentials secret manager বা secure environment configuration-এ রাখুন।

---

## 11. Retry, Duplicate ও Dead-Letter Handling

ধরা যাক, Email Worker message গ্রহণ করেছে কিন্তু email provider সাময়িকভাবে unavailable।

```text
RabbitMQ
   |
   v
Email Worker
   |
   v
Email Provider Error
   |
   v
Retry with Backoff
   |
   +----> Success ----> ACK
   |
   +----> Retry Limit Exceeded
                  |
                  v
                 DLQ
```

Retry সীমাহীন হওয়া উচিত নয়। নইলে poison message বারবার process হয়ে system-এর resource নষ্ট করতে পারে।

DLQ-তে যাওয়া message-এর জন্য alert, investigation এবং controlled replay process থাকা উচিত।

একই message পুনরায় process হতে পারে—এ কারণে সাধারণ queue-based system-এ **at-least-once delivery** ধরে নিয়ে design করা নিরাপদ।

---

## 12. কখন Message Queue ব্যবহার করবেন?

| Use Case | Queue উপযোগী? | কারণ |
|---|---|---|
| Email/SMS notification | হ্যাঁ | Background processing |
| PDF invoice generation | হ্যাঁ | সময়সাপেক্ষ কাজ |
| Image/video processing | হ্যাঁ | CPU-intensive processing |
| Order/booking event distribution | হ্যাঁ | একাধিক consumer-কে notify করা |
| Real-time balance validation | সাধারণত সরাসরি request | তাৎক্ষণিক ফলাফল প্রয়োজন |
| Critical payment confirmation | Hybrid approach | Synchronous business validation ও reliable asynchronous event processing |

সব communication-এ queue ব্যবহার করার প্রয়োজন নেই। যেসব কাজের ফলাফল সঙ্গে সঙ্গে client-কে জানাতে হয়, সেখানে synchronous API উপযুক্ত হতে পারে।

---

## 13. Message Queue বনাম Event Streaming

RabbitMQ ও Apache Kafka-কে সব ক্ষেত্রে একই ধরনের tool হিসেবে দেখা উচিত নয়।

| বিষয় | RabbitMQ | Apache Kafka |
|---|---|---|
| সাধারণ ব্যবহার | Task queue ও messaging | Event streaming ও durable event log |
| Message handling | Consumer acknowledgment ও queue semantics | Partition ও offset-based consumption |
| Replay | সাধারণ queue workflow-তে আলাদা ব্যবস্থা লাগতে পারে | Retention-এর মধ্যে event পুনরায় পড়া যায় |
| Typical use case | Email, background jobs, routing | Analytics, event pipelines, large-scale streaming |

এটি একটি সাধারণ তুলনা। উভয় প্রযুক্তির capabilities configuration ও deployment অনুযায়ী পরিবর্তিত হতে পারে।

---

## 14. শেখার Roadmap

Professional দক্ষতা অর্জনের জন্য নিচের ধাপগুলো অনুসরণ করুন:

1. Producer, Consumer, Queue ও Exchange বুঝুন।
2. Docker Compose দিয়ে RabbitMQ চালান।
3. ASP.NET Core Web API থেকে message publish করুন।
4. .NET Worker Service দিয়ে message consume করুন।
5. Manual ACK ও retry যোগ করুন।
6. Dead-Letter Exchange/Queue configure করুন।
7. Idempotency ও duplicate handling implement করুন।
8. SQL Server Transactional Outbox implement করুন।
9. OpenTelemetry, structured logging ও metrics যোগ করুন।
10. Load testing, failure testing ও production deployment অনুশীলন করুন।

---

## 15. Final Summary

- **Message Queue** হলো asynchronous message delivery ও processing-এর একটি mechanism।
- **Event-Driven Architecture** হলো event-এর মাধ্যমে loosely coupled component তৈরি করার architectural pattern।
- **RabbitMQ** message routing ও queue management-এর জন্য ব্যবহৃত একটি broker।
- **Producer** message প্রকাশ করে, **Consumer** message process করে।
- **Transactional Outbox**, publisher confirms, retry, DLQ, idempotency ও monitoring production reliability বাড়াতে গুরুত্বপূর্ণ।
- Queue ব্যবহার করলেই microservices architecture তৈরি হয় না; একটি monolithic application-ও background worker ও queue ব্যবহার করতে পারে।


## ✅ সারসংক্ষেপ

- Queue = async ও decoupled যোগাযোগ; **at-least-once + idempotency**
- EDA = "কিছু ঘটেছে" ঘোষণা, আগ্রহীরা শোনে
- Outbox = DB save ও event publish-কে নিরাপদে এক করে

⬅️ [Monolith vs Microservices](06-monolith-vs-microservices.md) | ➡️ [CQRS](08-cqrs.md)
