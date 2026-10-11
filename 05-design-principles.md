# Chapter ০৫: Design Principles — SOLID, DRY, KISS, YAGNI

সহজ বাংলা ব্যাখ্যা ও C# Practical Examples

## 📌 Introduction

Software Development-এ শুধু code লিখতে পারলেই হয় না। Code এমনভাবে লিখতে হয়, যাতে সেটি সহজে বোঝা যায়, maintain করা যায়, test করা যায় এবং ভবিষ্যতে পরিবর্তন করা যায়।

এই লক্ষ্য অর্জনে কিছু গুরুত্বপূর্ণ Design Principles ব্যবহার করা হয়:

- **SOLID** — Object-Oriented Design উন্নত করার পাঁচটি principle।
- **DRY** — একই business logic অপ্রয়োজনে বারবার না লেখা।
- **KISS** — সমস্যার সমাধান যতটা সম্ভব সহজ রাখা।
- **YAGNI** — বর্তমানে প্রয়োজন নেই এমন feature আগেভাগে তৈরি না করা।

এই README-তে ASP.NET Core ও C#-এর practical examples দিয়ে প্রতিটি principle ব্যাখ্যা করা হয়েছে।

---

# 1. SOLID Principles

SOLID হলো Object-Oriented Programming (OOP)-এ maintainable এবং extensible software design করার পাঁচটি গুরুত্বপূর্ণ principle।

## S — Single Responsibility Principle (SRP)

**একটি class-এর পরিবর্তনের জন্য একটি প্রধান দায়িত্ব থাকা উচিত।**

### ❌ ভুল পদ্ধতি

```csharp
public class BookingService
{
    public void CreateBooking()
    {
        // Booking create logic
    }

    public void SendEmail()
    {
        // Email sending logic
    }

    public void GenerateReport()
    {
        // Report generation logic
    }
}
```

সমস্যা: Booking, Email এবং Report-এর নিয়ম পরিবর্তন হলে একই class পরিবর্তন করতে হতে পারে।

### ✅ সঠিক পদ্ধতি

```csharp
public class BookingService
{
    public void CreateBooking()
    {
        // Booking business logic
    }
}

public class EmailService
{
    public void SendEmail()
    {
        // Email sending logic
    }
}

public class ReportService
{
    public void GenerateReport()
    {
        // Report generation logic
    }
}
```

### ব্যাখ্যা

- `BookingService` — Booking পরিচালনা করে।
- `EmailService` — Email পাঠায়।
- `ReportService` — Report তৈরি করে।

**বাস্তব ব্যবহার:** ASP.NET Core application-এ booking logic, email delivery এবং report generation-এর দায়িত্ব প্রয়োজন অনুযায়ী আলাদা রাখা।

---

## O — Open/Closed Principle (OCP)

**Software component এমনভাবে design করা উচিত, যাতে নতুন behavior যোগ করা যায়, কিন্তু existing code বারবার পরিবর্তন করতে না হয়।**

### ❌ ভুল পদ্ধতি

```csharp
public class PaymentService
{
    public void Pay(string method)
    {
        if (method == "Cash")
        {
            Console.WriteLine("Cash payment");
        }
        else if (method == "Card")
        {
            Console.WriteLine("Card payment");
        }
    }
}
```

সমস্যা: নতুন payment method যোগ করতে হলে `PaymentService`-এর conditional logic পরিবর্তন করতে হবে।

### ✅ সঠিক পদ্ধতি

```csharp
public interface IPaymentMethod
{
    void Pay(decimal amount);
}

public class CashPayment : IPaymentMethod
{
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Cash payment: {amount}");
    }
}

public class CardPayment : IPaymentMethod
{
    public void Pay(decimal amount)
    {
        Console.WriteLine($"Card payment: {amount}");
    }
}

public class PaymentService
{
    public void Process(
        IPaymentMethod paymentMethod,
        decimal amount)
    {
        paymentMethod.Pay(amount);
    }
}
```

ব্যবহার:

```csharp
var paymentService = new PaymentService();

paymentService.Process(
    new CashPayment(), 500);

paymentService.Process(
    new CardPayment(), 1000);
```

