# Chapter ০৪: REST + ACID + DDD — একসাথে Professional Architecture

আগের তিন chapter-এ আলাদাভাবে শিখেছি। এখন দেখি একটি real feature-এ তিনটি কীভাবে একসাথে কাজ করে।

## ১. তিনটির ভূমিকা এক নজরে

| ধারণা | কোন প্রশ্নের উত্তর দেয় | কোন layer-এ |
|---|---|---|
| **REST** | Client কীভাবে server-কে বলবে? | API / Presentation |
| **DDD** | Business rule কোথায় ও কীভাবে থাকবে? | Domain |
| **ACID** | Data কীভাবে নিরাপদে সংরক্ষিত হবে? | Infrastructure / Database |

---

## ২. পরিস্থিতি: "Lab Order Place করা"

**Business rule:**
- Order-এ কমপক্ষে ১টি test থাকতে হবে
- Order place হলে sample collection request তৈরি হবে
- Order ও item একসাথে save হতে হবে (আংশিক save চলবে না)

---

## ৩. পুরো Flow

```
Angular/Postman
     │  POST /api/v1/orders          ← REST
     ▼
OrdersController
     │  PlaceOrderCommand
     ▼
Application Service (Handler)         ← Use case, Transaction সীমানা
     │  order.Place()                 ← DDD: rule Domain-এ
     ▼
Order (Aggregate Root)
     │
     ▼
Repository + UnitOfWork.Commit()      ← ACID: এক transaction-এ save
     ▼
SQL Server
```

---

## ৪. কোড

### ১) REST — Controller (পাতলা রাখুন)

```csharp
[ApiController]
[Route("api/v1/orders")]
public class OrdersController : ControllerBase
{
    private readonly IPlaceOrderHandler _handler;
    public OrdersController(IPlaceOrderHandler handler) => _handler = handler;

    [HttpPost]
    public async Task<IActionResult> Place(PlaceOrderRequest request)
    {
        var orderId = await _handler.HandleAsync(request.ToCommand());
        return CreatedAtAction(nameof(Get), new { id = orderId }, new { id = orderId });
    }

    [HttpGet("{id:int}")]
    public Task<IActionResult> Get(int id) => throw new NotImplementedException();
}
```

### ২) Application Layer — Use case ও Transaction

```csharp
public class PlaceOrderHandler : IPlaceOrderHandler
{
    private readonly IOrderRepository _orders;
    private readonly IUnitOfWork _uow;

    public PlaceOrderHandler(IOrderRepository orders, IUnitOfWork uow)
        => (_orders, _uow) = (orders, uow);

    public async Task<int> HandleAsync(PlaceOrderCommand cmd)
    {
        var order = new Order(cmd.CustomerId);

        foreach (var item in cmd.Items)
            order.AddItem(item.TestId, new Money(item.Price, "BDT"), item.Qty);

        order.Place();                       // DDD: business rule এখানে চলে

        await _orders.AddAsync(order);
        await _uow.CommitAsync();            // ACID: সব একসাথে save
        return order.Id;
    }
}
```

### ৩) Infrastructure — UnitOfWork (EF Core)

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _db;
    public UnitOfWork(AppDbContext db) => _db = db;

    // SaveChanges নিজেই একটি transaction → Order + Items একসাথে বা কিছুই না
    public Task<int> CommitAsync() => _db.SaveChangesAsync();
}
```

### ৪) Domain Exception → সঠিক HTTP Status (Global Handler)

```csharp
app.UseExceptionHandler(a => a.Run(async ctx =>
{
    var ex = ctx.Features.Get<IExceptionHandlerFeature>()?.Error;

    var (status, title) = ex switch
    {
        DomainException        => (422, "Business rule violation"),
        DbUpdateConcurrencyException => (409, "Conflict"),
        _                      => (500, "Server error")
    };

    ctx.Response.StatusCode = status;
    await ctx.Response.WriteAsJsonAsync(new ProblemDetails
    {
        Status = status, Title = title, Detail = ex?.Message
    });
}));
```

এখানে **DDD-র rule ভাঙলে → REST-এ 422** ফিরে যায়।

---

## ৫. Project Folder Structure (প্রস্তাবিত)

```
src/
├── MyApp.Domain/            ← Entity, VO, Aggregate, Domain Event, IRepository
├── MyApp.Application/       ← Command, Handler, DTO, Validator
├── MyApp.Infrastructure/    ← EF Core, Repository impl, UnitOfWork, Email
└── MyApp.Api/               ← Controller, Middleware, Program.cs
tests/
├── MyApp.Domain.Tests/      ← DB ছাড়াই unit test
└── MyApp.Api.Tests/         ← Integration test
```

---

## ৬. Professional Checklist ✅

**REST**
- [ ] Noun-based URL, সঠিক HTTP method ও status code
- [ ] DTO, pagination, versioning, ProblemDetails
- [ ] JWT auth, rate limiting, Swagger

**DDD**
- [ ] Business rule Entity/Aggregate-এর ভেতরে
- [ ] Value Object, private setter, ছোট Aggregate
- [ ] Domain layer framework-মুক্ত

**ACID**
- [ ] এক use case = এক transaction
- [ ] Transaction ছোট; ভেতরে external call নেই
- [ ] RowVersion/Concurrency token, Deadlock retry
- [ ] DB constraint (FK, UNIQUE, CHECK) রাখা

---

## ৭. আরও এগোতে চাইলে (Next Steps)

| বিষয় | কেন শিখবেন |
|---|---|
| **CQRS** | Read/Write আলাদা করে performance ও clarity |
| **Outbox Pattern** | DB save ও event publish নিরাপদে একসাথে |
| **Saga Pattern** | একাধিক service জুড়ে transaction |
| **Event-Driven Architecture** | Service গুলোকে loosely coupled করা |
| **Clean Architecture** | Layer ও dependency rule গভীরভাবে |
| **Testing (xUnit, Testcontainers)** | Domain ও integration test |

---

## ✅ শেষ কথা

> **REST** দরজা, **DDD** ঘরের নিয়ম, **ACID** ঘরের নিরাপত্তা।

তিনটি মিলেই একটি **নির্ভরযোগ্য, বোঝা-সহজ ও বড় হওয়ার উপযোগী** backend তৈরি হয়।

⬅️ আগের chapter: [DDD](03-ddd.md) | 🏠 [README](README.md)
