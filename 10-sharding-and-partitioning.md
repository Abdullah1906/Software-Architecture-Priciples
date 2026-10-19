# Chapter ১০: Database Sharding ও Partitioning

## ১. সমস্যাটা কী?

Data বাড়তে বাড়তে একটি table-এ কোটি কোটি row হলে:
- Query ধীর হয়ে যায়
- Index বিশাল হয়, memory-তে ধরে না
- Backup/Restore/Maintenance দীর্ঘ সময় নেয়
- একটি server-এর CPU/Disk সীমায় পৌঁছায়

সমাধান: **data ভাগ করা**। ভাগ করার দুই স্তর — **Partitioning** ও **Sharding**।

---

## ২. Partitioning vs Sharding

| | Partitioning | Sharding |
|---|---|---|
| কোথায় ভাগ? | **একই** database server-এর ভেতরে | **একাধিক** আলাদা server-এ |
| ফলাফল | Table-কে ছোট অংশে ভাগ | পুরো dataset অনেক machine-এ ছড়ানো |
| Application জানে? | সাধারণত না (DB নিজেই সামলায়) | হ্যাঁ/Router/Proxy লাগে |
| সুবিধা | Query ও maintenance দ্রুত | **Horizontal scale** — প্রায় সীমাহীন |
| জটিলতা | কম | অনেক বেশি |

> Sharding আসলে একধরনের **horizontal partitioning**, যা একাধিক node জুড়ে।

---

## ৩. Partitioning-এর ধরন

### ক) Horizontal Partitioning
**Row** ভাগ হয়। প্রতিটি অংশে একই column, ভিন্ন row।

```
Orders (সব) ─┬─ Orders_2024  (২০২৪-এর row)
             ├─ Orders_2025
             └─ Orders_2026
```

### খ) Vertical Partitioning
**Column** ভাগ হয়। প্রায়ই-ব্যবহৃত column এক table-এ, ভারী/কম ব্যবহৃত (ছবি, বড় text) আলাদা table-এ।

```
Users (Id, Name, Email)      ← ঘন ঘন
UserProfileDetails (UserId, Bio, Photo) ← ভারী
```

---

## ৪. SQL Server-এ Table Partitioning উদাহরণ

```sql
-- ১) Partition function: তারিখ অনুযায়ী সীমানা
CREATE PARTITION FUNCTION pf_OrderDate (DATETIME2)
AS RANGE RIGHT FOR VALUES ('2025-01-01', '2026-01-01');

-- ২) Partition scheme: কোন filegroup-এ যাবে
CREATE PARTITION SCHEME ps_OrderDate
AS PARTITION pf_OrderDate ALL TO ([PRIMARY]);

-- ৩) Table partition scheme-এ তৈরি
CREATE TABLE Orders
(
    Id          BIGINT IDENTITY,
    OrderDate   DATETIME2 NOT NULL,
    CustomerId  INT NOT NULL,
    Total       DECIMAL(18,2),
    CONSTRAINT PK_Orders PRIMARY KEY (Id, OrderDate)
) ON ps_OrderDate (OrderDate);
```

**সুবিধা:**
- নির্দিষ্ট সময়ের query শুধু প্রাসঙ্গিক partition ছোঁয় (**partition elimination**)
- পুরনো data দ্রুত archive/মুছে ফেলা যায় (`SWITCH PARTITION`)

> 💡 **Query-তে partition key (এখানে `OrderDate`) রাখুন**, নইলে সব partition স্ক্যান হবে।

---

## ৫. Sharding

Data কয়েকটি **shard (আলাদা DB server)**-এ ছড়ানো; প্রতিটি shard পুরো data-র একটি অংশ রাখে।

```
              ┌── Shard ১ (Customer 1–1M)
Application ──┼── Shard ২ (Customer 1M–2M)
 + Router     └── Shard ৩ (Customer 2M–3M)
```

### Shard Key
কোন row কোন shard-এ যাবে তা ঠিক করে। **সবচেয়ে গুরুত্বপূর্ণ সিদ্ধান্ত** — পরে বদলানো কঠিন।

ভালো shard key-র বৈশিষ্ট্য:
- **High cardinality** (অনেক ভিন্ন মান)
- **সমান বণ্টন** (কোনো shard যেন অতিরিক্ত ভারী না হয়)
- Query-তে **ঘন ঘন ব্যবহৃত** (যাতে এক shard-এই উত্তর মেলে)

