# Domain-Driven Design (DDD) with C# — সহজ বাংলায়

Domain-Driven Design (DDD) হলো এমন একটি software design approach, যেখানে business requirements এবং business rules-এর ওপর ভিত্তি করে software-এর Domain Model তৈরি করা হয়।

এই README-তে ASP.NET Core Web API এবং C# ব্যবহার করে একটি Bus Booking System-এর উদাহরণের মাধ্যমে DDD-এর গুরুত্বপূর্ণ concepts ব্যাখ্যা করা হয়েছে।

## 📌 ১. Domain-Driven Design (DDD) কী?

ধরুন, আমরা একটি Bus Booking System তৈরি করছি।

আমাদের business requirements:

- Customer একটি bus-এর seat বুক করতে পারবে।
- একই seat একাধিক customer বুক করতে পারবে না।
- Payment সম্পন্ন না হলে booking Confirmed হবে না।
- Booking cancel হলে business rules অনুযায়ী seat আবার Available হবে।

এই নিয়মগুলোই আমাদের application-এর **Business Rules**।

DDD-এর উদ্দেশ্য হলো business rules-গুলোকে সঠিকভাবে model করা এবং code-এর মধ্যে enforce করা।

## 🏗️ ২. Project Structure

ASP.NET Core solution-এর একটি সম্ভাব্য structure:

```text
BPS.sln
│
├── BPS.Domain
│   ├── Entities
│   │   └── Booking.cs
│   └── ValueObjects
│
├── BPS.Application
│   └── UseCases
│       └── ConfirmBooking.cs
│
├── BPS.Infrastructure
│   └── Repositories
│       └── BookingRepository.cs
│
└── BPS.API
    └── Controllers
        └── BookingController.cs
```

প্রতিটি layer-এর দায়িত্ব:

| Layer | কাজ |
|---|---|
| Domain | Business rules এবং domain model |
| Application | Business use case পরিচালনা |
| Infrastructure | Database ও external services |
| API | HTTP request এবং response |

এটি DDD-এর সঙ্গে Clean Architecture ব্যবহার করার একটি উদাহরণ। DDD নিজে এই নির্দিষ্ট folder structure বাধ্যতামূলক করে না।

## 🧩 ৩. Entity কী? — বাস্তব উদাহরণ

Entity হলো এমন একটি object, যার নিজস্ব identity থাকে।

উদাহরণ: `Booking`।

একটি booking-এর ID `101`। Booking-এর status পরিবর্তন হলেও তার ID একই থাকে।

**File: `BPS.Domain/Entities/Booking.cs`**

```csharp
namespace BPS.Domain.Entities;

public class Booking
{
    public int Id { get; private set; }

    public string Status { get; private set; } = "Pending";

    private Booking()
    {
    }

    public Booking(int id)
    {
        Id = id;
    }

    public void Confirm(bool paymentCompleted)
    {
        if (!paymentCompleted)
        {
            throw new InvalidOperationException(
                "Payment is not completed.");
        }

        if (Status != "Pending")
        {
            throw new InvalidOperationException(
                "Only pending bookings can be confirmed.");
        }

        Status = "Confirmed";
    }

    public void Cancel()
    {
        if (Status != "Pending" && Status != "Confirmed")
        {
            throw new InvalidOperationException(
                "Booking cannot be cancelled.");
        }

        Status = "Cancelled";
    }
}
```

### এখানে কী হচ্ছে?

- `Id` হলো Entity-এর identity।
- `private set` বাইরের code থেকে সরাসরি status পরিবর্তন সীমিত করে।
- `Confirm()` payment এবং booking status যাচাই করে।
- `Cancel()` শুধু নির্ধারিত status থেকে cancellation অনুমোদন করে।

**DDD-এর মূল শিক্ষা:** Business rules-কে শুধু Controller-এর মধ্যে না রেখে Domain Model-এর মধ্যে enforce করা।

এই উদাহরণে status string ব্যবহার করা হয়েছে সহজভাবে বোঝানোর জন্য। Production code-এ enum বা আরও শক্তিশালী domain types ব্যবহার করা যেতে পারে।

## 🧪 ৪. Entity-কে ব্যবহার করে দেখা

**Example: `Program.cs` অথবা একটি Console Test**

```csharp
using BPS.Domain.Entities;

var booking = new Booking(101);

Console.WriteLine(booking.Status);
// Output: Pending

booking.Confirm(paymentCompleted: true);

Console.WriteLine(booking.Status);
// Output: Confirmed
```

Payment সম্পন্ন না হলে:

