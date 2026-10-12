# Chapter ০৭: Message Queue ও Event-Driven Architecture

## ভাগ ১: Message Queue

### ১. Message Queue কী?

দুটি অংশের মধ্যে **মাঝখানে একটি মধ্যস্থতাকারী (broker)** থাকে। Sender বার্তা (message) queue-তে রাখে, Receiver নিজের সুবিধামতো সময়ে তুলে নিয়ে কাজ করে।

> উদাহরণ: ডাকঘরের চিঠির বাক্স। আপনি চিঠি ফেলে চলে যান; ডাকপিয়ন পরে নিয়ে যায়। দুজনের একসাথে উপস্থিত থাকতে হয় না।

```
Producer ──▶ [ Queue: m1 m2 m3 ] ──▶ Consumer
```

### ২. কেন ব্যবহার করি?

| সুবিধা | ব্যাখ্যা |
|---|---|
| **Async processing** | ভারী কাজ (email, report) background-এ; user-কে অপেক্ষা করতে হয় না |
| **Decoupling** | Producer জানে না consumer কে বা কোথায় |
| **Load leveling** | হঠাৎ ভিড়ে system পড়ে না; queue জমা রাখে |
| **Reliability** | Consumer down থাকলেও message হারায় না |
| **Scaling** | Consumer-এর সংখ্যা বাড়িয়ে দ্রুত process |

### ৩. মূল ধারণা

- **Producer** — message পাঠায়
- **Consumer** — message নেয় ও process করে
- **Broker** — RabbitMQ, Kafka, Azure Service Bus, AWS SQS
- **Queue (Point-to-Point)** — একটি message **একজন** consumer পায়
- **Topic (Pub/Sub)** — একটি message **অনেক** subscriber পায়
- **Acknowledgement (Ack)** — consumer সফলভাবে শেষ করলে জানায়; না জানালে message আবার আসে
- **Dead Letter Queue (DLQ)** — বারবার ব্যর্থ message আলাদা queue-তে, পরে পরীক্ষার জন্য

### ৪. Delivery Guarantee

| ধরন | মানে | ঝুঁকি |
|---|---|---|
| At-most-once | সর্বোচ্চ একবার | message হারাতে পারে |
| **At-least-once** ⭐ | অন্তত একবার | **duplicate** আসতে পারে |
| Exactly-once | ঠিক একবার | কঠিন ও ব্যয়বহুল |

বাস্তবে সবচেয়ে প্রচলিত **at-least-once** + **Idempotent Consumer**।

### ৫. কোড উদাহরণ (MassTransit + RabbitMQ)

```csharp
// Message contract
public record OrderPlaced(int OrderId, int CustomerId, decimal Total);

// Producer — order save-এর পর publish
await _publishEndpoint.Publish(new OrderPlaced(order.Id, order.CustomerId, order.Total));

// Consumer
public class SendOrderEmailConsumer : IConsumer<OrderPlaced>
{
    public async Task Consume(ConsumeContext<OrderPlaced> context)
    {
        var msg = context.Message;
        // Idempotency: আগেই পাঠানো হয়ে থাকলে skip
        await _emailService.SendOrderConfirmationAsync(msg.OrderId);
    }
}

// Program.cs
builder.Services.AddMassTransit(x =>
{
    x.AddConsumer<SendOrderEmailConsumer>();
    x.UsingRabbitMq((ctx, cfg) =>
    {
        cfg.Host("localhost");
        cfg.ConfigureEndpoints(ctx);
    });
});
```

### ৬. কোন পরিস্থিতিতে?

- Order হলে email/SMS পাঠানো
- Report/PDF তৈরি, ছবি process
- Payment-এর পর inventory update (service-to-service)
- Traffic spike সামলানো

### ৭. RabbitMQ বনাম Kafka (সংক্ষেপে)

