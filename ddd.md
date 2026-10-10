# Chapter ০৩: DDD (Domain-Driven Design)

## ১. DDD কী?

**Domain-Driven Design** হলো সফটওয়্যার design-এর একটি পদ্ধতি, যেখানে **business-এর সমস্যা (domain)** কোডের কেন্দ্রে থাকে — database বা framework নয়।

> মূল কথা: Developer ও business expert (যেমন lab technician, ব্যাংকার) **একই ভাষায়** কথা বলবে, এবং সেই ভাষাই কোডে থাকবে।

প্রবর্তক: Eric Evans (বই: *Domain-Driven Design*, ২০০৩)।

---

## ২. কেন ব্যবহার করি? — সমস্যাটা কোথায়?

সাধারণ CRUD-style কোডে যা হয়:

```csharp
// সব logic Service-এ, Entity শুধু data বহন করে (Anemic Model)
order.Status = "Cancelled";
order.Total = 0;
```

- Business rule ছড়িয়ে যায় (Controller, Service, Stored Procedure সবখানে)
- কেউ ভুল করে rule ভেঙে data বদলে দিতে পারে
- প্রকল্প বড় হলে কোড বোঝা ও বদলানো কঠিন
- Business ও developer-এর কথোপকথনে ভুল বোঝাবুঝি

DDD এর সমাধান: **Business rule Entity-র ভেতরেই থাকবে**, বাইরে থেকে rule ভাঙা যাবে না।

---

## ৩. কখন DDD ব্যবহার করবেন, কখন নয়?

| ✅ উপযোগী | ❌ অপ্রয়োজনীয় |
|---|---|
| জটিল business rule (Lab, Banking, Insurance, Booking) | সাধারণ CRUD / Admin panel |
| বড় ও দীর্ঘমেয়াদি প্রকল্প | ছোট প্রোটোটাইপ |
| অনেকজনের টিম | একা ছোট tool |
| Domain নিয়ে business expert-এর সাথে কাজ | শুধু data দেখানো/সংরক্ষণ |

> **সতর্কতা:** সব প্রকল্পে DDD জোর করে লাগাবেন না — over-engineering হয়ে যাবে।

---

## ৪. DDD-র দুই স্তর

### 🧭 Strategic Design (বড় ছবি)

**ক) Ubiquitous Language**
Business ও developer একই শব্দ ব্যবহার করবে। যদি lab-এ বলে "Sample Collect করা", কোডে `sample.Collect()` — `UpdateStatus(3)` নয়।

**খ) Bounded Context**
একই শব্দের অর্থ বিভিন্ন জায়গায় ভিন্ন হতে পারে। প্রতিটি অংশের নিজস্ব সীমানা।

```
┌────────────┐   ┌────────────┐   ┌────────────┐
│  Ordering  │   │  Billing   │   │  Reporting │
│ (Customer= │   │ (Customer= │   │ (Customer= │
│ যিনি order │   │ যিনি bill  │   │ statistic) │
│  দেন)      │   │  দেন)      │   │            │
└────────────┘   └────────────┘   └────────────┘
```

**গ) Context Map** — Context-গুলো একে অপরের সাথে কীভাবে যোগাযোগ করে।

### 🧱 Tactical Design (কোডের building block)

| Block | মানে | উদাহরণ |
|---|---|---|
| **Entity** | নিজস্ব পরিচয় (Id) আছে, সময়ের সাথে বদলায় | `Order`, `Customer` |
| **Value Object** | পরিচয় নেই, মান দিয়ে চেনা যায়, immutable | `Money`, `Address`, `Email` |
| **Aggregate** | সম্পর্কিত object-এর group, একটি Root-এর মাধ্যমে নিয়ন্ত্রিত | `Order` + `OrderItem` |
| **Aggregate Root** | Aggregate-এর একমাত্র প্রবেশদ্বার | `Order` |
| **Domain Event** | "কিছু ঘটেছে" — অতীতকালে নাম | `OrderPlaced` |
| **Repository** | Aggregate save/load করার interface | `IOrderRepository` |
| **Domain Service** | যে logic কোনো এক Entity-র নয় | `PricingService` |
| **Factory** | জটিল object তৈরি | `OrderFactory` |

---

## ৫. কোড উদাহরণ

### Value Object — `Money`

