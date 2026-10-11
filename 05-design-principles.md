# Chapter ০৫: Design Principles — SOLID, DRY, KISS, YAGNI

## ১. এগুলো কী ও কেন দরকার?

এগুলো কোনো framework বা technology নয় — **ভালো কোড লেখার দিকনির্দেশনা**। লক্ষ্য একটাই: কোড যেন **বুঝতে সহজ, বদলাতে সহজ, test করতে সহজ** হয়।

প্রকল্প ছোট থাকলে সবই সহজ। কিন্তু ছয় মাস পরে নতুন feature যোগ করতে গিয়ে পুরনো কোড ভেঙে পড়লে বোঝা যায় এই নীতিগুলোর দাম।

---

## ২. SOLID

SOLID = ৫টি OOP design principle (Robert C. Martin / Uncle Bob)।

### S — Single Responsibility Principle (SRP)
> একটি class-এর **পরিবর্তনের কারণ একটিই** হওয়া উচিত।

❌ **খারাপ:** একটি class সব করছে
```csharp
public class OrderService
{
    public void PlaceOrder(Order o) { /* business logic */ }
    public void SaveToDb(Order o)   { /* SQL */ }
    public void SendEmail(Order o)  { /* SMTP */ }
}
```
✅ **ভালো:** দায়িত্ব আলাদা
```csharp
public class OrderService    { /* শুধু business logic */ }
public class OrderRepository { /* শুধু DB */ }
public class EmailSender     { /* শুধু email */ }
```

### O — Open/Closed Principle (OCP)
> Class **extend করার জন্য খোলা**, কিন্তু **modify করার জন্য বন্ধ**।

❌ নতুন discount এলেই `if/else` বাড়ে:
```csharp
if (type == "Regular") ... else if (type == "Premium") ... // বাড়তেই থাকে
```
✅ নতুন class যোগ করুন, পুরনো কোড ছোঁবেন না:
```csharp
public interface IDiscount { decimal Apply(decimal amount); }
public class RegularDiscount : IDiscount { public decimal Apply(decimal a) => a * 0.95m; }
public class PremiumDiscount : IDiscount { public decimal Apply(decimal a) => a * 0.80m; }
```

### L — Liskov Substitution Principle (LSP)
> Child class-কে Parent-এর জায়গায় বসালে program **ভাঙবে না**।

❌ ক্লাসিক উদাহরণ: `Penguin : Bird` যেখানে `Fly()` exception ছোঁড়ে।
✅ সমাধান: `Bird` থেকে `Fly()` সরিয়ে `IFlyable` interface আলাদা করুন।

### I — Interface Segregation Principle (ISP)
> Client-কে এমন method implement করতে বাধ্য করবেন না যা সে ব্যবহার করে না।

❌ একটি বিশাল `IWorker { Work(); Eat(); Sleep(); }`
✅ ছোট ছোট `IWorkable`, `IFeedable`

### D — Dependency Inversion Principle (DIP)
> High-level module **concrete class**-এর উপর নয়, **abstraction (interface)**-এর উপর নির্ভর করবে।

❌ `private SqlOrderRepository _repo = new SqlOrderRepository();`
✅ Constructor-এ interface নিন, DI container inject করবে:
```csharp
public class OrderService(IOrderRepository repo) { }

builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();
```
এতে test-এ fake repository বসানো যায়।

---

## ৩. DRY — Don't Repeat Yourself

> একই **knowledge/logic** একাধিক জায়গায় থাকবে না।

❌ ৫টি controller-এ একই validation কোড কপি-পেস্ট।
✅ একটি shared method/validator/filter/middleware।

> ⚠️ **সতর্কতা:** দুটি কোড **দেখতে এক** হলেই merge করবেন না। যদি তারা **ভিন্ন কারণে বদলায়**, তবে তারা আলাদা জিনিস। ভুল abstraction, duplication-এর চেয়ে বেশি ক্ষতিকর।

---

## ৪. KISS — Keep It Simple, Stupid

> সবচেয়ে সহজ সমাধানই সাধারণত সেরা।

❌ ছোট একটি কাজের জন্য Factory + Strategy + Observer।
✅ একটি সাধারণ method।

নিজেকে প্রশ্ন করুন: *"ছয় মাস পরে নতুন কেউ এটা কত তাড়াতাড়ি বুঝবে?"*

---

## ৫. YAGNI — You Aren't Gonna Need It

> **এখন দরকার নেই**, এমন feature আগে থেকে বানাবেন না।

❌ "ভবিষ্যতে হয়তো multi-tenant লাগবে" ভেবে আজই পুরো কাঠামো বানানো।
✅ আজকের requirement বানান; দরকার হলে তখন refactor করুন।

---

## ৬. পরস্পরের সম্পর্ক ও ভারসাম্য

| নীতি | ঝুঁকি (অতিরিক্ত করলে) |
|---|---|
| SOLID | ছোট কাজে অতিরিক্ত interface/class (over-engineering) |
| DRY | জোর করে merge → ভুল abstraction |
| KISS | অতিরিক্ত সরল করতে গিয়ে rule বাদ পড়া |
| YAGNI | সত্যিকারের জানা requirement উপেক্ষা |

> **মনে রাখুন:** এগুলো **নিয়ম নয়, দিকনির্দেশনা**। KISS ও YAGNI প্রায়ই SOLID-এর অতিরিক্ত ব্যবহার থামায়।

---

## ✅ সারসংক্ষেপ

- **SRP** একটি কাজ | **OCP** বাড়াও, বদলিও না | **LSP** child বদলযোগ্য | **ISP** ছোট interface | **DIP** abstraction-এ নির্ভর
- **DRY** পুনরাবৃত্তি কমাও | **KISS** সহজ রাখো | **YAGNI** আজ যা লাগে তাই

⬅️ [Chapter ০৪](04-putting-it-together.md) | ➡️ [Monolith vs Microservices](06-monolith-vs-microservices.md)
