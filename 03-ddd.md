# Domain-Driven Design (DDD) — সহজ বাংলায়

## 📌 Domain-Driven Design (DDD) কী?

**Domain-Driven Design (DDD)** হলো একটি software design approach, যেখানে software-এর structure এবং business logic তৈরি করা হয় business requirements ও business rules-এর ওপর ভিত্তি করে।

সহজভাবে বললে, software তৈরি করার আগে business কীভাবে কাজ করে, কী কী নিয়ম আছে এবং কোন সমস্যাগুলো সমাধান করতে হবে—সেগুলো ভালোভাবে বুঝে সেই অনুযায়ী application design করাই DDD-এর মূল উদ্দেশ্য।

## 🎯 DDD কেন ব্যবহার করা হয়?

একটি বড় software application-এ অনেক business rules, validation এবং complex operations থাকে। সব logic Controller বা Service-এর মধ্যে রাখলে application maintain করা কঠিন হয়ে যায়।

DDD ব্যবহার করলে:

- Business logic সুন্দরভাবে organize করা যায়।
- Code maintain ও test করা সহজ হয়।
- Business rules একটি নির্দিষ্ট জায়গায় রাখা যায়।
- Application-এর complexity নিয়ন্ত্রণ করা যায়।
- Developer এবং business team একই terminology ব্যবহার করতে পারে।
- নতুন features যোগ করা এবং existing code পরিবর্তন করা সহজ হয়।

## 🚌 বাস্তব উদাহরণ: Bus Booking System

ধরুন, আমরা একটি Bus Booking System তৈরি করছি।

এখানে কিছু গুরুত্বপূর্ণ business rules রয়েছে:

- একই seat একই সময়ে একাধিক customer বুক করতে পারবে না।
- Payment সম্পন্ন না হলে booking Confirmed করা যাবে না।
- Booking cancel হলে নির্দিষ্ট নিয়ম অনুযায়ী seat পুনরায় Available হতে পারে।
- Expired seat lock নির্দিষ্ট সময় পরে release করতে হবে।

DDD ব্যবহার করে এই business rules-গুলোকে application-এর Domain Model-এর মধ্যে সঠিকভাবে design ও enforce করা যায়।

## 🧩 DDD-এর গুরুত্বপূর্ণ Concepts

### 1. Domain

Domain হলো সেই business area, যার সমস্যা সমাধানের জন্য software তৈরি করা হয়।

উদাহরণ:
- Bus Booking System
- Pharmacy Management System
- Banking System
- E-commerce System

### 2. Entity

Entity হলো এমন একটি object, যার নিজস্ব identity থাকে এবং সময়ের সঙ্গে তার state পরিবর্তন হতে পারে।

উদাহরণ: `Booking`, `Customer`, `Bus`

একটি Booking-এর `Id = 101` থাকলে তার status পরিবর্তন হলেও Booking-টির identity একই থাকে।

### 3. Value Object

Value Object এমন একটি object, যা আলাদা identity-এর পরিবর্তে তার value দিয়ে চিহ্নিত হয়।

উদাহরণ:
- `Money`
- `Address`
- `DateRange`

সঠিকভাবে design করা Value Object সাধারণত immutable হয়।

### 4. Aggregate

Aggregate হলো related domain objects-এর একটি group, যাদের consistency ও business rules একটি নির্দিষ্ট boundary-এর মধ্যে নিয়ন্ত্রণ করা হয়।

Aggregate-এর বাইরের code সাধারণত Aggregate Root-এর মাধ্যমে সেই group-এর state পরিবর্তন করে।

উদাহরণ: `Booking` একটি Aggregate Root হতে পারে, যা booking-এর status ও সংশ্লিষ্ট business rules নিয়ন্ত্রণ করে।

### 5. Repository

Repository হলো Domain Model-এর জন্য persistence access-এর abstraction।

এটি database থেকে Entity load করা এবং পরিবর্তিত state সংরক্ষণ করার কাজকে application-এর business logic থেকে আলাদা রাখতে সাহায্য করে।

উদাহরণ: `IBookingRepository`

### 6. Domain Service

কিছু business operation একটি Entity বা Value Object-এর মধ্যে স্বাভাবিকভাবে রাখা যায় না। সেসব operation-এর জন্য Domain Service ব্যবহার করা যেতে পারে।

### 7. Domain Event

Domain Event হলো Domain-এর মধ্যে ঘটে যাওয়া একটি গুরুত্বপূর্ণ business ঘটনা।

উদাহরণ:
- `BookingConfirmed`
- `BookingCancelled`
- `PaymentCompleted`

Domain Event-এর মাধ্যমে সংশ্লিষ্ট business process-গুলোকে decouple করা যায়।

### 8. Ubiquitous Language

Developer, business analyst এবং domain expert যেন একই business terminology ব্যবহার করেন, সেটিই Ubiquitous Language-এর মূল ধারণা।

