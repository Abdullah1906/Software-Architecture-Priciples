# Chapter ১১: Caching && Read Replica

---

## 📌 সূচিপত্র (Table of Contents)

1. [Caching কী?](#1-caching-কী-ki)
2. [Caching কেন লাগে?](#2-caching-কেন-লাগে-keno)
3. [Cache-এর প্রকারভেদ](#3-cache-এর-প্রকারভেদ)
4. [Caching Pattern](#4-caching-patterns)
5. [Read Replica কী?](#5-read-replica-কী-ki)
6. [Read Replica কেন লাগে?](#6-read-replica-কেন-লাগে-keno)
7. [SQL Server-এ Read Replica বানানোর ৩টি উপায়](#7-sql-server-এ-read-replica-বানানোর-৩টি-উপায়)
8. [SSMS দিয়ে Read Replica সেটআপ (Step by Step)](#8-ssms-দিয়ে-read-replica-সেটআপ-step-by-step)
9. [ADO.NET দিয়ে Read Replica Routing](#9-ado.net-দিয়ে-read-replica-routing)
10. [Professional Grade Architecture](#10-professional-grade-architecture)
11. [Best Practices & Common Mistakes](#11-best-practices--common-mistakes)
12. [Glossary](#12-glossary)

---

## 1. Caching কী? (Ki)

**Caching** মানে হলো — বারবার প্রয়োজন হওয়া data-কে মূল Database (slow disk storage)-তে
না গিয়ে, দ্রুত access করা যায় এমন জায়গায় (RAM) আগে থেকে রেখে দেওয়া।

```
মনে করুন একটি E-commerce সাইটের হোমপেজে "Category List" দেখায়।
এই list প্রতি ৫ মিনিটেও বদলায় না, অথচ প্রতিদিন ১০ লাখ মানুষ হোমপেজ দেখে।

❌ Caching ছাড়া: ১০ লাখ বার Database hit = Database slow, খরচ বেশি
✅ Caching দিয়ে: ১ বার Database থেকে নিয়ে RAM-এ রাখা = ৯,৯৯,৯৯৯ বার RAM থেকে serve
```

**সহজ ভাষায়:** দোকানের সবচেয়ে বিক্রি হওয়া পণ্যটা সবসময় ক্যাশ কাউন্টারের পাশে রাখা —
যাতে গুদামে (Database) না গিয়ে সঙ্গে সঙ্গে দেওয়া যায়। 😄

---

## 2. Caching কেন লাগে? (Keno)

| সমস্যা | Caching কীভাবে সমাধান করে |
|---|---|
| Database CPU/IO overload | ৯০%+ read request cache-এ serve হয় |
| Slow response time | RAM read ≈ nanoseconds, Disk read ≈ milliseconds |
| Scalability কম | এক DB-র চাপ অনেক cache server-এ ভাগ হয় |
| Cloud DB cost বেশি | প্রতি query = টাকা, cache কম query = কম টাকা |
| Report/Analytics জটিল query | Heavy query-র result cache-এ রাখা যায় |

**Performance তুলনা:**

```
RAM Cache    : ~ 0.0001 ms  (লাইন লাগে না, সরাসরি দেয়)
SSD Disk     : ~ 0.1–1 ms
Network DB   : ~ 1–10 ms (LAN), ~ 50–200 ms (WAN)
```

> ⚖️ **Trade-off সচেতন থাকুন:** Cache-এর data হালনাগাদ নাও হতে পারে (staleness)।
> তাই **কোন data cache করবেন** (প্রায়-স্থির data) আর কোনটা করবেন না
> (real-time bank balance) — সেটা architectural decision।

---

## 3. Cache-এর প্রকারভেদ

### 3.1 In-Memory Cache (অ্যাপ্লিকেশনের ভেতরে)

অ্যাপ্লিকেশনের RAM-এ cache। দ্রুততম, কিন্তু **multi-server হলে কাজ করে না** —
প্রতিটা server-এর আলাদা cache থাকে।

```csharp
// Program.cs (.NET 6+)
builder.Services.AddMemoryCache();
```

```csharp
// ADO.NET + In-Memory Cache example
using Microsoft.Extensions.Caching.Memory;
using Microsoft.Data.SqlClient;

public class CategoryRepository
{
    private readonly IMemoryCache _cache;
    private readonly string _connStr;

    // Cache key fixed rakhi, sliding expiration 5 min
    private const string CacheKey = "categories:all";
    private static readonly TimeSpan CacheDuration = TimeSpan.FromMinutes(5);

    public CategoryRepository(IMemoryCache cache, IConfiguration config)
    {
        _cache = cache;
        _connStr = config.GetConnectionString("PrimaryDB")!;
    }

    public async Task<List<string>> GetCategoriesAsync()
    {
        // ১. Cache-এ আছে কি না দেখি (Fast path)
        if (_cache.TryGetValue(CacheKey, out List<string>? cached))
            return cached!;   // 🎉 Database-এ যাওয়াই লাগলো না

        // ২. Cache miss → Database-এ যাই (ADO.NET)
        var categories = new List<string>();
        await using var conn = new SqlConnection(_connStr);
        await conn.OpenAsync();

        await using var cmd = new SqlCommand(
            "SELECT CategoryName FROM dbo.Categories WHERE IsActive = 1", conn);
        await using var reader = await cmd.ExecuteReaderAsync();

        while (await reader.ReadAsync())
            categories.Add(reader.GetString(0));

        // ৩. Database থেকে নেওয়া data cache-এ রাখি
        var options = new MemoryCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = CacheDuration,
            SlidingExpiration = TimeSpan.FromMinutes(2)
        };
        _cache.Set(CacheKey, categories, options);

        return categories;
    }

    // ⚠️ Data update করলে cache invalidate (ছিঁড়ে ফেলা) করতে হবে!
    public async Task UpdateCategoryAsync(int id, string newName)
    {
        await using var conn = new SqlConnection(_connStr);
        await conn.OpenAsync();

        await using var cmd = new SqlCommand(
            "UPDATE dbo.Categories SET CategoryName = @name WHERE Id = @id", conn);
        cmd.Parameters.AddWithValue("@name", newName);
        cmd.Parameters.AddWithValue("@id", id);
        await cmd.ExecuteNonQueryAsync();

        // গুরুত্বপূর্ণ: পুরনো cache মুছে দাও, নাহলে ৫ মিনিট পর্যন্ত ভুল data দেখাবে
        _cache.Remove(CacheKey);
    }
}
```

### 3.2 Distributed Cache (Redis / SQL Server Cache)

Multi-server / load-balanced environment-এ **সব server একই cache** ব্যবহার করে।
Redis হলো industry standard।

```csharp
// Program.cs — Redis setup
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "my-redis-server:6379";
    options.InstanceName = "MyShop_";
});
```

```csharp
// ADO.NET + IDistributedCache example
using Microsoft.Extensions.Caching.Distributed;
using Microsoft.Data.SqlClient;
using System.Text.Json;

public class ProductRepository
{
    private readonly IDistributedCache _cache;
    private readonly string _connStr;

    public ProductRepository(IDistributedCache cache, IConfiguration config)
    {
        _cache = cache;
        _connStr = config.GetConnectionString("PrimaryDB")!;
    }

    public async Task<Product?> GetProductAsync(int id)
    {
        string cacheKey = $"product:{id}";

        // ১. Redis-এ আছে কি?
        var cachedBytes = await _cache.GetAsync(cacheKey);
        if (cachedBytes is not null)
            return JsonSerializer.Deserialize<Product>(cachedBytes); // 🎉 Cache hit

        // ২. Cache miss → ADO.NET দিয়ে SQL Server-এ যাই
        await using var conn = new SqlConnection(_connStr);
        await conn.OpenAsync();

        await using var cmd = new SqlCommand(
            "SELECT Id, Name, Price FROM dbo.Products WHERE Id = @id", conn);
        cmd.Parameters.AddWithValue("@id", id);

        await using var reader = await cmd.ExecuteReaderAsync();
        if (!await reader.ReadAsync()) return null;

        var product = new Product(
            reader.GetInt32(0), reader.GetString(1), reader.GetDecimal(2));

        // ৩. Redis-এ serialize করে রাখি (10 মিনিট TTL)
        var options = new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
        };
        await _cache.SetAsync(cacheKey,
            JsonSerializer.SerializeToUtf8Bytes(product), options);

        return product;
    }
}

public record Product(int Id, string Name, decimal Price);
```

### 3.3 In-Memory vs Distributed — কোনটা কখন?

| বিষয় | In-Memory | Distributed (Redis) |
|---|---|---|
| গতি | সবচেয়ে দ্রুত | দ্রুত (network hop আছে) |
| Multi-server support | ❌ | ✅ |
| Data share | ❌ প্রতি server আলাদা | ✅ সবাই একই cache |
| Persistency | App restart-এ হারায় | Redis-এ survive করে |
| Setup difficulty | খুব সহজ | Redis server লাগে |
| **কখন ব্যবহার** | ছোট single-server অ্যাপ | Production, load-balanced, cloud |

---

## 4. Caching Patterns

### 4.1 Cache-Aside (Lazy Loading) — সবচেয়ে জনপ্রিয়

```
Request → Cache-এ আছে?
              ├── হ্যাঁ  → Cache থেকে return ✅
              └── না    → Database → Cache-এ রাখো → Return
```

> ✔️ সহজ, কেবল প্রয়োজনীয় data-ই cache হয়
> ❌ প্রথম request-এ সবসময় cache miss (cold start) → **Cache Stampede** ঠেকাতে
>   `lock` বা Redis-এ lock ব্যবহার করুন

```csharp
// Cache Stampede রোধ — একসাথে অনেক request database-এ না যায়
private static readonly SemaphoreSlim _lock = new(1, 1);

public async Task<Product?> GetProductSafeAsync(int id)
{
    string key = $"product:{id}";
    var cached = await _cache.GetAsync(key);
    if (cached is not null)
        return JsonSerializer.Deserialize<Product>(cached);

    await _lock.WaitAsync();           // 👈 একজনই database-এ যাবে
    try
    {
        cached = await _cache.GetAsync(key);   // দ্বিতীয়বার check (double-check)
        if (cached is not null)
            return JsonSerializer.Deserialize<Product>(cached);

        var product = await LoadFromDatabaseAsync(id);   // ADO.NET query
        if (product is not null)
            await _cache.SetAsync(key,
                JsonSerializer.SerializeToUtf8Bytes(product),
                new DistributedCacheEntryOptions
                { AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10) });
        return product;
    }
    finally
    {
        _lock.Release();
    }
}
```

### 4.2 Read-Through / Write-Through

Cache-ই database-র proxy হিসেবে কাজ করে (Redis Enterprise / custom layer লাগে)।

### 4.3 Write-Behind (Write-Back)

প্রথমে cache-এ লেখে, পরে background-এ database-এ লেখে — write খুব দ্রুত হয়,
কিন্তু crash হলে data হারানোর ঝুঁকি। (Logging, analytics-এ ব্যবহার হয়)

### 4.4 Cache Invalidation Strategies

| Strategy | ব্যাখ্যা |
|---|---|
| **TTL (Time To Live)** | নির্দিষ্ট সময় পর cache নিজে মরে যায়। সহজ, কিন্তু fixed delay-এ ভুল data |
| **Write Invalidate** | Data update হলেই cache মুছে দাও (উপরের example-এ দেখানো) |
| **Event-Based** | SQL Server trigger / message queue (RabbitMQ) দিয়ে সব cache server-কে জানানো |
| **Versioned Key** | Key-এর সাথে version বসানো: `product:v3:101` — schema বদলালে পুরনো key অচল |

> 🏛️ *"There are only two hard things in Computer Science: cache invalidation,
> naming things, and off-by-one errors."* — ঠিক এই কারণেই invalidation strategy আগে ঠিক করুন।

---

## 5. Read Replica কী? (Ki)

**Read Replica** হলো মূল Database-র (**Primary/Master**) একটি **read-only কপি**,
যেটা প্রধানত শুধু `SELECT` query serve করে।

```
                 ┌──────────────────┐
   Write (INSERT/│   PRIMARY DB      │
   UPDATE/DELETE)│  (Read + Write)   │
        ┌───────▶│  SQL-SRV-01       │──────────────┐
        │        └──────────────────┘              │
        │                   │ Log Shipping /       │ Writes
        │                   │ Replication / AG     │
        │                   ▼                      │
        │        ┌──────────────────┐              ▼
   Reports/      │  READ REPLICA     │        ┌──────────┐
   Analytics /   │  (Read-Only)      │        │  USERS   │
   Dashboards ──▶│  SQL-SRV-02       │        └──────────┘
                 └──────────────────┘
```

**সহজ ভাষায়:** একটা বইয়ের মূল কপি লাইব্রেরিতে (Primary), আর ফটোকপি পাশের রুমে (Replica) —
সবাই মূল কপি না নিয়ে ফটোকপি থেকে পড়ে, তাই মূল কপি সবসময় ফাঁকা থাকে। 😄

---

## 6. Read Replica কেন লাগে? (Keno)

| সমস্যা | Replica সমাধান করে |
|---|---|
| Report/Analytics এর ভারী query প্রধান DB-কে ধীর করে | ভারী read replica-তে চলে |
| Read:Write ratio বেশি (90:10) হলে Primary overload | Read load ভাগ হয়ে যায় |
| Disaster Recovery | Primary crash হলে Replica-কে promote করা যায় |
| Geo-distribution | ভিন্ন region-এ replica → কাছের user-এর জন্য কম latency |
| Backup নেওয়ার সময় performance impact | Replica থেকে backup নেওয়া যায় |

> ⚠️ **সতর্কতা:**
> - Replica-র data সামান্য পুরনো হতে পারে (**Replication Lag** — সাধারণত মিলিসেকেন্ড~সেকেন্ড)
> - তাই `bank balance` মতো real-time data replica থেকে পড়া ঝুঁকিপূর্ণ
> - এজন্যই **Caching + Read Replica একসাথে** কাজ করে (নিচে দেখুন)

---

## 7. SQL Server-এ Read Replica বানানোর ৩টি উপায়

| পদ্ধতি | কখন ব্যবহার | Complexity |
|---|---|---|
| **1. Log Shipping** | পুরনো/simple সেটআপ, DR-focused, replica অনেকটা read-only | সহজ |
| **2. Transactional Replication** | নির্দিষ্ট table/db subset replicate, lowest latency | মাঝারি |
| **3. Always On Availability Group (AG)** ⭐ | **Enterprise production standard**, automatic failover, readable secondary | জটিল (Enterprise Edition) |

> ✅ **Professional grade-এ Always On AG-ই standard।** নিচে AG-সহ সবগুলো দেখানো হলো।

---

## 8. SSMS দিয়ে Read Replica সেটআপ (Step by Step)

### পদ্ধতি ১: Log Shipping (সহজ, Standard Edition-এও চলে)

**প্রয়োজন:** Primary server `SQL-SRV-01`, Secondary `SQL-SRV-02`, share folder `\\SQL-SRV-01\LogShipping$`

**Step 1 — Backup Share তৈরি (Primary server-এ):**
```powershell
# Primary server-এ চালান
New-Item -Path "C:\LogShipping" -ItemType Directory
New-SmbShare -Name "LogShipping$" -Path "C:\LogShipping" `
    -FullAccess "SQLSERVICE_ACCOUNT"
```

**Step 2 — Recovery Model ঠিক করুন (SSMS):**
```sql
-- SSMS → Primary DB → Right click → Properties → Options
ALTER DATABASE MyShopDB SET RECOVERY FULL;
```

**Step 3 — Full Backup নিন:**
```sql
-- SSMS-এ query window-এ
BACKUP DATABASE MyShopDB
TO DISK = 'C:\LogShipping\MyShopDB_Full.bak'
WITH INIT, COMPRESSION;
```

**Step 4 — Log Shipping চালু করুন:**
```
SSMS → Databases → MyShopDB → Right Click
  → Tasks → Ship Transaction Logs...

☑ Enable this as a primary database in a log shipping configuration

Backup Settings:
  ├── Network path to backup folder: \\SQL-SRV-01\LogShipping$
  ├── Local path: C:\LogShipping
  └── Backup job schedule: Every 1 minute

Secondary Database (Add):
  ├── Connect to SQL-SRV-02
  ├── Initialize: Yes, generate full backup (No recovery mode) ✅
  ├── Copy files: Every 1 minute
  └── Restore: Every 1 minute, Standby mode (অথবা No recovery)

→ OK → Start job
```

**Step 5 — Verify করুন:**
```sql
-- Primary-এ
SELECT * FROM msdb.dbo.log_shipping_monitor_primary;

-- Secondary-এ replica read-only পড়া যাবে standby mode-এ
SELECT CategoryName FROM MyShopDB.dbo.Categories;  -- ✅ Read works
-- INSERT/UPDATE → ❌ Error: database is read-only
```

### পদ্ধতি ২: Always On Availability Group (Professional Grade) ⭐

**প্রয়োজন:**
- SQL Server **Enterprise Edition** (সব instance-এ)
- সব instance একই **Windows Failover Cluster (WSFC)**-এ (বা Linux cluster)
- Primary + কমপক্ষে ১টি Secondary

**Step 1 — Cluster তৈরি (PowerShell, Domain Admin):**
```powershell
# উভয় node-এ feature install এর পর
New-Cluster -Name "SQLCLUSTER" -Node "SQL-SRV-01","SQL-SRV-02" `
    -StaticAddress "192.168.1.50"
```

**Step 2 — Always On Enable করুন (প্রতিটা instance-এ):**
```sql
-- SSMS → Server → Properties → Enable Always On Availability Groups ✅
-- অথবা SQL Configuration Manager → SQL Server Services → Properties → AlwaysOn
```

**Step 3 — Database Preparation:**
```sql
ALTER DATABASE MyShopDB SET RECOVERY FULL;

BACKUP DATABASE MyShopDB TO DISK = '\\SQL-SRV-01\AGShare\MyShopDB.bak' WITH COMPRESSION;
BACKUP LOG MyShopDB      TO DISK = '\\SQL-SRV-01\AGShare\MyShopDB.trn';
```

**Step 4 — Availability Group Wizard (SSMS):**
```
SSMS → Always On High Availability → Right Click
  → New Availability Group Wizard...

✔ Availability group name: AG-MyShop-Prod

✔ Select databases: MyShopDB  (Primary replica হিসেবে)

✔ Add Replica: SQL-SRV-02
     ├── Availability Mode: Synchronous commit (HA) / Asynchronous (DR)
     ├── Readable Secondary: YES — Read-intent only ✅  ← খুবই গুরুত্বপূর্ণ!
     └── Endpoint: HADR (port 5022) — firewall-এ allow করুন

✔ Listener (Virtual network name — অ্যাপ্লিকেশন এটাই ব্যবহার করবে):
     ├── Listener DNS Name: AG-MyShop-Listener
     └── Port: 1433, Static IP: 192.168.1.60

✔ Initial Data Synchronization: Automatic (Full + Log backup path দিন)

→ Validation → Finish
```

**Step 5 — Verify:**
```sql
-- AG স্ট্যাটাস দেখুন
SELECT ag.name, ar.replica_server_name, drs.role_desc, drs.synchronization_health_desc
FROM sys.dm_hadr_availability_replica_states drs
JOIN sys.availability_replicas ar ON drs.replica_id = ar.replica_id
JOIN sys.availability_groups ag ON ar.group_id = ag.group_id;

-- Secondary-এ readable কিনা test (SSMS-এ SQL-SRV-02-তে)
USE MyShopDB;
SELECT TOP 10 * FROM dbo.Products;   -- ✅ Read works (read-intent connection দিয়ে)
```

---

## 9. ADO.NET দিয়ে Read Replica Routing

### 9.1 Connection Strings

```json
// appsettings.json
{
  "ConnectionStrings": {
    // লেখা সবসময় Primary-তে যাবে
    "PrimaryDB": "Server=AG-MyShop-Listener,1433;Database=MyShopDB;User Id=app_user;Password=***;Encrypt=True;TrustServerCertificate=True;",

    // পড়া Read Replica (readable secondary)-তে routing হবে 👇
    "ReadOnlyDB": "Server=AG-MyShop-Listener,1433;Database=MyShopDB;ApplicationIntent=ReadOnly;User Id=app_user;Password=***;Encrypt=True;TrustServerCertificate=True;MultiSubnetFailover=True;"
  }
}
```

> 🔑 **ম্যাজিক কী:** `ApplicationIntent=ReadOnly` —
> এটা দিলে SQL Server Listener নিজেই connection-টা idle Secondary replica-তে পাঠিয়ে দেয়।
> আর `MultiSubnetFailover=True` failover-এর সময় দ্রুত reconnect করে।

### 9.2 Read/Write Separation — Repository Pattern

```csharp
using Microsoft.Data.SqlClient;
using Microsoft.Extensions.Configuration;

public class OrderService
{
    private readonly string _writeConn;   // Primary
    private readonly string _readConn;    // Read Replica

    public OrderService(IConfiguration config)
    {
        _writeConn = config.GetConnectionString("PrimaryDB")!;
        _readConn  = config.GetConnectionString("ReadOnlyDB")!;
    }

    // ───── WRITE → Primary DB ─────
    public async Task<int> CreateOrderAsync(int customerId, decimal amount)
    {
        await using var conn = new SqlConnection(_writeConn);
        await conn.OpenAsync();

        await using var cmd = new SqlCommand(@"
            INSERT INTO dbo.Orders (CustomerId, Amount, CreatedAt)
            OUTPUT INSERTED.Id
            VALUES (@cid, @amt, SYSUTCDATETIME());", conn);

        cmd.Parameters.AddWithValue("@cid", customerId);
        cmd.Parameters.AddWithValue("@amt", amount);

        return (int)(await cmd.ExecuteScalarAsync())!;
    }

    // ───── READ → Read Replica (ভারী query, replica-তে চাপ পড়ে না) ─────
    public async Task<List<OrderSummary>> GetMonthlySalesReportAsync(int year)
    {
        var result = new List<OrderSummary>();

        // ⚡ ApplicationIntent=ReadOnly — এটা replica-তে route হবে
        await using var conn = new SqlConnection(_readConn);
        await conn.OpenAsync();

        await using var cmd = new SqlCommand(@"
            SELECT MONTH(CreatedAt) AS Month, COUNT(*) AS TotalOrders, SUM(Amount) AS Revenue
            FROM dbo.Orders
            WHERE YEAR(CreatedAt) = @year
            GROUP BY MONTH(CreatedAt)
            ORDER BY Month;", conn);

        cmd.Parameters.AddWithValue("@year", year);
        cmd.CommandTimeout = 120;   // ভারী report → বেশি timeout

        await using var reader = await cmd.ExecuteReaderAsync();
        while (await reader.ReadAsync())
        {
            result.Add(new OrderSummary(
                reader.GetInt32(0), reader.GetInt32(1), reader.GetDecimal(2)));
        }
        return result;
    }
}

public record OrderSummary(int Month, int TotalOrders, decimal Revenue);
```

### 9.3 EF Core-এও একই নীতি

```csharp
// DbContextFactory দিয়ে read/write আলাদা connection
services.AddDbContext<AppDbContext>((sp, options) =>
{
    var config = sp.GetRequiredService<IConfiguration>();
    var http = sp.GetRequiredService<IHttpContextAccessor>();

    // হালকা heuristic: read-only হলে replica, নাহলে primary
    var isReadOnly = http.HttpContext?.Request.Method == "GET";
    var connStr = isReadOnly
        ? config.GetConnectionString("ReadOnlyDB")
        : config.GetConnectionString("PrimaryDB");

    options.UseSqlServer(connStr);
});
```

---

## 10. Professional Grade Architecture

Production-grade সিস্টেমে **Caching + Read Replica একসাথে** tiered strategy হিসেবে কাজ করে:

```
                        ┌──────────────────────────────┐
   HTTP Request ───────▶│  1️⃣ In-Memory Cache (per app)│── Hit? → Return (ns)
                        └──────────────┬───────────────┘
                                       │ Miss
                        ┌──────────────▼───────────────┐
                        │  2️⃣ Redis Distributed Cache   │── Hit? → Return (ms)
                        └──────────────┬───────────────┘
                                       │ Miss
                        ┌──────────────▼───────────────┐
                        │  3️⃣ Read Replica (SELECT)     │── Return (ms)
                        └──────────────┬───────────────┘
                                       │ Only for writes ↓
                        ┌──────────────▼───────────────┐
                        │  4️⃣ Primary DB (INSERT/UPDATE)│
                        │  → Cache Invalidate/Update    │
                        └──────────────────────────────┘
```

### Production Checklist

- [ ] **Health check:** Redis down হলে cache miss path-এ gracefully DB-তে fallback
- [ ] **Failover handling:** `MultiSubnetFailover=True`, retry policy (Polly)
- [ ] **Monitoring:** Redis hit ratio > 85% হওয়া উচিত; SQL `dm_hadr_database_replica_states` দিয়ে lag monitor
- [ ] **Cache warming:** Cold start-এ গুরুত্বপূর্ণ data preload (background job)
- [ ] **Secret management:** Connection string-এ password না — Azure Key Vault / AWS Secrets Manager
- [ ] **Load test:** Cache stampede পরিস্থিতিতে database collapse হয় কিনা দেখুন
- [ ] **DR plan:** Replica থেকে backup নিন; Primary crash → AG automatic failover test করুন

### Monitoring Queries (SSMS)

```sql
-- Replication lag কত?
SELECT ag.name, ar.replica_server_name,
       drs.log_send_queue_size, drs.log_send_rate,
       drs.redo_queue_size, drs.redo_rate
FROM sys.dm_hadr_database_replica_states drs
JOIN sys.availability_replicas ar ON drs.replica_id = ar.replica_id
JOIN sys.availability_groups ag ON ar.group_id = ag.group_id;

-- কোন connection কোথায় যাচ্ছে? (Primary না Replica?)
SELECT c.session_id, c.net_transport, c.client_net_address,
       s.host_name, s.program_name, c.connect_time
FROM sys.dm_exec_connections c
JOIN sys.dm_exec_sessions s ON c.session_id = s.session_id
WHERE s.program_name LIKE '%MyApp%';
```

---

## 11. Best Practices & Common Mistakes

### ✅ Best Practices

1. **Cache করুন:** category list, product catalog, config/settings, report results
2. **Cache করবেন না:** real-time balance, stock-এর exact count, authentication data
3. প্রতিটা cache key-এ **prefix + version** রাখুন (`shop:v2:product:101`)
4. TTL-এ **random jitter** যোগ করুন (সব key একসাথে expire না হয়)
5. Write-এর পরে **সঙ্গে সঙ্গে invalidate** — "পুরনো ভুল data" user দেখবে না
6. Replica থেকে **tolerable staleness** আছে এমন data-ই পড়ুন (reports, feeds)
7. `ApplicationIntent=ReadOnly` দিয়ে **connection-level routing** করুন — অ্যাপ্লিকেশন code-এ replica-র নাম hardcode করবেন না

### ❌ Common Mistakes

| Mistake | ফলাফল |
|---|---|
| Update করেও cache invalidate না করা | User ৫-১০ মিনিট ভুল data দেখে |
| Replica থেকে real-time data পড়া | Lag-এর কারণে inconsistent result |
| Cache key-এ user-specific data না বসানো | এক user আরেক user-র data দেখে! 🔴 |
| Cache-এ NULL result না রাখা | না-থাকা data-র জন্য বারবার DB hit (cache penetration) |
| সব data একসাথে warm করা | Cold start-এ database spike |
| Redis-এ sensitive data plain text-এ | Encryption / Hash ব্যবহার করুন |

---

## 12. Glossary

| শব্দ | ব্যাখ্যা |
|---|---|
| **Cache Hit / Miss** | Cache-এ data পাওয়া গেলো / না গেলো |
| **TTL** | Time To Live — cache কতক্ষণ বাঁচবে |
| **Cache Stampede** | একসাথে হাজারো request cache miss হওয়ায় DB collapse |
| **Cache Penetration** | এমন key query করা যেটা DB-তেও নেই |
| **Primary / Master** | মূল read-write database |
| **Replica / Secondary** | Primary-র read-only কপি |
| **Replication Lag** | Replica-র data primary থেকে কতটুকু পেছনে |
| **Listener** | AG-র virtual network name — অ্যাপ এটাই ব্যবহার করে |
| **Failover** | Primary নষ্ট হলে Secondary-কে primary বানানো |
| **AG (Availability Group)** | SQL Server-এর HA/DR solution, readable secondary সহ |

---

⬅️ [Sharding ও Partitioning](10-sharding-and-partitioning.md) | ➡️ [gRPC ও GraphQL](12-grpc-and-graphql.md)
