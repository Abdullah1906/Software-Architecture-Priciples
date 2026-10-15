# Chapter ০৮: CQRS

## ১. CQRS কী?

**C**ommand **Q**uery **R**esponsibility **S**egregation — **লেখা (Command) ও পড়া (Query)-র দায়িত্ব আলাদা করা**।

| | Command | Query |
|---|---|---|
| কাজ | অবস্থা **বদলায়** | শুধু **পড়ে** |
| ফেরত দেয় | কিছু না / Id | Data (DTO) |
| উদাহরণ | `PlaceOrder`, `CancelOrder` | `GetOrderById`, `SearchOrders` |
| Side effect | আছে | **নেই** |

---

## ২. কেন দরকার?

বাস্তবে **Write ও Read-এর চাহিদা ভিন্ন**:

- **Write:** জটিল business rule, validation, transaction (→ DDD, Entity)
- **Read:** দ্রুত, flat data, join, filter, report, pagination

একই model দিয়ে দুটোই করলে:
- Entity-তে read-এর জন্য অকারণ property
- Query ধীর (Entity load করে mapping)
- Read ও Write আলাদাভাবে scale করা যায় না

---

## ৩. কীভাবে কাজ করে?

### স্তর ১: সরল CQRS (একই DB)

```
          ┌─ Command ─▶ Handler ─▶ Domain/EF Core ─┐
Client ───┤                                         ├─▶ একটি Database
          └─ Query ───▶ Handler ─▶ Dapper/SQL ─────┘
```
- Write: EF Core + Domain model
- Read: Dapper/raw SQL → সরাসরি DTO

### স্তর ২: আলাদা Read DB (উন্নত)

```
Command ─▶ Write DB (SQL Server)
              │  Event / Sync
              ▼
Query ──▶ Read DB (Elasticsearch / Redis / Read replica)
```
এখানে Read DB **eventually consistent** — কয়েক মিলিসেকেন্ড পিছিয়ে থাকতে পারে।

---

## ৪. কোড উদাহরণ (MediatR)

```csharp
// ---------- Command ----------
public record PlaceOrderCommand(int CustomerId, List<ItemDto> Items) : IRequest<int>;

public class PlaceOrderHandler : IRequestHandler<PlaceOrderCommand, int>
{
    private readonly IOrderRepository _repo;
    private readonly IUnitOfWork _uow;
    public PlaceOrderHandler(IOrderRepository repo, IUnitOfWork uow) => (_repo, _uow) = (repo, uow);

    public async Task<int> Handle(PlaceOrderCommand cmd, CancellationToken ct)
    {
        var order = new Order(cmd.CustomerId);
        foreach (var i in cmd.Items) order.AddItem(i.TestId, new Money(i.Price, "BDT"), i.Qty);
        order.Place();
        await _repo.AddAsync(order);
        await _uow.CommitAsync();
        return order.Id;
    }
}

// ---------- Query ----------
public record GetOrdersQuery(int Page, int PageSize) : IRequest<List<OrderListDto>>;

public class GetOrdersHandler : IRequestHandler<GetOrdersQuery, List<OrderListDto>>
{
    private readonly IDbConnection _db; // Dapper
    public GetOrdersHandler(IDbConnection db) => _db = db;

    public async Task<List<OrderListDto>> Handle(GetOrdersQuery q, CancellationToken ct)
    {
        const string sql = @"
            SELECT o.Id, c.Name AS CustomerName, o.Total, o.Status
            FROM Orders o JOIN Customers c ON c.Id = o.CustomerId
            ORDER BY o.Id DESC
            OFFSET @Skip ROWS FETCH NEXT @Take ROWS ONLY";
        var rows = await _db.QueryAsync<OrderListDto>(sql,
            new { Skip = (q.Page - 1) * q.PageSize, Take = q.PageSize });
        return rows.ToList();
    }
}

// ---------- Controller ----------
[HttpPost] public async Task<IActionResult> Place(PlaceOrderCommand c) => Ok(await _mediator.Send(c));
[HttpGet]  public async Task<IActionResult> List([FromQuery] GetOrdersQuery q) => Ok(await _mediator.Send(q));
```

---

## ৫. কোন পরিস্থিতিতে ব্যবহার করবেন?

| ✅ উপযোগী | ❌ অপ্রয়োজনীয় |
|---|---|
| Read ও Write-এর ভার অনেক আলাদা | সাধারণ CRUD |
| জটিল domain (DDD-র সাথে) | ছোট প্রকল্প |
| Read-heavy system, দ্রুত report দরকার | টিম নতুন, বাড়তি জটিলতা সামলাতে অক্ষম |
| Event Sourcing ব্যবহার করছেন | Strong consistency সবসময় দরকার |

> 💡 **সরল CQRS** (একই DB, আলাদা handler) প্রায় বিনা খরচে পাওয়া যায় — এখান থেকে শুরু করুন। আলাদা Read DB শুধু প্রয়োজন হলে।

---

## ৬. Professional Practices

1. Query **কখনো state বদলাবে না**
2. Read-এ Entity নয়, সরাসরি **DTO/projection** (`AsNoTracking()` বা Dapper)
3. Command-এ **validation pipeline** (FluentValidation + MediatR behavior)
4. Command ছোট ও **একটি উদ্দেশ্যে** (`ChangeOrderAddress`, `UpdateOrder` নয়)
5. আলাদা Read DB হলে **eventual consistency** UI-তে সামলান ("প্রক্রিয়াধীন...")
6. Event + **Outbox** দিয়ে Read model আপডেট

```csharp
// Validation pipeline
public class ValidationBehavior<TReq, TRes>(IEnumerable<IValidator<TReq>> validators)
    : IPipelineBehavior<TReq, TRes> where TReq : notnull
{
    public async Task<TRes> Handle(TReq req, RequestHandlerDelegate<TRes> next, CancellationToken ct)
    {
        var failures = validators.Select(v => v.Validate(req))
                                 .SelectMany(r => r.Errors).Where(f => f != null).ToList();
        if (failures.Count != 0) throw new ValidationException(failures);
        return await next();
    }
}
```

---

## ৭. সাধারণ ভুল

| ভুল | ব্যাখ্যা |
|---|---|
| সব জায়গায় CQRS | সাধারণ CRUD-এ over-engineering |
| Query-তে data বদলানো | মূল নীতি ভাঙা |
| আলাদা Read DB শুরুতেই | অকারণ জটিলতা ও sync সমস্যা |
| Command থেকে বিশাল data ফেরত | Query-র কাজ Command-এ মেশানো |

---

## ✅ সারসংক্ষেপ

- Command = বদলায়, Query = শুধু পড়ে
- সরল CQRS দিয়ে শুরু; প্রয়োজনে আলাদা Read DB
- DDD ও Event-Driven-এর সাথে চমৎকার মেলে

⬅️ [Message Queue ও EDA](07-message-queue-and-eda.md) | ➡️ [CAP Theorem](09-cap-theorem.md)
