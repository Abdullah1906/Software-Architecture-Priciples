# Chapter ০৬: Monolithic বনাম Microservices

## ১. Monolithic কী?

পুরো application **একটি একক deployable unit** — UI, business logic, data access সব একই codebase ও একই process-এ।

```
┌──────────────────────────────┐
│   একটি Application           │
│  ┌──────┬──────┬──────────┐  │
│  │ User │Order │ Payment  │  │
│  └──────┴──────┴──────────┘  │
│        একটি Database         │
└──────────────────────────────┘
```

## ২. Microservices কী?

Application ছোট ছোট **স্বাধীন service**-এ ভাগ করা। প্রতিটি service নিজের কাজ করে, নিজের database রাখে, আলাদাভাবে deploy হয়।

```
 Client → API Gateway ─┬→ [User Service]    → DB১
                       ├→ [Order Service]   → DB২
                       └→ [Payment Service] → DB৩
         (Service-গুলো HTTP/gRPC/Message Queue-তে কথা বলে)
```

---

## ৩. তুলনা

| বিষয় | Monolith | Microservices |
|---|---|---|
| শুরু করা | ✅ সহজ ও দ্রুত | ❌ অনেক setup |
| Deployment | একসাথে সব | প্রতিটি আলাদা |
| Scaling | পুরোটা একসাথে | যে service-এ লোড, শুধু সেটি |
| Technology | সাধারণত একটি stack | প্রতি service-এ আলাদা হতে পারে |
| Debugging | সহজ (এক জায়গায়) | কঠিন (distributed tracing লাগে) |
| Data consistency | সহজ (ACID) | কঠিন (eventual consistency, Saga) |
| একটি bug-এর প্রভাব | পুরো app পড়তে পারে | সীমাবদ্ধ থাকতে পারে |
| টিম সংগঠন | ছোট টিমের জন্য ভালো | বড়, স্বাধীন টিমের জন্য ভালো |
| অবকাঠামো খরচ | কম | বেশি (Docker, K8s, monitoring) |

---

## ৪. কখন কোনটি?

### ✅ Monolith বেছে নিন যখন
- নতুন প্রকল্প/startup, requirement এখনো বদলাচ্ছে
- টিম ছোট (১–১০ জন)
- Domain এখনো স্পষ্ট নয়
- দ্রুত MVP দরকার

### ✅ Microservices বেছে নিন যখন
- অনেক টিম আলাদাভাবে কাজ করে
- কিছু অংশে অনেক বেশি লোড (scale আলাদা করতে হবে)
- আলাদা deploy cycle দরকার
- Domain সীমানা (Bounded Context) স্পষ্ট

> 💡 **Martin Fowler-এর পরামর্শ — "Monolith First":** প্রায় সব সফল microservices system শুরু হয়েছিল monolith দিয়ে, পরে ভাগ করা হয়েছে।

---

## ৫. মাঝামাঝি পথ: Modular Monolith ⭐

একটি deployable unit, কিন্তু ভেতরে **স্পষ্ট module সীমানা**।

```
MyApp/
├── Modules/
│   ├── Ordering/   (নিজের Domain, Application, Data)
│   ├── Billing/
│   └── Inventory/
```
- Module গুলো একে অপরের **public interface** দিয়ে কথা বলে, ভেতরের table সরাসরি ছোঁয় না
- পরে দরকার হলে একটি module আলাদা service করা সহজ

অনেক বাস্তব প্রকল্পের জন্য **সবচেয়ে বুদ্ধিমানের শুরু**।

---

## ৬. Microservices-এ Professional Practices

1. **Database per service** — অন্য service-এর DB সরাসরি ধরবেন না
2. **API Gateway** — একটি প্রবেশপথ (YARP, Ocelot)
3. **Service যোগাযোগ** — sync (REST/gRPC) + async (Message Queue)
4. **Resilience** — Timeout, Retry, Circuit Breaker (Polly)
5. **Distributed Tracing & Logging** — Correlation ID, OpenTelemetry
6. **Centralized Config ও Secrets**
7. **Health checks** (`/health`)
8. **Containerization** — Docker + Kubernetes
9. **Saga/Outbox** — একাধিক service জুড়ে data consistency
10. **Contract testing** — service-এর API ভাঙছে কিনা পরীক্ষা

```csharp
// Polly দিয়ে Retry (সরল উদাহরণ)
builder.Services.AddHttpClient<IInventoryClient, InventoryClient>()
    .AddStandardResilienceHandler();
```

---

## ৭. সাধারণ ভুল

| ভুল | ব্যাখ্যা |
|---|---|
| শুরুতেই microservices | অকারণ জটিলতা, টিম ধীর হয়ে যায় |
| **Distributed Monolith** | Service আলাদা কিন্তু একে অপরের উপর এত নির্ভরশীল যে একসাথেই deploy করতে হয় — দুই জগতের সবচেয়ে খারাপটা |
| Shared database | Service-গুলো আসলে coupled |
| অতি ছোট (nano) service | নেটওয়ার্ক call-এর বন্যা |
| Monitoring ছাড়া | সমস্যা কোথায় খুঁজে পাওয়া যায় না |

---

## ✅ সারসংক্ষেপ

- Monolith = সহজ শুরু | Microservices = স্বাধীনতা, কিন্তু জটিলতার দাম
- অনিশ্চিত হলে **Modular Monolith** দিয়ে শুরু করুন
- Microservices-এ চাই: database per service, resilience, observability

⬅️ [Design Principles](05-design-principles.md) | ➡️ [Message Queue ও EDA](07-message-queue-and-eda.md)