```csharp
var booking = new Booking(102);

booking.Confirm(paymentCompleted: false);
```

Expected result:

```text
System.InvalidOperationException:
Payment is not completed.
```

এখানে booking-এর status পরিবর্তন হওয়ার আগে Domain Model business rule যাচাই করেছে।

## 💎 ৫. Value Object কী?

Value Object হলো এমন একটি object, যার পরিচয় আলাদা ID দিয়ে নয়; তার value দিয়ে নির্ধারিত হয়।

উদাহরণ: `Money`।

**File: `BPS.Domain/ValueObjects/Money.cs`**

```csharp
namespace BPS.Domain.ValueObjects;

public sealed record Money(decimal Amount, string Currency)
{
    public Money
    {
        if (Amount < 0)
        {
            throw new ArgumentOutOfRangeException(
                nameof(Amount));
        }

        if (string.IsNullOrWhiteSpace(Currency))
        {
            throw new ArgumentException(
                "Currency is required.");
        }
    }
}
```

উপরের compact constructor syntax সাধারণ C# class-এর জন্য বৈধ নয়। পরিবর্তে নিচের compile-ready implementation ব্যবহার করুন:

```csharp
namespace BPS.Domain.ValueObjects;

public sealed record Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        if (amount < 0)
            throw new ArgumentOutOfRangeException(nameof(amount));

        if (string.IsNullOrWhiteSpace(currency))
            throw new ArgumentException(
                "Currency is required.", nameof(currency));

        Amount = amount;
        Currency = currency;
    }
}
```

ব্যবহার:

```csharp
var ticketPrice = new Money(500, "BDT");
var anotherPrice = new Money(500, "BDT");

Console.WriteLine(ticketPrice == anotherPrice);
// Output: True
```

দুটি `Money` object-এর value একই, তাই record value equality ব্যবহার করে তাদের সমান হিসেবে তুলনা করে।

## 📦 ৬. Repository Pattern কী?

Repository হলো Domain Model-এর data load এবং save করার abstraction।

Domain-এ interface রাখা যায়:

**File: `BPS.Domain/Repositories/IBookingRepository.cs`**

```csharp
using BPS.Domain.Entities;

namespace BPS.Domain.Repositories;

public interface IBookingRepository
{
    Task<Booking?> GetByIdAsync(
        int id,
        CancellationToken cancellationToken = default);

    Task SaveAsync(
        Booking booking,
        CancellationToken cancellationToken = default);
}
```

Infrastructure project-এ এই interface-এর implementation থাকবে। সেখানে ADO.NET, Entity Framework Core বা অন্য persistence technology ব্যবহার করা যেতে পারে।

Repository interface থাকার কারণে business logic-কে সরাসরি SQL query লিখতে হয় না।

## ⚙️ ৭. Application Layer-এ Use Case

Application Layer নির্দিষ্ট business operation পরিচালনা করে।

**File: `BPS.Application/UseCases/ConfirmBooking.cs`**

```csharp
using BPS.Domain.Repositories;

namespace BPS.Application.UseCases;

public sealed class ConfirmBooking
{
    private readonly IBookingRepository _repository;

    public ConfirmBooking(IBookingRepository repository)
    {
        _repository = repository;
    }

    public async Task ExecuteAsync(
        int bookingId,
        bool paymentCompleted,
        CancellationToken cancellationToken = default)
    {
        var booking = await _repository.GetByIdAsync(
            bookingId, cancellationToken);

        if (booking is null)
        {
            throw new InvalidOperationException(
                "Booking not found.");
        }

        booking.Confirm(paymentCompleted);

        await _repository.SaveAsync(
            booking, cancellationToken);
    }
}
```

এখানে:

1. Repository থেকে booking load করা হচ্ছে।
2. Booking না থাকলে error দেওয়া হচ্ছে।
3. Domain Entity-এর `Confirm()` method business rules যাচাই করছে।
4. সফল হলে Repository দিয়ে পরিবর্তন save করা হচ্ছে।

এই code-এ payment status একটি Boolean হিসেবে দেওয়া হয়েছে শুধু উদাহরণের জন্য। Production system-এ payment-এর সত্যতা server-side payment record বা trusted payment service থেকে যাচাই করতে হবে।

## 🌐 ৮. API Controller কীভাবে কাজ করবে?

Controller HTTP request গ্রহণ করে Application Layer-এর use case চালাবে।

