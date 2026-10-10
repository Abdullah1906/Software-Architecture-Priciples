# Chapter ০২: ACID

## ১. ACID কী?

ACID হলো database **transaction**-এর ৪টি গ্যারান্টি, যা data-কে নির্ভরযোগ্য রাখে:

| অক্ষর | নাম | সংক্ষেপে |
|---|---|---|
| **A** | Atomicity | হয় সব হবে, নয় কিছুই না |
| **C** | Consistency | Data সবসময় নিয়ম মেনে বৈধ থাকবে |
| **I** | Isolation | একসাথে চলা transaction একে অপরকে নষ্ট করবে না |
| **D** | Durability | Commit হলে data স্থায়ীভাবে সংরক্ষিত |

> **Transaction** = একাধিক database কাজের একটি যৌথ একক।

---

## ২. কেন দরকার? — একটি বাস্তব গল্প

ব্যাংক transfer: রহিম → করিম ১০০০ টাকা।

```
১. রহিমের account থেকে ১০০০ কাটো
২. করিমের account-এ ১০০০ যোগ করো
```

ধাপ ১ হওয়ার পর server crash করলে? রহিমের টাকা গেল, করিম পেল না। 💥
ACID থাকলে এই অবস্থা **কখনো** হবে না।

---

## ৩. চারটি property বিস্তারিত

### 🅰️ Atomicity (অবিভাজ্যতা)

পুরো transaction একটি একক। মাঝপথে ব্যর্থ হলে সব **ROLLBACK**।

```sql
BEGIN TRANSACTION;
    UPDATE Accounts SET Balance = Balance - 1000 WHERE Id = 1;
    UPDATE Accounts SET Balance = Balance + 1000 WHERE Id = 2;
COMMIT;   -- সব ঠিক থাকলে
-- ROLLBACK;  -- কোনো সমস্যা হলে
```

### 🅲 Consistency (সঙ্গতি)

Transaction-এর আগে ও পরে database সব নিয়ম (constraint) মেনে চলবে।

- `CHECK (Balance >= 0)` — balance ঋণাত্মক হবে না
- `FOREIGN KEY` — না থাকা customer-এর order হবে না
- `UNIQUE` — duplicate email হবে না

```sql
ALTER TABLE Accounts ADD CONSTRAINT CK_Balance CHECK (Balance >= 0);
```
> নিয়ম ভাঙলে transaction আপনা-আপনি fail করবে।

### 🅸 Isolation (বিচ্ছিন্নতা)

একই সময়ে অনেক user কাজ করলেও, প্রত্যেকের মনে হবে সে একাই কাজ করছে।

**Isolation না থাকলে যে সমস্যা:**

| সমস্যা | মানে |
|---|---|
| **Dirty Read** | অন্যের uncommitted data পড়া |
| **Non-repeatable Read** | একই row দুবার পড়লে ভিন্ন মান |
| **Phantom Read** | একই query-তে দ্বিতীয়বার নতুন row এসে যাওয়া |
| **Lost Update** | দুজন একই row update করলে একজনের পরিবর্তন হারিয়ে যাওয়া |

**SQL Server Isolation Level:**

| Level | Dirty | Non-repeatable | Phantom | মন্তব্য |
|---|---|---|---|---|
| READ UNCOMMITTED | ✔ সম্ভব | ✔ | ✔ | সবচেয়ে দ্রুত, ঝুঁকিপূর্ণ |
| **READ COMMITTED** (default) | ✘ | ✔ | ✔ | সাধারণ ব্যবহার |
| REPEATABLE READ | ✘ | ✘ | ✔ | |
| SERIALIZABLE | ✘ | ✘ | ✘ | সবচেয়ে নিরাপদ, ধীর |
| SNAPSHOT | ✘ | ✘ | ✘ | Row versioning, lock কম |

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### 🅳 Durability (স্থায়িত্ব)

`COMMIT` হয়ে গেলে power চলে গেলেও data হারাবে না — কারণ DB আগে **Transaction Log (Write-Ahead Log)**-এ লেখে, পরে data file-এ।

---

## ৪. কীভাবে কাজ করে? (ভেতরের কথা)

- **Atomicity/Durability** → Transaction Log
- **Isolation** → Lock বা Row Versioning (MVCC/Snapshot)
- **Consistency** → Constraint, Trigger, ও application-এর business rule

---

## ৫. ADO.NET দিয়ে Transaction (C#)

```csharp
await using var conn = new SqlConnection(_connectionString);
await conn.OpenAsync();
await using var tx = conn.BeginTransaction();

try
{
    await using var debit = new SqlCommand(
        "UPDATE Accounts SET Balance = Balance - @amt WHERE Id = @from", conn, tx);
    debit.Parameters.AddWithValue("@amt", 1000m);
    debit.Parameters.AddWithValue("@from", 1);
    await debit.ExecuteNonQueryAsync();

    await using var credit = new SqlCommand(
        "UPDATE Accounts SET Balance = Balance + @amt WHERE Id = @to", conn, tx);
    credit.Parameters.AddWithValue("@amt", 1000m);
    credit.Parameters.AddWithValue("@to", 2);
    await credit.ExecuteNonQueryAsync();

    await tx.CommitAsync();
}
catch
{
    await tx.RollbackAsync();
    throw;
}
```

