# Chapter ০৬: Monolithic বনাম Microservices

একটি software application কীভাবে design, develop, deploy এবং scale করা হয়, তার অন্যতম গুরুত্বপূর্ণ সিদ্ধান্ত হলো Architecture নির্বাচন করা।

সাধারণত software development-এ বহুল ব্যবহৃত দুটি architecture হলো:

- **Monolithic Architecture**
- **Microservices Architecture**

এই গাইডে আমরা জানব—এগুলো কী, কেন ব্যবহার করা হয়, কীভাবে কাজ করে, এদের মধ্যে পার্থক্য কী এবং কোন পরিস্থিতিতে কোন architecture ব্যবহার করা উচিত।

---

## 📚 Table of Contents

1. [Monolithic Architecture কী?](#1-monolithic-architecture-কী)
2. [Monolithic কীভাবে কাজ করে?](#2-monolithic-কীভাবে-কাজ-করে)
3. [Monolithic-এর Practical Example](#3-monolithic-এর-practical-example)
4. [Microservices Architecture কী?](#4-microservices-architecture-কী)
5. [Microservices কীভাবে কাজ করে?](#5-microservices-কীভাবে-কাজ-করে)
6. [Microservices-এর Practical Example](#6-microservices-এর-practical-example)
7. [Monolithic বনাম Microservices](#7-monolithic-বনাম-microservices)
8. [Database Design-এর পার্থক্য](#8-database-design-এর-পার্থক্য)
9. [Scaling কীভাবে কাজ করে?](#9-scaling-কীভাবে-কাজ-করে)
10. [Docker ও Kubernetes-এর ভূমিকা](#10-docker-ও-kubernetes-এর-ভূমিকা)
11. [Security ও Communication](#11-security-ও-communication)
12. [Production-Grade Architecture](#12-production-grade-architecture)
13. [কখন কোন Architecture ব্যবহার করবেন?](#13-কখন-কোন-architecture-ব্যবহার-করবেন)
14. [Monolith থেকে Microservices-এ Migration](#14-monolith-থেকে-microservices-এ-migration)
15. [Best Practices](#15-best-practices)
16. [সারসংক্ষেপ](#16-সারসংক্ষেপ)

---

# 1. Monolithic Architecture কী?

**Monolithic Architecture** হলো এমন একটি software architecture যেখানে application-এর একাধিক business feature সাধারণত একটি application unit-এর মধ্যে তৈরি, build এবং deploy করা হয়।

উদাহরণ হিসেবে একটি Bus Booking System বিবেচনা করা যাক।

এই application-এ রয়েছে:

- Authentication
- User Management
- Bus Management
- Route Management
- Trip Management
- Booking Management
- Payment Management
- Reporting

Monolithic architecture-এ এই feature-গুলো একই application-এর আলাদা module হিসেবে থাকতে পারে।

### Architecture Diagram

```text
             Client / Browser
                    |
                    v
          ASP.NET Core Application
          +-----------------------+
          | Authentication        |
          | User Management       |
          | Bus Management        |
          | Route Management      |
          | Trip Management       |
          | Booking Management    |
          | Payment Management    |
          | Reporting             |
          +-----------------------+
                    |
                    v
               SQL Server
```

এখানে সব module একই application-এর অংশ। তবে এর অর্থ এই নয় যে সব code একই file-এ থাকবে।

একটি ভালো Monolithic application-এ Controller, Service, Repository, Domain এবং Infrastructure আলাদা project বা layer-এ রাখা যায়।

### কেন Monolithic ব্যবহার করা হয়?

- শুরুতে development সহজ রাখতে।
- দ্রুত MVP বা minimum viable product তৈরি করতে।
- deployment ও debugging সহজ করতে।
- একই database transaction-এর মধ্যে একাধিক business operation সম্পন্ন করতে।
- ছোট বা মাঝারি দলের development overhead কমাতে।

---

# 2. Monolithic কীভাবে কাজ করে?

ধরা যাক, একজন customer bus booking করতে চান।

প্রক্রিয়াটি হতে পারে:

1. Customer frontend থেকে booking request পাঠান।
2. Booking Controller request গ্রহণ করে।
3. Booking Service business rules যাচাই করে।
4. Trip ও Seat availability যাচাই করা হয়।
5. Database-এ booking সংরক্ষণ করা হয়।
6. Response customer-কে পাঠানো হয়।

```text
Customer
   |
   v
BookingController
   |
   v
BookingService
   |
   +------> TripService
   |
   +------> SeatService
   |
   v
SQL Server
   |
   v
Booking Response
```

সব service একই application-এর মধ্যে থাকায়, একটি service থেকে অন্য service-এ সাধারণ method call করা সম্ভব।

উদাহরণ:

```csharp
public class BookingService
{
    private readonly TripService _tripService;
    private readonly SeatService _seatService;

    public BookingService(
        TripService tripService,
        SeatService seatService)
    {
        _tripService = tripService;
        _seatService = seatService;
    }

    public async Task<string> CreateBookingAsync(
        int tripId,
        int seatId)
    {
        var trip = await _tripService.GetTripAsync(tripId);

        if (trip is null)
            throw new InvalidOperationException(
                "Trip not found.");

        var available =
            await _seatService.IsAvailableAsync(seatId);

        if (!available)
            throw new InvalidOperationException(
                "Seat is not available.");

        // Production application-এ এখানে
        // atomic seat reservation এবং database
        // transaction নিশ্চিত করতে হবে।

        return "Booking request processed.";
    }
}
```

**গুরুত্বপূর্ণ:** এটি বোঝার জন্য একটি simplified example। Production booking system-এ একই seat একাধিক customer যেন একসঙ্গে book করতে না পারেন, তার জন্য atomic database operation, concurrency control এবং transaction দরকার।

### Monolithic-এর সুবিধা

- সহজ development workflow।
- Local debugging সহজ।
- Deployment সাধারণত একটি application artifact দিয়ে করা যায়।
- Internal method call হওয়ায় inter-service network latency থাকে না।
- Database transaction তুলনামূলক সহজে পরিচালনা করা যায়।

### Monolithic-এর অসুবিধা

- Application বড় হলে codebase maintain করা কঠিন হতে পারে।
- একটি module-এর পরিবর্তনের জন্য পুরো application redeploy করতে হতে পারে।
- একই application-এর সব feature একসঙ্গে scale করতে হয়।
- Module-গুলোর মধ্যে tight coupling তৈরি হতে পারে।
- বড় codebase build ও test করতে বেশি সময় লাগতে পারে।

**মনে রাখবেন:** ভালো modular design থাকলে Monolithic application-ও দীর্ঘদিন maintainable এবং scalable থাকতে পারে।

---

# 3. Monolithic-এর Practical Example

ধরা যাক, আপনি ASP.NET Core Web API দিয়ে একটি Bus Booking System তৈরি করছেন।

### সম্ভাব্য project structure

```text
BPS/
├── BPS.sln
│
├── BPS.API/
│   ├── Controllers/
│   │   ├── AuthController.cs
│   │   ├── BusController.cs
│   │   ├── TripController.cs
│   │   └── BookingController.cs
│   └── Program.cs
│
├── BPS.Application/
│   ├── Services/
│   └── DTOs/
│
├── BPS.Domain/
│   ├── Entities/
│   └── Interfaces/
│
├── BPS.Infrastructure/
│   ├── Repositories/
│   └── Data/
│
└── BPS.Tests/
```

এখানে আলাদা project থাকলেও application-টি Monolithic হতে পারে।

কারণ architecture নির্ধারিত হয় মূলত deployment boundary ও runtime structure দিয়ে, solution-এ project-এর সংখ্যা দিয়ে নয়।

### একটি সাধারণ API endpoint

```csharp
[ApiController]
[Route("api/bookings")]
public class BookingController : ControllerBase
{
    private readonly BookingService _bookingService;

    public BookingController(
        BookingService bookingService)
    {
        _bookingService = bookingService;
    }

    [HttpPost]
    public async Task<IActionResult> Create(
        [FromBody] CreateBookingRequest request)
    {
        var result = await _bookingService.CreateBookingAsync(
            request.TripId,
            request.SeatId);

        return Ok(new
        {
            message = result
        });
    }
}

public record CreateBookingRequest(
    int TripId,
    int SeatId);
```

এই উদাহরণটি architecture বোঝানোর জন্য। বাস্তব project-এ request validation, authentication, authorization, exception handling এবং concurrency-safe booking logic যোগ করতে হবে।

---

# 4. Microservices Architecture কী?

**Microservices Architecture** হলো এমন একটি architecture যেখানে একটি বড় application-কে একাধিক ছোট, business-capability-কেন্দ্রিক, independently deployable service-এ ভাগ করা হয়।

প্রতিটি service একটি নির্দিষ্ট business responsibility পরিচালনা করে।

Bus Booking System-এর উদাহরণ:

- Auth Service
- User Service
- Bus Service
- Trip Service
- Booking Service
- Payment Service
- Notification Service

### Architecture Diagram

```text
                Client / Browser
                       |
                       v
                  API Gateway
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
    Auth Service   Trip Service   Booking Service
        |              |              |
        v              v              v
    Auth DB          Trip DB        Booking DB
                                       |
                                       v
                                 Payment Service
                                       |
                                       v
                                   Payment DB

                 Notification Service
                          ^
                          |
                     Message Broker
                  (RabbitMQ / Kafka)
```

এই diagram-টি একটি conceptual architecture। বাস্তব system-এ service-to-service communication এবং event flow business requirements অনুযায়ী নির্ধারিত হয়।

### কেন Microservices ব্যবহার করা হয়?

- বিভিন্ন service আলাদাভাবে deploy করার জন্য।
- নির্দিষ্ট service-এর workload অনুযায়ী scaling করার জন্য।
- বড় engineering team-কে স্বাধীনভাবে কাজ করতে দেওয়ার জন্য।
- Business capability অনুযায়ী ownership নির্ধারণের জন্য।
- বিভিন্ন service-এর release cycle আলাদা রাখার জন্য।
- নির্দিষ্ট service-এর technology বা infrastructure প্রয়োজন অনুযায়ী নির্বাচন করার জন্য।

তবে Microservices ব্যবহার করলেই application স্বয়ংক্রিয়ভাবে দ্রুত, নিরাপদ বা সহজে maintainable হয়ে যায় না। Service boundary ভুল হলে complexity আরও বেড়ে যেতে পারে।

---

# 5. Microservices কীভাবে কাজ করে?

ধরা যাক, একজন customer একটি bus ticket book করছেন।

একটি সম্ভাব্য flow:

1. Customer API Gateway-তে booking request পাঠান।
2. Gateway request-টি Booking Service-এ route করে।
3. Booking Service trip ও seat availability যাচাই করে।
4. Booking Service নিজের database-এ booking state সংরক্ষণ করে।
5. Payment Service payment process করে।
6. Payment সফল হলে একটি event প্রকাশ করা হয়।
7. Notification Service event পেয়ে confirmation পাঠায়।

```text
Customer
   |
   v
API Gateway
   |
   v
Booking Service
   |
   +------> Trip Service
   |
   v
Booking Database
   |
   v
Payment Service
   |
   v
Payment Provider
   |
   v
PaymentSucceeded Event
   |
   v
Notification Service
   |
   v
Customer Notification
```

এখানে Payment Service এবং Notification Service আলাদাভাবে deploy করা সম্ভব।

তবে distributed system-এ সব operation একটি সাধারণ database transaction-এর মধ্যে থাকে না। তাই failure handling, retries, duplicate messages, idempotency এবং eventual consistency বিবেচনা করতে হয়।

### Microservices-এর সুবিধা

- Independent deployment।
- Service-specific scaling।
- Business ownership পরিষ্কার করা।
- একটি service-এর release অন্য service-এর release থেকে আলাদা রাখা।
- নির্দিষ্ট workload-এর জন্য আলাদা resource বা technology ব্যবহার করা।

### Microservices-এর অসুবিধা

- Network failure ও latency।
- Distributed tracing এবং monitoring-এর জটিলতা।
- Service-to-service authentication ও authorization।
- Distributed data consistency।
- Retry, timeout, circuit breaker এবং idempotency পরিচালনা।
- Deployment, networking ও infrastructure-এর অতিরিক্ত খরচ।
- Local development এবং integration testing আরও জটিল হতে পারে।

---

# 6. Microservices-এর Practical Example

ASP.NET Core দিয়ে একটি Booking Service-এর simplified example:

```csharp
[ApiController]
[Route("api/bookings")]
public class BookingController : ControllerBase
{
    private readonly BookingService _bookingService;

    public BookingController(
        BookingService bookingService)
    {
        _bookingService = bookingService;
    }

    [HttpPost]
    public async Task<IActionResult> Create(
        [FromBody] CreateBookingRequest request,
        CancellationToken cancellationToken)
    {
        var booking = await _bookingService.CreateAsync(
            request.TripId,
            request.SeatId,
            cancellationToken);

        return Accepted(new
        {
            bookingId = booking.Id,
            status = booking.Status
        });
    }
}
```

এই endpoint একটি Booking Service-এর অংশ হতে পারে।

অন্যদিকে Payment Service আলাদা ASP.NET Core Web API project হিসেবে তৈরি করা হবে:

```text
BPS.BookingService/
├── Controllers/
├── Application/
├── Domain/
├── Infrastructure/
└── Program.cs

BPS.PaymentService/
├── Controllers/
├── Application/
├── Domain/
├── Infrastructure/
└── Program.cs

BPS.NotificationService/
├── Consumers/
├── Application/
└── Program.cs
```

প্রতিটি service-এর নিজস্ব deployment artifact, configuration, health endpoint এবং operational responsibility থাকতে পারে।

### Service communication-এর দুটি ধরন

**1. Synchronous communication**

HTTP/REST বা gRPC ব্যবহার করে একটি service অন্য service থেকে সরাসরি response চায়।

উদাহরণ:

```text
Booking Service
       |
       | HTTP Request
       v
Trip Service
       |
       | HTTP Response
       v
Booking Service
```

ব্যবহার: তাৎক্ষণিকভাবে trip information যাচাই করা।

**2. Asynchronous communication**

RabbitMQ বা Kafka-এর মতো message broker-এর মাধ্যমে event বা message আদান-প্রদান করা হয়।

```text
Booking Service
       |
       | BookingCreated Event
       v
Message Broker
       |
       +------> Notification Service
       |
       +------> Analytics Service
```

ব্যবহার: notification, analytics এবং অন্যান্য asynchronous workflow।

Event-driven communication-এ duplicate delivery, retry এবং message processing-এর consistency নিশ্চিত করতে হয়।

---

# 7. Monolithic বনাম Microservices

| বিষয় | Monolithic | Microservices |
|---|---|---|
| Application structure | একটি deployable application | একাধিক deployable service |
| Development | শুরুতে তুলনামূলক সহজ | শুরুতে বেশি planning দরকার |
| Deployment | সাধারণত একসঙ্গে | Service অনুযায়ী আলাদা |
| Scaling | সাধারণত পুরো application | প্রয়োজনীয় service |
| Communication | Internal method call | HTTP, gRPC, messaging |
| Database | Shared database প্রচলিত | Service-owned data preferred |
| Transaction | তুলনামূলক সহজ | Distributed workflow জটিল |
| Debugging | তুলনামূলক সহজ | Distributed tracing দরকার হতে পারে |
| Failure handling | Process/application-level | Network ও service failure-ও সামলাতে হয় |
| Infrastructure | তুলনামূলক কম | তুলনামূলক বেশি |
| Team structure | ছোট বা মাঝারি team | একাধিক autonomous team-এর জন্য উপযোগী |
| Best fit | MVP, ছোট/মাঝারি application | বড়, জটিল, independently scalable system |

### মূল পার্থক্য

Monolithic-এ module-গুলো একই application deployment-এর অংশ।

Microservices-এ service-গুলো আলাদা deployment boundary এবং সাধারণত নিজস্ব data ownership নিয়ে কাজ করে।

**শুধু আলাদা project বা আলাদা Docker container তৈরি করলেই Microservices Architecture হয় না।** Business boundary, deployment independence এবং data ownership-ও গুরুত্বপূর্ণ।

---

# 8. Database Design-এর পার্থক্য

## Monolithic Database

একটি Monolithic application-এ একাধিক module একই SQL Server database ব্যবহার করতে পারে।

```text
BPS Application
       |
       v
    SQL Server
       |
       +---- Users
       +---- Buses
       +---- Trips
       +---- Bookings
       +---- Payments
```

সুবিধা:

- Foreign key এবং relational constraint ব্যবহার সহজ।
- একাধিক table-এর ওপর transaction পরিচালনা সহজ।
- Reporting এবং cross-module query সহজ হতে পারে।

ঝুঁকি:

- Module-গুলোর মধ্যে database-level coupling বাড়তে পারে।
- Schema পরিবর্তন অনেক module-কে প্রভাবিত করতে পারে।

ভালো practice হলো module অনুযায়ী table ownership ও data access boundary নির্ধারণ করা।

## Microservices Database

Microservices-এ প্রতিটি service তার data-এর owner হওয়া উচিত।

```text
Auth Service       -> Auth DB
Trip Service       -> Trip DB
Booking Service    -> Booking DB
Payment Service    -> Payment DB
```

এর অর্থ সব service-এর জন্য অবশ্যই আলাদা physical database server লাগবে, এমন নয়। শুরুতে একই SQL Server instance-এ আলাদা database বা schema ব্যবহার করা সম্ভব। গুরুত্বপূর্ণ হলো service-এর data access ও ownership boundary বজায় রাখা।

### কেন Database-per-Service?

- একটি service নিজের schema স্বাধীনভাবে পরিবর্তন করতে পারে।
- অন্য service-এর সরাসরি table access এড়ানো যায়।
- Service-specific backup, scaling ও security policy নির্ধারণ সহজ হতে পারে।

### তাহলে cross-service query কীভাবে হবে?

একটি service অন্য service-এর database-এ সরাসরি query না করে API, event বা read-model ব্যবহার করতে পারে।

উদাহরণ:

```text
Booking Service
       |
       | BookingCreated Event
       v
Reporting Read Model
       |
       v
Reporting Dashboard
```

এখানে reporting read model বিভিন্ন service থেকে আসা event ব্যবহার করে তৈরি হতে পারে।

তবে এর ফলে eventual consistency তৈরি হতে পারে—অর্থাৎ মূল service-এ পরিবর্তন হওয়ার সঙ্গে সঙ্গে reporting view আপডেট নাও হতে পারে।

---

# 9. Scaling কীভাবে কাজ করে?

Scaling হলো workload বাড়লে application-এর processing capacity বাড়ানোর প্রক্রিয়া।

## Monolithic Scaling

ধরা যাক, Booking, Reporting এবং Authentication একই application-এ চলছে।

```text
              Load Balancer
               /         \
              v           v
         App Instance 1  App Instance 2
              |           |
              +-----+-----+
                    |
                    v
                SQL Server
```

Application-এর একাধিক instance চালানো যায়। কিন্তু Booking-এর traffic বেশি হলেও সাধারণত পুরো application instance-ই scale করতে হয়।

এটি **Horizontal Scaling**।

একটি server-এর CPU বা RAM বাড়ানো হলো **Vertical Scaling**।

## Microservices Scaling

```text
                 API Gateway
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
      Auth Service  Trip Service  Booking Service
          2 Pods      2 Pods        6 Pods
                                      |
                                      v
                                 Booking DB
```

এখানে Booking Service-এর workload বেশি হলে Booking Service-এর instance বা Pod বাড়ানো যায়।

উপরের সংখ্যা শুধু উদাহরণ; বাস্তবে replica সংখ্যা load test, CPU/memory usage, latency এবং downstream database capacity অনুযায়ী নির্ধারণ করতে হয়।

### গুরুত্বপূর্ণ সতর্কতা

Microservices scale করলেও database bottleneck থাকলে performance বাড়বে না। Connection pool, query performance, locking, indexing এবং database capacity-ও পর্যবেক্ষণ করতে হবে।

---

# 10. Docker ও Kubernetes-এর ভূমিকা

Docker application-কে container image হিসেবে package করতে সাহায্য করে।

Kubernetes containerized application deploy, schedule, restart এবং scaling পরিচালনা করতে পারে।

কিন্তু মনে রাখবেন:

- Docker ব্যবহার করলেই Microservices হয় না।
- Monolithic application-ও Docker-এ চালানো যায়।
- Kubernetes ব্যবহার করতেই হবে, এমন নয়।
- Microservices-এর জন্য managed container platform বা অন্য deployment platform-ও ব্যবহার করা যায়।

### Monolithic Docker Deployment

```text
Docker Host
    |
    v
BPS Application Container
    |
    v
SQL Server
```

### Microservices Docker Deployment

```text
Docker Host / Container Platform
    |
    +---- Auth Container
    |
    +---- Trip Container
    |
    +---- Booking Container
    |
    +---- Payment Container
    |
    +---- Notification Container
```

Production environment-এ database container চালালে persistent storage, backup, recovery, credential management এবং database lifecycle আলাদাভাবে পরিকল্পনা করতে হবে।

Kubernetes-এ production deployment-এর ক্ষেত্রে সাধারণত নিচের বিষয়গুলো বিবেচনা করা হয়:

- Deployment ও Service
- Ingress বা Gateway
- ConfigMap ও Secret
- Resource requests ও limits
- Liveness ও readiness probes
- Horizontal Pod Autoscaler (HPA)
- Persistent storage, যদি প্রয়োজন হয়
- Centralized logs, metrics ও distributed tracing

---

# 11. Security ও Communication

Microservices architecture-এ একাধিক service থাকায় security boundary সঠিকভাবে নির্ধারণ করা জরুরি।

### Authentication বনাম Authorization

- **Authentication:** ব্যবহারকারী কে?
- **Authorization:** ব্যবহারকারী কী করতে পারবেন?

উদাহরণ:

```text
Client
   |
   v
API Gateway
   |
   v
JWT Validation
   |
   v
Booking Service
   |
   v
Permission Check
```

Gateway-তে JWT validate করা উপকারী হতে পারে, তবে sensitive operation-এর ক্ষেত্রে backend service-এও authorization এবং business rule enforce করতে হবে।

### Production Security Checklist

- HTTPS/TLS ব্যবহার করুন।
- JWT signing key নিরাপদে সংরক্ষণ করুন।
- Secret source code বা Docker image-এ রাখবেন না।
- Least-privilege access policy প্রয়োগ করুন।
- Service-to-service identity যাচাই করুন।
- Input validation ও rate limiting ব্যবহার করুন।
- Audit logging এবং security monitoring রাখুন।
- Database-এ least-privilege credentials ব্যবহার করুন।

### Timeout ও Retry

একটি service অন্য service-কে call করলে timeout নির্ধারণ করুন।

Retry ব্যবহার করার আগে নিশ্চিত করুন যে operation পুনরায় চালালে duplicate booking বা duplicate payment তৈরি হবে না।

Payment processing-এর মতো operation-এ idempotency key, durable state এবং reconciliation গুরুত্বপূর্ণ।

---

# 12. Production-Grade Architecture

## Option A: Modular Monolith

```text
             Client
                |
                v
        Reverse Proxy / Gateway
                |
                v
        ASP.NET Core Application
        +-----------------------+
        | Auth Module           |
        | Trip Module           |
        | Booking Module        |
        | Payment Module        |
        | Reporting Module      |
        +-----------------------+
                |
                v
             SQL Server

       Logs + Metrics + Tracing
```

এটি একটি ভালো starting architecture হতে পারে।

একটি application হলেও module boundary পরিষ্কার রাখা হয়, যাতে ভবিষ্যতে প্রয়োজন হলে নির্দিষ্ট module আলাদা service-এ বের করা যায়।

## Option B: Microservices

```text
                    Client
                       |
                       v
                API Gateway
                       |
          +------------+------------+
          |            |            |
          v            v            v
      Auth Service  Trip Service  Booking Service
          |            |            |
          v            v            v
        Auth DB      Trip DB      Booking DB
                                     |
                                     v
                               Payment Service
                                     |
                                     v
                                  Payment DB

        Booking / Payment Events
                   |
                   v
              RabbitMQ
                   |
                   v
          Notification Service

      Centralized Logs + Metrics + Tracing
```

একটি mature production system-এ সাধারণত আরও বিবেচনা করতে হয়:

- Centralized logging
- Metrics এবং alerting
- Distributed tracing
- Health checks
- Secrets management
- API versioning
- Database migration strategy
- Automated testing ও CI/CD
- Backup ও disaster recovery
- Service-level objectives (SLOs)
- Failure isolation ও incident response

### Monitoring Tools

- Prometheus — metrics সংগ্রহ
- Grafana — dashboard ও visualization
- Grafana Loki — log aggregation
- OpenTelemetry — traces, metrics এবং logs instrument করার framework/tooling
- Jaeger বা Tempo — distributed tracing backend হিসেবে ব্যবহার করা যেতে পারে

Microservices-এ observability বিশেষ গুরুত্বপূর্ণ, কারণ একটি request একাধিক service অতিক্রম করতে পারে।

---

# 13. কখন কোন Architecture ব্যবহার করবেন?

## Monolithic নির্বাচন করুন যখন:

- নতুন project শুরু করছেন।
- ছোট team নিয়ে কাজ করছেন।
- Business requirements এখনও পরিবর্তিত হচ্ছে।
- Deployment ও infrastructure সহজ রাখতে চান।
- Traffic ও workload মাঝারি পর্যায়ে আছে।
- Microservices পরিচালনার operational capacity নেই।

## Microservices নির্বাচন করুন যখন:

- Business domain পরিষ্কারভাবে আলাদা করা সম্ভব।
- একাধিক team স্বাধীনভাবে কাজ ও deploy করতে চায়।
- নির্দিষ্ট service-এর workload অন্যগুলোর তুলনায় অনেক বেশি।
- Independent deployment বাস্তব business value দেয়।
- Monitoring, automation, security ও distributed systems পরিচালনার দক্ষতা আছে।
- Service boundary ও data ownership নির্ধারণ করা সম্ভব।

### Practical Decision

| Project | সম্ভাব্য পছন্দ |
|---|---|
| Portfolio project | Modular Monolith |
| Small business application | Monolith |
| Internal management system | Modular Monolith |
| Early-stage startup MVP | Modular Monolith |
| Large booking platform | Requirements অনুযায়ী Monolith বা Microservices |
| Multiple autonomous engineering teams | Microservices উপযোগী হতে পারে |
| Independently scaling payment platform | যথাযথ boundary থাকলে Microservices বিবেচনা করা যায় |

এগুলো সাধারণ নির্দেশনা, কঠোর নিয়ম নয়। Application-এর traffic একাই Microservices নেওয়ার যথেষ্ট কারণ নয়।

**প্রথমে modular Monolith দিয়ে শুরু করা অনেক project-এর জন্য যুক্তিসংগত সিদ্ধান্ত।** যখন স্বাধীন scaling, deployment বা team ownership-এর বাস্তব প্রয়োজন দেখা দেয়, তখন নির্দিষ্ট module আলাদা service-এ নেওয়া যেতে পারে।

---

# 14. Monolith থেকে Microservices-এ Migration

বিদ্যমান application-কে একবারে Microservices-এ রূপান্তর করা সাধারণত ঝুঁকিপূর্ণ।

এর পরিবর্তে ধাপে ধাপে migration করা যেতে পারে।

### Step 1: Module Boundary নির্ধারণ

উদাহরণ:

```text
Existing BPS Monolith
    |
    +---- Authentication
    +---- Trip Management
    +---- Booking
    +---- Payment
    +---- Reporting
```

প্রতিটি module-এর business responsibility ও data ownership নির্ধারণ করুন।

### Step 2: Coupling কমানো

- Shared business logic সীমিত করুন।
- Module-এর মধ্যে explicit interface ব্যবহার করুন।
- অন্য module-এর table সরাসরি access করার প্রবণতা কমান।
- Automated test যোগ করুন।

### Step 3: একটি Module আলাদা করা

উদাহরণ হিসেবে Notification Module আলাদা Notification Service-এ রূপান্তর করা যেতে পারে।

```text
Monolith
    |
    | Notification Event
    v
Message Broker
    |
    v
Notification Service
```

এটি event-driven migration-এর একটি উদাহরণ। শুরুতে সরল HTTP integration দিয়েও migration করা সম্ভব।

### Step 4: Independent Deployment

আলাদা service image তৈরি করুন এবং CI/CD pipeline-এ independent build, test ও deployment যুক্ত করুন।

### Step 5: Monitoring ও Reliability

- Timeout
- Retry policy
- Idempotency
- Distributed tracing
- Health checks
- Alerting
- Failure recovery

এসব ব্যবস্থা নিশ্চিত করুন।

### Step 6: ফলাফল পরিমাপ করুন

Migration-এর আগে ও পরে নিচের metric তুলনা করুন:

- API latency
- Error rate
- Deployment frequency
- Resource utilization
- Infrastructure cost
- Mean time to recovery (MTTR)

পরিমাপযোগ্য business বা operational benefit না থাকলে শুধু আধুনিক architecture ব্যবহারের জন্য migration করা উচিত নয়।

---

# 15. Best Practices

## Monolithic Best Practices

1. Controller, business logic এবং data access আলাদা রাখুন।
2. Domain অনুযায়ী module তৈরি করুন।
3. অপ্রয়োজনীয় shared dependency এড়িয়ে চলুন।
4. Database transaction ও concurrency সঠিকভাবে পরিচালনা করুন।
5. Unit ও integration test লিখুন।
6. Structured logging ও health checks যোগ করুন।
7. CI/CD ও automated deployment ব্যবহার করুন।

## Microservices Best Practices

1. Technical layer নয়, business capability অনুযায়ী service ভাগ করুন।
2. প্রতিটি service-এর data ownership পরিষ্কার রাখুন।
3. Service-to-service communication সীমিত ও উদ্দেশ্যপূর্ণ রাখুন।
4. Timeout, retry এবং idempotency পরিকল্পনা করুন।
5. Distributed tracing ও centralized monitoring ব্যবহার করুন।
6. API contract ও event schema versioning করুন।
7. Backward-compatible deployment নিশ্চিত করুন।
8. Secret management ও least-privilege access প্রয়োগ করুন।
9. Integration ও contract testing করুন।
10. Distributed transaction-এর পরিবর্তে প্রয়োজন অনুযায়ী Saga বা compensating action বিবেচনা করুন।

### সাধারণ ভুল

- প্রতিটি ছোট feature-এর জন্য আলাদা service তৈরি করা।
- একই database-এর সব table সব service-কে ব্যবহার করতে দেওয়া।
- প্রতিটি service-এর মধ্যে অপ্রয়োজনীয় synchronous dependency তৈরি করা।
- Message retry করলেও duplicate processing প্রতিরোধ না করা।
- Monitoring ছাড়া production-এ service deploy করা।
- Business requirement না বুঝে Kubernetes বা Microservices বেছে নেওয়া।

---

# 16. সারসংক্ষেপ

**Monolithic Architecture:** একটি deployable application-এর মধ্যে একাধিক business module রাখা হয়।

**Microservices Architecture:** Business capability অনুযায়ী একাধিক independently deployable service তৈরি করা হয়।

মূল সিদ্ধান্তটি হলো—কোন architecture বেশি আধুনিক, সেটি নয়; বরং আপনার business requirements, team size, deployment needs, workload এবং operational capacity-এর জন্য কোন architecture সবচেয়ে উপযুক্ত।

একজন .NET Developer হিসেবে প্রথমে Modular Monolith, Clean Architecture, SOLID principles, database transaction এবং testing ভালোভাবে শেখা উপকারী। এরপর Microservices, messaging, distributed consistency, Docker, Kubernetes এবং observability শিখলে বড় distributed system design করা সহজ হবে।

---

## 📖 Further Learning

- [Microsoft — Microservices Architecture](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/)
- [Microsoft — Common Web Application Architectures](https://learn.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures)
- [Microsoft — .NET Architecture Guides](https://dotnet.microsoft.com/en-us/learn/dotnet/architecture-guides)
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)

---

**Author:** Joni Miah

**Topics:** Software Architecture, Monolithic Architecture, Microservices, ASP.NET Core, SQL Server, Docker, Kubernetes, Distributed Systems

**Purpose:** Practical learning guide for .NET developers.

⬅️ [Design Principles](05-design-principles.md) | ➡️ [Message Queue ও EDA](07-message-queue-and-eda.md)