উদাহরণ: `Booking`, `Seat Lock`, `Payment`, `Cancellation`।

## 🏗️ ASP.NET Core-এ DDD Project Structure

ASP.NET Core Web API-তে DDD-এর সঙ্গে Clean Architecture ব্যবহার করলে একটি সম্ভাব্য solution structure হতে পারে:

```text
BPS.sln
│
├── BPS.Domain
│   ├── Entities
│   ├── ValueObjects
│   ├── Aggregates
│   ├── DomainServices
│   └── DomainEvents
│
├── BPS.Application
│   ├── Features
│   ├── DTOs
│   ├── Interfaces
│   └── UseCases
│
├── BPS.Infrastructure
│   ├── Persistence
│   ├── Repositories
│   └── Database
│
└── BPS.API
    ├── Controllers
    ├── Middleware
    └── Configuration
```

### Project-গুলোর দায়িত্ব

| Project | দায়িত্ব |
|---|---|
| `BPS.Domain` | Entities, Value Objects ও Business Rules |
| `BPS.Application` | Use Cases, DTOs ও Application Interfaces |
| `BPS.Infrastructure` | Database, ADO.NET ও Repository Implementations |
| `BPS.API` | HTTP Requests, Controllers ও API Configuration |

**Dependency Rule:** Domain Layer যতটা সম্ভব independent থাকবে। Infrastructure Layer-এর implementation details সরাসরি Domain Layer-এর ওপর নির্ভরশীল হবে না।

এই structure DDD-এর একমাত্র বাধ্যতামূলক structure নয়। Project-এর complexity অনুযায়ী structure পরিবর্তন করা যায়।

## 🔄 Booking Confirmation-এর উদাহরণ

একটি booking confirm করার সময় সাধারণ flow হতে পারে:

```text
Angular Frontend
       |
       v
ASP.NET Core API
       |
       v
Application Layer
       |
       v
Domain Model
       |
       v
Infrastructure Layer
       |
       v
SQL Server
```

এই flow-তে:

1. API booking confirmation request গ্রহণ করে।
2. Application Layer confirmation use case পরিচালনা করে।
3. Domain Model business rules যাচাই করে।
4. Infrastructure Layer database operation সম্পন্ন করে।
5. সফল হলে API response ফেরত দেয়।

বাস্তব booking system-এ payment verification, seat locking, concurrency control এবং database transaction-ও সঠিকভাবে পরিচালনা করতে হবে।

## ⚖️ DDD কখন ব্যবহার করা উচিত?

**DDD উপযোগী হতে পারে যখন:**
- Business rules জটিল।
- একাধিক business workflow রয়েছে।
- Application দীর্ঘমেয়াদে maintain করতে হবে।
- Business logic সঠিকভাবে test করা প্রয়োজন।
- একাধিক team একই domain নিয়ে কাজ করে।

**DDD সব project-এর জন্য প্রয়োজনীয় নয়।**

সাধারণ CRUD application বা ছোট project-এর ক্ষেত্রে একটি সহজ layered architecture যথেষ্ট হতে পারে। অপ্রয়োজনীয় abstraction তৈরি করলে DDD application-কে আরও জটিল করে তুলতে পারে।

## 🛠️ DDD-এর সঙ্গে যেসব Technology ব্যবহার করা যায়

- C#
- ASP.NET Core Web API
- Entity Framework Core
- ADO.NET
- SQL Server
- Clean Architecture
- CQRS
- Domain Events
- Unit Testing

এগুলো DDD-এর বাধ্যতামূলক অংশ নয়; project-এর প্রয়োজন অনুযায়ী ব্যবহার করা হয়।

## 📚 শেখার Roadmap

1. Domain এবং Business Rules বুঝুন।
2. Entity ও Value Object তৈরি করা শিখুন।
3. Aggregate এবং Aggregate Root বুঝুন।
4. Repository Pattern ও Domain Service শিখুন।
5. ASP.NET Core-এ Domain Model implement করুন।
6. Clean Architecture-এর সঙ্গে DDD ব্যবহার করুন।
7. Unit Testing ও Integration Testing করুন।
8. প্রয়োজন অনুযায়ী CQRS এবং Domain Events যুক্ত করুন।

## ✅ Conclusion

Domain-Driven Design-এর মূল লক্ষ্য শুধু code-কে বিভিন্ন project বা folder-এ ভাগ করা নয়। এর উদ্দেশ্য হলো business requirements এবং rules-কে সঠিকভাবে model করা, business logic-কে পরিষ্কার রাখা এবং complex software-কে maintainable ও testable করা।

**Remember:** Good software design starts with understanding the business, not just writing code.

---

**Author:** Joni Miah  
**Topic:** Software Architecture | Domain-Driven Design | ASP.NET Core


⬅️ আগের chapter: [ACID](02-acid.md) | ➡️ পরের: [তিনটি একসাথে](04-putting-it-together.md)