```csharp
[ApiController]
[Route("api/bookings")]
public class BookingController : ControllerBase
{
    private readonly ConfirmBooking _confirmBooking;

    public BookingController(ConfirmBooking confirmBooking)
    {
        _confirmBooking = confirmBooking;
    }

    [HttpPost("{id:int}/confirm")]
    public async Task<IActionResult> Confirm(
        int id,
        CancellationToken cancellationToken)
    {
        await _confirmBooking.ExecuteAsync(
            id,
            paymentCompleted: true,
            cancellationToken);

        return Ok(new
        {
            Message = "Booking confirmed successfully."
        });
    }
}
```

উদাহরণ request:

```http
POST /api/bookings/101/confirm
```

সফল response:

```json
{
  "message": "Booking confirmed successfully."
}
```

এটি flow বোঝানোর উদাহরণ। Production API-তে authentication, authorization, trusted payment verification এবং exception-to-HTTP mapping যুক্ত করতে হবে। Controller থেকে `paymentCompleted: true` সরাসরি দেওয়া নিরাপদ নয়।

## 🔄 ৯. Aggregate ও Aggregate Root

Aggregate হলো related domain objects-এর consistency boundary। Aggregate Root হলো সেই boundary-এর প্রধান Entity, যার মাধ্যমে বাইরের code state পরিবর্তন করে।

উদাহরণ:

```text
Booking (Aggregate Root)
   |
   ├── BookingId
   ├── CustomerId
   ├── BookingStatus
   └── TicketPrice (Money)
```

যদি `Booking` Aggregate Root হয়, তাহলে বাইরের code সরাসরি তার internal state পরিবর্তন না করে `Confirm()` বা `Cancel()`-এর মতো method ব্যবহার করবে।

**গুরুত্বপূর্ণ:** একটি seat একই সময়ে একাধিক customer-এর কাছে বিক্রি হওয়া ঠেকাতে শুধু Domain Entity যথেষ্ট নয়। Database transaction, concurrency control এবং atomic seat-lock operation-ও প্রয়োজন।

## 📢 ১০. Domain Event কী?

Domain Event হলো Domain-এ ঘটে যাওয়া গুরুত্বপূর্ণ business ঘটনা।

উদাহরণ: `BookingConfirmed`।

```csharp
public sealed record BookingConfirmed(
    int BookingId,
    DateTime ConfirmedAtUtc);
```

Booking confirm হওয়ার পর এই event প্রকাশ করা যেতে পারে। অন্য component event-টি ব্যবহার করে ticket notification বা অন্যান্য workflow চালাতে পারে।

Event publishing এবং database save-এর মধ্যে consistency নিশ্চিত করতে production system-এ Outbox Pattern-এর মতো পদ্ধতি প্রয়োজন হতে পারে।

## 🆚 ১১. সাধারণ CRUD বনাম DDD

| সাধারণ CRUD | DDD |
|---|---|
| মূলত data create, read, update, delete করে | Business rules ও domain behavior-কে গুরুত্ব দেয় |
| Logic Controller বা Service-এ কেন্দ্রীভূত হতে পারে | Domain behavior Entity বা Domain Service-এ model করা হয় |
| সাধারণ application-এর জন্য সহজ | Complex business domain-এর জন্য কার্যকর |
| কম abstraction প্রয়োজন হতে পারে | প্রয়োজন অনুযায়ী অতিরিক্ত abstraction থাকতে পারে |

সব application-এ DDD ব্যবহার করা প্রয়োজন নেই। ছোট CRUD application-এর জন্য simple layered architecture যথেষ্ট হতে পারে।

## 🗺️ ১২. DDD শেখার Roadmap

1. Domain ও Business Rules
2. Entity ও Value Object
3. Aggregate ও Aggregate Root
4. Repository Pattern
5. Domain Service ও Domain Event
6. ASP.NET Core Application Layer
7. Clean Architecture
8. Unit Testing ও Integration Testing
9. প্রয়োজন অনুযায়ী CQRS ও Outbox Pattern

## ✅ Conclusion

Domain-Driven Design-এর মূল উদ্দেশ্য শুধু project-কে বিভিন্ন folder-এ ভাগ করা নয়। বরং business requirements বুঝে এমন Domain Model তৈরি করা, যা business rules enforce করে এবং complex application-কে maintainable ও testable রাখে।

**Remember:** Understand the business first, model the domain, and then write the code.

**Technologies:** C#, ASP.NET Core Web API, SQL Server, ADO.NET, Clean Architecture, Unit Testing.


**Author:** Joni Miah  
**Topic:** Software Architecture | Domain-Driven Design | ASP.NET Core


⬅️ আগের chapter: [ACID](02-acid.md) | ➡️ পরের: [তিনটি একসাথে](04-putting-it-together.md)