| | RabbitMQ | Kafka |
|---|---|---|
| ধরন | Smart broker, task queue | Distributed log / event stream |
| Message পড়ার পর | সাধারণত মুছে যায় | রেখে দেওয়া হয় (retention) |
| উপযোগী | Task/job, routing | বিশাল volume, event streaming, replay |

---

## ভাগ ২: Event-Driven Architecture (EDA)

### ১. EDA কী?

Service গুলো একে অপরকে সরাসরি "এটা করো" বলে না। বরং কিছু **ঘটলে (event)** সেটা ঘোষণা করে; আগ্রহী service গুলো শুনে নিজের কাজ করে।

- **Command** = "এটা করো" (`SendEmail`) — নির্দিষ্ট প্রাপক
- **Event** = "এটা ঘটেছে" (`OrderPlaced`) — কে শুনছে প্রেরক জানে না

```
                      ┌─▶ [Inventory]  stock কমাও
[Order Service] ─OrderPlaced─▶ Broker ─┼─▶ [Billing]    invoice বানাও
                      └─▶ [Email]      confirmation পাঠাও
```

### ২. সুবিধা

- নতুন service যোগ করা সহজ — Order Service বদলাতে হয় না
- Loose coupling, আলাদাভাবে scale
- Real-time প্রতিক্রিয়া, audit trail

### ৩. চ্যালেঞ্জ

| সমস্যা | সমাধান |
|---|---|
| Flow বোঝা কঠিন (কে কী শুনছে?) | Event catalog, distributed tracing |
| Eventual consistency | UI-তে সেভাবে design করা |
| Duplicate/ক্রমহীন event | Idempotent consumer, version/sequence |
| Debugging কঠিন | Correlation ID, ভালো logging |

### ৪. Outbox Pattern — Professional-grade অপরিহার্য ⭐

**সমস্যা:** DB-তে order save হলো, কিন্তু broker-এ publish করার আগেই app crash — event হারিয়ে গেল (বা উল্টোটা)।

**সমাধান:** Event-টি **একই DB transaction-এ** `Outbox` table-এ সংরক্ষণ করুন (ACID!)। আলাদা একটি background worker table থেকে broker-এ পাঠিয়ে `Processed` চিহ্নিত করে।

```csharp
await using var tx = await _db.Database.BeginTransactionAsync();
_db.Orders.Add(order);
_db.OutboxMessages.Add(new OutboxMessage
{
    Type = nameof(OrderPlaced),
    Payload = JsonSerializer.Serialize(new OrderPlaced(order.Id, order.CustomerId, order.Total)),
    CreatedAt = DateTime.UtcNow
});
await _db.SaveChangesAsync();
await tx.CommitAsync();   // Order + Event একসাথে বা কিছুই না
```
(MassTransit-এ Outbox built-in আছে।)

### ৫. Event-এর ধরন

- **Event Notification** — ছোট ("Order ১০ হয়েছে"), বিস্তারিত দরকার হলে API call
- **Event-Carried State Transfer** — event-এই প্রয়োজনীয় সব data
- **Event Sourcing** — state নয়, **ঘটনার ক্রম** সংরক্ষণ (উন্নত বিষয়; CQRS-এর সাথে চলে)

### ৬. Professional Practices

- Event-এর নাম **অতীতকালে** (`OrderPlaced`)
- Event schema **versioning** রাখুন (পুরনো consumer যেন না ভাঙে)
- সব consumer **idempotent**
- **DLQ** ও retry policy রাখুন
- Queue-র গভীরতা ও lag **monitor** করুন
- Event-এ sensitive data (পাসওয়ার্ড) রাখবেন না

---

## ✅ সারসংক্ষেপ

- Queue = async ও decoupled যোগাযোগ; **at-least-once + idempotency**
- EDA = "কিছু ঘটেছে" ঘোষণা, আগ্রহীরা শোনে
- Outbox = DB save ও event publish-কে নিরাপদে এক করে

⬅️ [Monolith vs Microservices](06-monolith-vs-microservices.md) | ➡️ [CQRS](08-cqrs.md)