### Stored Procedure-এ (TRY/CATCH)

```sql
CREATE PROCEDURE usp_TransferMoney
    @FromId INT, @ToId INT, @Amount DECIMAL(18,2)
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON;   -- error হলে auto rollback

    BEGIN TRY
        BEGIN TRANSACTION;

        UPDATE Accounts SET Balance = Balance - @Amount WHERE Id = @FromId;
        UPDATE Accounts SET Balance = Balance + @Amount WHERE Id = @ToId;

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0 ROLLBACK TRANSACTION;
        THROW;
    END CATCH
END
```

### Entity Framework Core-এ
`SaveChanges()` নিজেই একটি transaction। একাধিক `SaveChanges` একসাথে করতে:

```csharp
await using var tx = await _db.Database.BeginTransactionAsync();
// ... একাধিক কাজ
await _db.SaveChangesAsync();
await tx.CommitAsync();
```

---

## ৬. কোন পরিস্থিতিতে ব্যবহার হয়?

| পরিস্থিতি | কেন ACID লাগে |
|---|---|
| ব্যাংক/Payment | টাকা হারানো বা দ্বিগুণ হওয়া চলবে না |
| Order + Stock update | Order হয়েছে কিন্তু stock কমেনি — চলবে না |
| Lab Order (QISSLAB) | Order header ও item একসাথে save হতে হবে |
| Seat Booking | দুজনকে একই seat দেওয়া যাবে না |
| Inventory | Stock negative হওয়া ঠেকাতে |

---

## ৭. Professional-grade Practices

### ক) Transaction ছোট রাখুন
Transaction-এর ভেতরে **HTTP call, email, file upload** করবেন না — lock অনেকক্ষণ ধরে থাকবে।

### খ) Concurrency — Optimistic Locking (RowVersion)

দুজন একই record edit করলে (Lost Update ঠেকাতে):

```csharp
public class Product
{
    public int Id { get; set; }
    public int Stock { get; set; }

    [Timestamp]
    public byte[] RowVersion { get; set; }
}
```
```csharp
try { await _db.SaveChangesAsync(); }
catch (DbUpdateConcurrencyException)
{
    // 409 Conflict — "অন্য কেউ আগেই পরিবর্তন করেছে"
}
```

### গ) Pessimistic Locking (Seat booking-এর মতো ক্ষেত্রে)
```sql
SELECT * FROM Seats WITH (UPDLOCK, ROWLOCK) WHERE Id = @SeatId;
```

### ঘ) Deadlock সামলানো
- সব জায়গায় table **একই ক্রমে** access করুন
- Deadlock-এ **retry** logic রাখুন (Polly)

### ঙ) ডিফল্ট `READ COMMITTED`, প্রয়োজনে `SNAPSHOT`
`SERIALIZABLE` শুধু বিশেষ ক্ষেত্রে — এটি performance কমায়।

### চ) Idempotency
Retry হলে যেন দুবার order না হয় — unique key দিয়ে ঠেকান।

---

## ৮. ACID বনাম BASE (সংক্ষেপে)

| | ACID (SQL Server, PostgreSQL) | BASE (অনেক NoSQL) |
|---|---|---|
| Consistency | তাৎক্ষণিক (Strong) | অবশেষে (Eventual) |
| উপযোগী | ব্যাংক, order, inventory | Social feed, analytics, cache |
| Scale | সাধারণত vertical | সহজে horizontal |

Microservices-এ একাধিক database জুড়ে ACID পাওয়া কঠিন — তখন **Saga pattern** ব্যবহার হয়।

---

## ৯. সাধারণ ভুল

| ভুল | ফল |
|---|---|
| Transaction ছাড়া একাধিক write | Data অর্ধেক save হয় |
| Transaction অনেক বড় | Lock/Deadlock/Slow |
| `NOLOCK` সবখানে | Dirty read, ভুল report |
| Catch-এ rollback না করা | Lock আটকে থাকে |
| Concurrency ভাবা হয়নি | Lost update, overselling |

---

## ✅ সারসংক্ষেপ

- **A** — সব নয় কিছুই না | **C** — নিয়ম ভাঙবে না | **I** — একে অপরকে বিরক্ত করবে না | **D** — commit মানে স্থায়ী
- Transaction ছোট রাখুন, concurrency ভাবুন, deadlock-এ retry করুন

⬅️ আগের chapter: [REST API](01-rest-api.md) | ➡️ পরের: [DDD](03-ddd.md)