নতুন `BkashPayment` বা `NagadPayment` যোগ করতে `IPaymentMethod` implement করলেই হবে; মূল `PaymentService` পরিবর্তন করতে হবে না।

**বাস্তব ব্যবহার:** Payment Gateway, Notification Provider এবং File Storage-এর মতো পরিবর্তনশীল feature।

---

## L — Liskov Substitution Principle (LSP)

**কোনো base type-এর পরিবর্তে তার subtype ব্যবহার করলেও program-এর প্রত্যাশিত behavior ঠিক থাকতে হবে।**

### ❌ ভুল পদ্ধতি

```csharp
public class Bird
{
    public virtual void Fly()
    {
        Console.WriteLine("Flying");
    }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new NotSupportedException(
            "Penguins cannot fly.");
    }
}
```

সমস্যা: `Bird` হিসেবে ব্যবহার করা code ধরে নিতে পারে যে সব bird উড়তে পারে। কিন্তু `Penguin` সেই প্রত্যাশা পূরণ করে না।

### ✅ সঠিক পদ্ধতি

```csharp
public abstract class Bird
{
    public abstract void Eat();
}

public interface IFlyingBird
{
    void Fly();
}

public class Eagle : Bird, IFlyingBird
{
    public override void Eat()
    {
        Console.WriteLine("Eagle is eating");
    }

    public void Fly()
    {
        Console.WriteLine("Eagle is flying");
    }
}

public class Penguin : Bird
{
    public override void Eat()
    {
        Console.WriteLine("Penguin is eating");
    }
}
```

### ব্যাখ্যা

সব `Bird` খেতে পারে, কিন্তু শুধু উড়তে সক্ষম bird-গুলো `IFlyingBird` implement করে।

**বাস্তব ব্যবহার:** Inheritance ও interface এমনভাবে design করা, যাতে subtype-গুলো parent contract ভঙ্গ না করে।

---

## I — Interface Segregation Principle (ISP)

**কোনো class-কে এমন method implement করতে বাধ্য করা উচিত নয়, যেগুলো তার প্রয়োজন নেই।**

### ❌ ভুল পদ্ধতি

```csharp
public interface IWorker
{
    void Work();
    void Eat();
}

public class Robot : IWorker
{
    public void Work()
    {
        Console.WriteLine("Robot working");
    }

    public void Eat()
    {
        throw new NotSupportedException();
    }
}
```

সমস্যা: Robot-কে এমন একটি method implement করতে হচ্ছে, যা তার জন্য প্রযোজ্য নয়।

### ✅ সঠিক পদ্ধতি

```csharp
public interface IWorkable
{
    void Work();
}

public interface IEatable
{
    void Eat();
}

public class Robot : IWorkable
{
    public void Work()
    {
        Console.WriteLine("Robot working");
    }
}

public class Human : IWorkable, IEatable
{
    public void Work()
    {
        Console.WriteLine("Human working");
    }

    public void Eat()
    {
        Console.WriteLine("Human eating");
    }
}
```

**বাস্তব ব্যবহার:** বড় interface-কে ছোট, নির্দিষ্ট এবং অর্থবহ interface-এ ভাগ করা।

---

## D — Dependency Inversion Principle (DIP)

**High-level business logic সরাসরি low-level implementation-এর ওপর নির্ভর না করে abstraction-এর ওপর নির্ভর করবে।**

### ❌ ভুল পদ্ধতি

```csharp
public class EmailService
{
    public void Send(string message)
    {
        Console.WriteLine(message);
    }
}

public class BookingService
{
    private readonly EmailService _emailService
        = new EmailService();

    public void CreateBooking()
    {
        // Create booking
        _emailService.Send("Booking created");
    }
}
```

সমস্যা: `BookingService` সরাসরি `EmailService` তৈরি করছে। ফলে implementation পরিবর্তন ও unit testing কঠিন হতে পারে।

### ✅ সঠিক পদ্ধতি