### Sharding কৌশল

| কৌশল | কীভাবে | সুবিধা | অসুবিধা |
|---|---|---|---|
| **Range-based** | মানের পরিসর (Id ১–১M → Shard১) | সরল, range query ভালো | **Hotspot** (নতুন data সব শেষ shard-এ) |
| **Hash-based** | `hash(key) % N` | সমান বণ্টন | Shard সংখ্যা বদলালে অনেক data সরাতে হয় |
| **Directory/Lookup** | আলাদা table-এ "key → shard" ম্যাপ | নমনীয় | Lookup table-ই bottleneck/single point |
| **Geo-based** | অঞ্চল অনুযায়ী (BD → Shard১, EU → Shard২) | কম latency, আইন মানা (data residency) | অসম বণ্টন |

### Consistent Hashing (সংক্ষেপে)
Hash ring ব্যবহার করে — shard যোগ/বাদ দিলে **শুধু অল্প data** সরাতে হয় (সাধারণ `% N`-এ প্রায় সব সরে)। Cassandra, DynamoDB-এ ব্যবহৃত।

### সরল C# Router উদাহরণ

```csharp
public class ShardRouter
{
    private readonly string[] _connectionStrings; // প্রতিটি shard-এর connection

    public ShardRouter(string[] connectionStrings) => _connectionStrings = connectionStrings;

    public string GetConnection(int customerId)
        => _connectionStrings[customerId % _connectionStrings.Length];
}

// ব্যবহার
var conn = _router.GetConnection(order.CustomerId);
await using var db = new SqlConnection(conn);
```

---

## ৬. Sharding-এর চ্যালেঞ্জ ⚠️

| সমস্যা | ব্যাখ্যা |
|---|---|
| **Cross-shard query/JOIN** | একাধিক shard-এ ছড়ানো data জোড়া কঠিন ও ধীর |
| **Distributed transaction** | ACID কঠিন; Saga/eventual consistency লাগে |
| **Rebalancing (Resharding)** | Shard বাড়ালে data সরানো ব্যয়বহুল |
| **Hotspot** | কোনো একটি shard অতিরিক্ত ভারী |
| **অপারেশনাল জটিলতা** | Backup, monitoring, schema change সব shard-এ |
| **Unique Id** | সব shard জুড়ে অনন্য Id (GUID/Snowflake/ULID) |

---

## ৭. কখন করবেন? — গুরুত্বপূর্ণ ক্রম ⭐

Sharding **শেষ অস্ত্র**। আগে এগুলো চেষ্টা করুন:

```
১. Query ও Index optimize করুন
২. Caching যোগ করুন (Redis)
৩. Read Replica রাখুন (read ভাগ করুন)
৪. Hardware বাড়ান (Vertical scale)
৫. Table Partitioning ব্যবহার করুন
৬. Archive করুন (পুরনো data সরিয়ে)
৭. ──── তারপরেও না হলে ──── Sharding
```

### বাস্তব ব্যবহার
- **Multi-tenant SaaS:** প্রতি tenant বা tenant-গ্রুপ আলাদা shard
- **বিশাল user base:** Facebook, Instagram-এর মতো
- **Time-series/Log:** সময় অনুযায়ী partition

---

## ৮. Professional Practices

1. **আগে partitioning/replica/cache** — sharding নয়
2. Shard key **সাবধানে** বাছুন; query pattern জেনে
3. Cross-shard query **এড়ানোর** মতো data model বানান (সম্পর্কিত data একই shard-এ)
4. **Globally unique Id** (GUID/ULID/Snowflake)
5. Managed সমাধান বিবেচনা করুন (Azure SQL Elastic Pools, Cosmos DB, MongoDB sharding)
6. প্রতিটি shard-এর জন্য **আলাদা backup ও monitoring**

---

## ✅ সারসংক্ষেপ

- **Partitioning** = এক server-এ table ভাগ (query ও maintenance দ্রুত)
- **Sharding** = অনেক server-এ data ছড়ানো (horizontal scale), কিন্তু জটিল
- Shard key সঠিক না হলে hotspot ও cross-shard সমস্যা
- আগে সহজ উপায় (index, cache, replica), sharding শেষে

⬅️ [CAP Theorem](09-cap-theorem.md) | ➡️ [Caching](11-caching.md)