```csharp
public sealed record Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        if (amount < 0) throw new DomainException("Amount ঋণাত্মক হতে পারে না।");
        Amount = amount;
        Currency = currency;
    }

    public Money Add(Money other) =>
        Currency == other.Currency
            ? new Money(Amount + other.Amount, Currency)
            : throw new DomainException("Currency মেলেনি।");
}
```

### Aggregate Root — `Order`

```csharp
public class Order
{
    private readonly List<OrderItem> _items = new();

    public int Id { get; private set; }
    public OrderStatus Status { get; private set; } = OrderStatus.Draft;
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();

    // বাইরে থেকে _items সরাসরি বদলানো যাবে না
    public void AddItem(int productId, Money price, int qty)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Draft ছাড়া item যোগ করা যায় না।");
        if (qty <= 0)
            throw new DomainException("Quantity অবশ্যই ১ বা তার বেশি।");

        _items.Add(new OrderItem(productId, price, qty));
    }

    public void Place()
    {
        if (!_items.Any())
            throw new DomainException("খালি order place করা যায় না।");

        Status = OrderStatus.Placed;
        // Domain Event raise করা যায় এখানে
    }

    public void Cancel()
    {
        if (Status == OrderStatus.Shipped)
            throw new DomainException("Ship হয়ে গেলে cancel করা যায় না।");
        Status = OrderStatus.Cancelled;
    }
}
```

লক্ষ্য করুন: `Status` ও `_items` **private setter/field** — rule না মেনে বদলানোর উপায় নেই। 🔒

### Repository

```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id);
    Task AddAsync(Order order);
}
```

---

## ৬. Layered / Clean Architecture-এ DDD

```
┌─────────────────────────────────────────┐
│ API / Presentation (Controllers)        │
├─────────────────────────────────────────┤
│ Application (Use Cases, Commands, DTO)  │
├─────────────────────────────────────────┤
│ Domain (Entity, VO, Aggregate, Events)  │ ← কেন্দ্র, কিছুর উপর নির্ভর করে না
├─────────────────────────────────────────┤
│ Infrastructure (EF Core, SQL, Email)    │
└─────────────────────────────────────────┘
```

**Dependency Rule:** নির্ভরতা সবসময় **ভেতরের দিকে**। Domain layer EF Core বা ASP.NET চেনে না।

---

## ৭. Professional-grade Practices

1. **Aggregate ছোট রাখুন** — বড় aggregate = lock/performance সমস্যা
2. **এক transaction = এক aggregate** — একাধিক aggregate-এর মধ্যে Domain Event/eventual consistency
3. **অন্য aggregate-কে Id দিয়ে reference করুন**, পুরো object দিয়ে নয়
4. **Anemic model এড়ান** — logic Entity-র ভেতরে রাখুন
5. **Primitive obsession এড়ান** — `string email` না, `Email` Value Object
6. **Domain Event-এর নাম অতীতকালে** — `OrderPlaced`, `PaymentReceived`
7. **Domain-এ framework dependency নয়** — pure C#
8. **Unit test সহজ** — DB ছাড়াই domain test করা যায়
9. **CQRS-এর সাথে ভালো মেলে** — Write side-এ DDD, Read side-এ সরাসরি SQL/Dapper

---

## ৮. সাধারণ ভুল

| ভুল | ব্যাখ্যা |
|---|---|
| সব প্রকল্পে DDD | সাধারণ CRUD-এ অতিরিক্ত জটিলতা |
| Public setter সব জায়গায় | Rule ভাঙার সুযোগ থেকে যায় |
| বিশাল Aggregate | `Customer` ভেতরে সব Order, Payment... |
| Business expert ছাড়া design | Ubiquitous Language তৈরি হয় না |
| Entity = Database Table ভাবা | Domain model ≠ DB schema |

---

## ✅ সারসংক্ষেপ

- DDD = Business কেন্দ্রিক design + সবার জন্য এক ভাষা
- Strategic: Ubiquitous Language, Bounded Context
- Tactical: Entity, Value Object, Aggregate, Repository, Domain Event
- জটিল domain-এ ব্যবহার করুন; সাধারণ CRUD-এ নয়

⬅️ আগের chapter: [ACID](02-acid.md) | ➡️ পরের: [তিনটি একসাথে](04-putting-it-together.md)