```csharp
public interface INotificationService
{
    void Send(string message);
}

public class EmailNotificationService
    : INotificationService
{
    public void Send(string message)
    {
        Console.WriteLine($"Email: {message}");
    }
}

public class BookingService
{
    private readonly INotificationService _notification;

    public BookingService(
        INotificationService notification)
    {
        _notification = notification;
    }

    public void CreateBooking()
    {
        // Create booking
        _notification.Send("Booking created");
    }
}
```

ASP.NET Core Dependency Injection:

```csharp
builder.Services.AddScoped<
    INotificationService,
    EmailNotificationService>();

builder.Services.AddScoped<BookingService>();
```

এখন `BookingService`-কে সরাসরি Email implementation জানতে হয় না। প্রয়োজন হলে অন্য notification implementation register করা যায়।

**মনে রাখুন:** Dependency Inversion হলো design principle; Dependency Injection হলো dependency সরবরাহ করার একটি পদ্ধতি।

---

# 2. DRY — Don't Repeat Yourself

DRY-এর অর্থ হলো একই knowledge বা business rule অপ্রয়োজনে একাধিক জায়গায় duplicate না করা।

## ❌ ভুল পদ্ধতি

```csharp
public class BookingService
{
    public decimal CalculateVat(decimal amount)
    {
        return amount * 0.05m;
    }
}

public class OrderService
{
    public decimal CalculateVat(decimal amount)
    {
        return amount * 0.05m;
    }
}
```

সমস্যা: VAT rate পরিবর্তন হলে দুই জায়গায় code পরিবর্তন করতে হতে পারে।

## ✅ সঠিক পদ্ধতি

```csharp
public static class TaxCalculator
{
    public static decimal CalculateVat(
        decimal amount,
        decimal vatRate)
    {
        return amount * vatRate;
    }
}
```

ব্যবহার:

```csharp
decimal bookingVat =
    TaxCalculator.CalculateVat(1000m, 0.05m);

decimal orderVat =
    TaxCalculator.CalculateVat(2000m, 0.05m);
```

এখন calculation-এর logic একটি জায়গায় রয়েছে।

**গুরুত্বপূর্ণ:** DRY মানে একই রকম দেখতে সব code এক class-এ নিয়ে যাওয়া নয়। দুই জায়গার business rules আলাদা হলে অন্ধভাবে abstraction তৈরি করা উচিত নয়।

---

# 3. KISS — Keep It Simple, Stupid

KISS-এর মূল কথা হলো: **সমস্যার সমাধান যতটা সম্ভব সহজ ও পরিষ্কার রাখুন।**

## ❌ অপ্রয়োজনীয় জটিলতা

```csharp
public bool IsAdult(int age)
{
    Func<int, bool> check =
        value => value >= 18;

    return check.Invoke(age);
}
```

এই কাজের জন্য `Func`, lambda এবং `Invoke()` ব্যবহার করার প্রয়োজন নেই।

## ✅ সহজ পদ্ধতি

```csharp
public bool IsAdult(int age)
{
    return age >= 18;
}
```

ব্যবহার:

```csharp
Console.WriteLine(IsAdult(20)); // True
Console.WriteLine(IsAdult(15)); // False
```

### ASP.NET Core-এর ক্ষেত্রে

যদি একটি সাধারণ CRUD endpoint-এর জন্য জটিল architecture, একাধিক unnecessary interface, factory এবং abstraction তৈরি করতে হয়, তাহলে design আবার পর্যালোচনা করা উচিত।

**লক্ষ্য:** সহজ code, পরিষ্কার দায়িত্ব এবং প্রয়োজন অনুযায়ী abstraction।

---

# 4. YAGNI — You Aren't Gonna Need It

YAGNI-এর অর্থ হলো, বর্তমানে প্রয়োজন নেই এমন feature শুধু ভবিষ্যতে লাগতে পারে ভেবে আগেভাগে implement না করা।

## ❌ ভুল পদ্ধতি

ধরুন, আপনার Bus Booking System-এ এখন শুধু Cash Payment প্রয়োজন। কিন্তু ভবিষ্যতে প্রয়োজন হতে পারে ভেবে শুরুতেই তৈরি করলেন:

- ১০টি Payment Gateway integration
- Cryptocurrency Payment
- Multi-currency Exchange Engine
- Complex Plugin Management System

এতে development time, maintenance cost এবং complexity বেড়ে যায়।

## ✅ সঠিক পদ্ধতি

প্রথমে প্রয়োজনীয় Cash Payment feature তৈরি করুন।

```csharp
public class CashPaymentService
{
    public void Pay(decimal amount)
    {
        if (amount <= 0)
        {
            throw new ArgumentOutOfRangeException(
                nameof(amount));
        }

        Console.WriteLine(
            $"Cash payment received: {amount}");
    }
}
```

পরবর্তীতে যখন Card, bKash বা Nagad Payment-এর বাস্তব requirement আসবে, তখন প্রয়োজন অনুযায়ী design সম্প্রসারণ করুন।

**গুরুত্বপূর্ণ:** YAGNI মানে ভবিষ্যতের কথা একেবারেই চিন্তা না করা নয়। এর অর্থ হলো অনুমানের ভিত্তিতে অপ্রয়োজনীয় functionality তৈরি না করা।

---

# 5. সব Principles একসঙ্গে কীভাবে ব্যবহার করবেন?

ধরুন, আপনি একটি ASP.NET Core Bus Booking API তৈরি করছেন।

| Principle | Practical application |
|---|---|
| SRP | Booking এবং Notification-এর দায়িত্ব আলাদা রাখা |
| OCP | নতুন Payment Method যোগ করার সুযোগ রাখা |
| LSP | Subtype যেন base contract-এর behavior ভঙ্গ না করে |
| ISP | ছোট ও নির্দিষ্ট interface তৈরি করা |
| DIP | Constructor Injection-এর মাধ্যমে abstraction ব্যবহার করা |
| DRY | Repeated business rules একটি উপযুক্ত জায়গায় রাখা |
| KISS | অপ্রয়োজনীয় abstraction এড়িয়ে চলা |
| YAGNI | Requirement ছাড়া feature implement না করা |

## Recommended Development Workflow

1. Business requirements বুঝুন।
2. Business rules চিহ্নিত করুন।
3. Simple, readable implementation তৈরি করুন।
4. Duplicate logic এবং responsibility পরীক্ষা করুন।
5. প্রয়োজন হলে abstraction ও interface যোগ করুন।
6. Unit test লিখুন।
7. বাস্তব requirement অনুযায়ী design উন্নত করুন।

---

# 6. Best Practices

- প্রতিটি class-এর দায়িত্ব পরিষ্কার রাখুন।
- Interface তৈরি করুন যখন abstraction বা decoupling-এর বাস্তব প্রয়োজন আছে।
- Inheritance ব্যবহার করার আগে composition বিবেচনা করুন।
- Business rules duplicate করবেন না।
- অপ্রয়োজনীয় design pattern ব্যবহার করবেন না।
- Code testable এবং readable রাখুন।
- Refactoring-এর সময় existing behavior test দিয়ে যাচাই করুন।
- Principle-গুলোকে কঠোর নিয়ম হিসেবে নয়, design decision নেওয়ার guide হিসেবে ব্যবহার করুন।

# Conclusion

SOLID, DRY, KISS এবং YAGNI হলো maintainable software design-এর গুরুত্বপূর্ণ guidelines।

এগুলো সঠিকভাবে ব্যবহার করলে application-এর codebase বোঝা, test করা, পরিবর্তন করা এবং দীর্ঘমেয়াদে maintain করা সহজ হয়।

**Remember:** Good code is not code with the most abstractions. Good code solves the required problem clearly, correctly, and maintainably.

**Technologies:** C#, OOP, ASP.NET Core, Dependency Injection, Software Architecture.

**Topics:** SOLID Principles, DRY, KISS, YAGNI, Clean Code, Maintainable Software Design.

⬅️ [Chapter ০৪](04-putting-it-together.md) | ➡️ [Monolith vs Microservices](06-monolith-vs-microservices.md)
