# Chapter ১০: Database Sharding ও Partitioning
# 🗄️ Database Sharding ও Partitioning — Complete Guide

&gt; এই ডকুমেন্টে Database **Partitioning** এবং **Sharding** কী, কেন দরকার, এবং Production/Professional level-এ কীভাবে ব্যবহার হয় — সবকিছু step-by-step, example সহ বাংলায় ব্যাখ্যা করা হয়েছে।

---

## 📑 Table of Contents

- [Partitioning কী?](#-partitioning-কী)
- [Sharding কী?](#-sharding-কী)
- [Partitioning vs Sharding](#-partitioning-vs-sharding-পার্থক্য)
- [কেন ব্যবহার করবো? (Why)](#-কেন-ব্যবহার-করবো-why)
- [কখন কোনটা ব্যবহার করবো?](#-কখন-কোনটা-ব্যবহার-করবো)
- [Partitioning-এর প্রকারভেদ ও ব্যবহার](#-partitioning-এর-প্রকারভেদ-ও-ব্যবহার)
- [Sharding-এর প্রকারভেদ ও ব্যবহার](#-sharding-এর-প্রকারভেদ-ও-ব্যবহার)
- [Professional Grade Implementation (Full Steps)](#-professional-grade-implementation-full-steps)
- [Real-Life Examples](#-real-life-examples)
- [Best Practices](#-best-practices)
- [Common Mistakes / Pitfalls](#-common-mistakes--pitfalls)
- [Tools & Technology Stack](#-tools--technology-stack)

---

## 🔹 Partitioning কী?

**Partitioning** মানে হলো — একটা বড় টেবিলকে **একই Database-র ভেতরে** ছোট ছোট ভাগে (logical chunks) ভাগ করে ফেলা।

👉 একই Database, একই টেবিল — শুধু ডেটা গুলো ভেতরে ভেতরে ভাগে ভাগে থাকে। User কিংবা Application-এর কাছে এটা সাধারণ একটা টেবিলের মতোই দেখায়।

### উদাহরণ:

ধরুন, আপনার `orders` টেবিলে **১০ কোটি (100 Million)** row আছে। প্রতিটা query-তে full table scan হলে অনেক slow হবে। তখন আপনি `created_at` column-এর basis-এ টেবিলটাকে ভাগ করে দিলেন:

```
orders_2024
orders_2025
orders_2026
```

এখন `WHERE created_at = '2026-01-01'` query-টা শুধু `orders_2026` partition-এর ভেতরে খুঁজবে — **১০ কোটি row-র বদলে ১ কোটি row scan** হবে। Speed অনেক গুণ বেড়ে যাবে। 🚀

---

## 🔹 Sharding কী?

**Sharding** মানে হলো — ডেটাকে **একাধিক আলাদা Database server/machine-এ** ভাগ করে রাখা। প্রতিটা shard-ই **independent database** — আলাদা CPU, RAM, storage নিয়ে আলাদা node-এ চলে।

👉 Sharding মূলত **Horizontal Scaling** — একটা মেশিনে আর ডেটা ঢোলানো যাচ্ছে না, তখন আরো মেশিন যোগ করে ডেটা ভাগ করে দেওয়া।

```
┌─────────────────┐     ┌─────────────────┐
│   Shard 1       │     │   Shard 2       │
│  (User ID 1-10M)│     │  (User ID 10M+) │
│  Server A       │     │  Server B       │
└─────────────────┘     └─────────────────┘
```

---

## 🔹 Partitioning vs Sharding (পার্থক্য)

| বিষয় | Partitioning | Sharding |
|------|-------------|----------|
| **কোথায় হয়** | একই Database-র ভেতরে | আলাদা আলাদা Database Server-এ |
| **Hardware** | একই machine/share storage | আলাদা machine, আলাদা resource |
| **Scaling Type** | Vertical (single DB-তে optimize) | Horizontal (নতুন server যোগ) |
| **Management** | Database engine নিজেই handle করে | Application / Middleware handle করে |
| **Complexity** | তুলনামূলক কম | অনেক বেশি |
| **Cross-query** | সহজ (সব এক DB-তেই) | কঠিন (Cross-shard query) |
| **Availability** | Single point of failure থাকে | High availability পাওয়া যায় |

&gt; 💡 **সহজ ভাষায়:** Partitioning = এক ঘরের ভেতরে আলমারি ভাগ করা। Sharding = একাধিক আলাদা বাড়িতে জিনিস রাখা।

---

## 🔹 কেন ব্যবহার করবো? (Why)

### যে সমস্যাগুলোর সমাধান দেয়:

1. **🐢 Slow Query Performance** — বড় টেবিলে index-এর size বড় হয়ে যায়, query slow হয়।
2. **📦 Storage Limit** — একটা মেশিনের disk সীমিত। বিলিয়ন row রাখা যায় না।
3. **✍️ Write Bottleneck** — সব write একটা DB-তে যাচ্ছে → lock, contention বাড়ে।
4. **💥 Single Point of Failure** — একটা DB down হলে সব down। Sharding-এ একটা shard down হলে বাকিগুলো চলে।
5. **💰 Cost Efficiency** — অনেক বড় single server-এর চেয়ে ছোট ছোট commodity server সস্তা।
6. **🔒 Compliance** — EU user-দের ডেটা EU server-এ রাখা (Geographic Sharding) — GDPR compliance-এর জন্য দরকার হয়।

---

## 🔹 কখন কোনটা ব্যবহার করবো?

### Partitioning যখন:
- ✅ টেবিলের size বড় (কয়েক কোটি row+), কিন্তু **একটা DB server যথেষ্ট**
- ✅ Time-series ডেটা (logs, events, orders)
- ✅ পুরনো ডেটা archive/delete করতে চান (partition drop করলেই হয়)
- ✅ Query সবসময় কোনো নির্দিষ্ট range/date-এর উপর হয়

### Sharding যখন:
- ✅ Single DB server-এর **CPU/Storage/Connection limit শেষ**
- ✅ Write throughput বাড়ানো দরকার (per-second লাখ লাখ write)
- ✅ High availability দরকার
- ✅ Globally distributed user base
- ✅ **মনে রাখবেন:** Sharding-এর আগে Partitioning + Caching + Read Replica দিয়ে দেখুন — যদি যথেষ্ট হয়, Sharding করবেন না। Sharding last resort। ⚠️

---

## 🔹 Partitioning-এর প্রকারভেদ ও ব্যবহার

### 1️⃣ Horizontal Partitioning (Range Partitioning)
একটি বিশাল টেবিলের সবগুলো Column একই থাকবে, কিন্তু Row (ডাটা) গুলোকে কোনো নির্দিষ্ট Column (যেমন: Date, Year, ID)-এর Range বা ব্যাপ্তি অনুসারে ছোট ছোট অংশে ভাগ করে রাখা হয়।

*cকেন ব্যবহার করবেন? ধরুন আপনার Orders টেবিলে ۱۰ বছরের ডাটা আছে (১০ কোটি Row)। আপনি যদি ২০২৬ সালের ডাটা সার্চ করেন, SQL Server পুরো ১০ কোটি Row না খুঁজে শুধু ২০২৬ সালের নির্দিষ্ট Partition-এ খুঁজবে। একে Partition Pruning বলা হয়, যা Query Performance অনেক বাড়িয়ে দেয়।

SSMS (SQL Server)-এ কীভাবে করবেন?
SQL Server-এ Range Partitioning করতে ৩টি ধাপ অনুসরণ করতে হয়:

1. Partition Function: Range নির্ধারণ করে (কোথায় ভাগ হবে)।

2. Partition Scheme: Partition গুলোকে Storage/Filegroup-এ ম্যাপ করে।

3. Partitioned Table: মূল টেবিল তৈরি করা।

**Example — SSMS:**

```sql
-- ১. Partition Function তৈরি (Date Range অনুযায়ী)
CREATE PARTITION FUNCTION OrderDateRangePF (DATETIME)
AS RANGE RIGHT FOR VALUES (
    '2024-01-01', 
    '2025-01-01', 
    '2026-01-01'
);

-- ২. Partition Scheme তৈরি
CREATE PARTITION SCHEME OrderDateRangePS
AS PARTITION OrderDateRangePF
ALL TO ([PRIMARY]); -- সব ডাটা Primary Filegroup-এ থাকবে

-- ৩. Partitioned Table তৈরি
CREATE TABLE Orders (
    OrderId INT IDENTITY(1,1),
    UserId INT NOT NULL,
    Amount DECIMAL(10,2),
    CreatedAt DATETIME NOT NULL,
    Status VARCHAR(20)
) ON OrderDateRangePS(CreatedAt); -- Partition Scheme-এর সাথে যুক্ত করা হলো
```


### 2️⃣ Vertical Partitioning
**Column** অনুযায়ী টেবিল ভাগ করা। বড়/কম ব্যবহৃত column আলাদা টেবিলে।

**Example:** `users` টেবিলে `profile_picture_blob` (৫ MB) সবসময় লাগে না → আলাদা `user_media` টেবিলে।

```sql
-- ১. Frequently Accessed Data (Main Table)
CREATE TABLE Users (
    UserId INT IDENTITY(1,1) PRIMARY KEY,
    Username VARCHAR(50) NOT NULL,
    Email VARCHAR(100) NOT NULL,
    PasswordHash VARCHAR(255) NOT NULL
);

-- ۲. Rarely Accessed Large Data (Separate Table)
CREATE TABLE UserMedia (
    UserId INT PRIMARY KEY,
    AvatarBlob VARBINARY(MAX), -- বড় বাইনারি ডাটা
    Bio NVARCHAR(MAX),
    CONSTRAINT FK_UserMedia_Users FOREIGN KEY (UserId) REFERENCES Users(UserId)
);
```

ব্যবহার করার নিয়ম:

* সাধারণ Login বা User List দেখানোর সময় শুধু Users টেবিল Query করবেন (SELECT Username, Email FROM Users).

* যখন User Profile Detail দেখাবেন, তখন JOIN করে ডাটা আনবেন:
```bash
SELECT u.Username, u.Email, m.Bio 
FROM Users u
LEFT JOIN UserMedia m ON u.UserId = m.UserId
WHERE u.UserId = 10;
```

### 3️⃣ List Partitioning
ডাটাকে নির্দিষ্ট কিছু মান বা ক্যাটাগরি (Discrete Values/List) অনুযায়ী ভাগ করা (যেমন: Division, Country, Status)।

* বিশেষ দ্রষ্টব্য: SQL Server (SSMS)-এ সরাসরি PARTITION BY LIST নামের কোনো বিল্ট-ইন ফিউচার নেই (যা PostgreSQL বা Oracle-এ আছে)। তবে SSMS-এ এটি Partition Function এবং CHECK Constraint দিয়ে অর্জন করা হয়।

```sql
-- Partition Function দিয়ে Discrete values হ্যান্ডেল করা
CREATE PARTITION FUNCTION RegionListPF (VARCHAR(20))
AS RANGE LEFT FOR VALUES ('CHATTOGRAM', 'DHAKA', 'KHULNA');

CREATE PARTITION SCHEME RegionListPS
AS PARTITION RegionListPF
ALL TO ([PRIMARY]);

CREATE TABLE OrdersByRegion (
    OrderId INT IDENTITY(1,1),
    Region VARCHAR(20) NOT NULL,
    Amount DECIMAL(10,2)
) ON RegionListPS(Region);
```

### 4️⃣ Hash Partitioning
ডাটা কোনো তারিখ বা ক্যাটাগরি মেনে চলে না। তাই SQL Server ডাটার ওপর একটি Hash Algorithm প্রয়োগ করে ডাটাগুলোকে সমানভাবে বিভিন্ন Partition-এ ছড়িয়ে দেয় (Even Distribution)।

* কেন ব্যবহার করবেন? যখন ডাটাতে কোনো নির্দিষ্ট Range থাকে না, কিন্তু ডাটার সাইজ এতো বেশি যে তা একাধিক Partition-এ সমানভাবে ভাগ করা প্রয়োজন (High Concurrency এড়াতে)।

* SSMS-এ কীভাবে কাজ করে? SQL Server-এ Hash Partitioning করতে আমরা একটি Computed Column (Hash Value) তৈরি করি এবং তার ওপর Range Partition 적용 করি।

```sql
-- ১. Hash Value বের করার জন্য Computed Column সহ টেবিল তৈরি
CREATE TABLE UserSessions (
    SessionId INT IDENTITY(1,1),
    UserId INT NOT NULL,
    Token VARCHAR(255),
    -- UserId-কে ৪টি ಭಾಗ (0, 1, 2, 3)-এ ভাগ করার জন্য Modulo/Hash কলাম
    HashPartitionKey AS (ABS(UserId) % 4) PERSISTED 
);

-- ۲. Hash Key-এর ওপর Partition Function
CREATE PARTITION FUNCTION SessionHashPF (INT)
AS RANGE LEFT FOR VALUES (0, 1, 2);

CREATE PARTITION SCHEME SessionHashPS
AS PARTITION SessionHashPF
ALL TO ([PRIMARY]);

-- ৩. Partitioned Table তৈরি
CREATE TABLE PartitionedSessions (
    SessionId INT IDENTITY(1,1),
    UserId INT NOT NULL,
    Token VARCHAR(255),
    HashPartitionKey AS (ABS(UserId) % 4) PERSISTED
) ON SessionHashPS(HashPartitionKey);
```

---

## 🔹 Sharding-এর প্রকারভেদ ও ব্যবহার

### 1️⃣ Range-Based Sharding
Key-এর range অনুযায়ী shard নির্বাচন।

```
Shard 1: user_id 1     – 10,000,000
Shard 2: user_id 10M+  – 20,000,000
Shard 3: user_id 20M+  – 30,000,000
```

**✅ সুবিধা:** সহজ, সহজে range query করা যায়।
**❌ অসুবিধা:** Uneven load (hotspot) হতে পারে — অনেক celebrity user একই shard-এ পড়লে সেই shard overload হবে।

### 2️⃣ Hash-Based Sharding
Key-এর hash করে shard বেছে নেওয়া।

```python
shard_number = hash(user_id) % total_shards   # 0, 1, 2
```

**✅ সুবিধা:** ডেটা evenly distribute হয়।
**❌ অসুবিধা:** Range query কষ্টসাধ্য; shard বাড়ালে (resharding) প্রায় সব ডেটা move করতে হয়।

### 3️⃣ Directory-Based Sharding
একটা **Lookup table/service** থাকে — কোন key কোন shard-এ আছে সেটা জানায়।

```
lookup_table:
┌─────────┬──────────┐
│ user_id │ shard_id │
├─────────┼──────────┤
│ 101     │ 1        │
│ 102     │ 3        │
│ 103     │ 2        │
└─────────┴──────────┘
```

**✅ সুবিধা:** Flexible — resharding সহজ, uneven data move করা যায়।
**❌ অসুবিধা:** Lookup service নিজেই single point of failure + extra latency।

### 4️⃣ Geographic (Geo-Based) Sharding
Region অনুযায়ী shard — latency কমানো ও compliance-এর জন্য।

```
Shard EU → European users
Shard US → American users
Shard ASIA → Asian users
```

---

## 🏆 Professional Grade Implementation (Full Steps)

এখন দেখবো Production environment-এ কীভাবে Sharding implement করা হয় — **ধাপে ধাপে**।

### Step 1: শার্ডিং কি সত্যিই দরকার? (Assessment)
- পুরো system profile করুন: QPS, read/write ratio, data growth rate, slow queries
- আগে try করুন: **Indexing → Partitioning → Caching (Redis) → Read Replicas → Connection Pooling**
- সব শেষে একটাই primary DB bottleneck হলে → Sharding-এ যান ✅

### Step 2: Sharding Key (Partition Key) বেছে নিন
এটাই **সবচেয়ে গুরুত্বপূর্ণ decision**। ভালো Sharding Key-এর গুণ:

- **High Cardinality** — অনেক unique value (user_id ✅, country ❌ — মাত্র ৬-৭টা value)
- **Even Distribution** — সব shard-এ সমান load
- **Query Locality** — এক user-এর related সব ডেটা এক shard-এ থাকে (user_id দিয়ে sharding করলে user-এর orders, sessions সব এক shard-এ থাকবে)
- **বদলায় না** — shard key কখনো update হওয়া উচিত না

&gt; ⚠️ **আইন নম্বর ১:** Multiple entity sharding করতে হলে সব entity-কে একই shard key-এ shard করুন (user_id based) — নাহলে cross-shard JOIN-এর ঝামেলা।

### Step 3: Number of Shards ঠিক করুন
- বর্তমান need + **৩-৫ বছরের projection** দেখে ঠিক করুন
- Shard কম হলে তাড়াতাড়ি resharding লাগবে
- শুরুতে ৪–৮ shard যথেষ্ট; **Consistent Hashing** ব্যবহার করলে পরে shard যোগ করা সহজ

### Step 4: Shard Routing Layer বানান
Application-কে জানতে হবে কোন query কোন shard-এ যাবে। ৩টা পদ্ধতি:

**Option A: Application-level routing (Simple)**

```python
# config.py
SHARD_COUNT = 4
SHARDS = {
    0: "postgresql://shard0.db.internal:5432/app",
    1: "postgresql://shard1.db.internal:5432/app",
    2: "postgresql://shard2.db.internal:5432/app",
    3: "postgresql://shard3.db.internal:5432/app",
}

# router.py
import hashlib

def get_shard(key: str) -&gt; int:
    """Consistent shard number বের করার জন্য hash"""
    h = int(hashlib.md5(key.encode()).hexdigest(), 16)
    return h % SHARD_COUNT

def get_connection(user_id: int):
    shard_id = get_shard(str(user_id))
    return connect(SHARDS[shard_id])

# Usage
def create_order(user_id: int, amount: float):
    conn = get_connection(user_id)   # সবসময় user_id অনুযায়ী shard
    conn.execute(
        "INSERT INTO orders (user_id, amount) VALUES (%s, %s)",
        (user_id, amount)
    )
```

**Option B: Shard Proxy / Middleware (Professional ⭐)**
- Application proxy-র সাথে কথা বলে, proxy নিজেই route করে
- Example: **Vitess** (MySQL), **Citus** (PostgreSQL), **ProxySQL**

**Option C: Distributed SQL Database (New Age)**
- Sharding built-in — নিজে handle করে
- Example: **CockroachDB, TiDB, YugabyteDB, Google Spanner**

### Step 5: Schema Design (Shard-এর জন্য)
- **একই schema সব shard-এ** থাকবে
- **Global unique ID** দরকার — auto-increment কাজ করবে না!

```sql
-- Global ID generation approaches:
-- 1. UUID / ULID
id UUID DEFAULT gen_random_uuid()

-- 2. Snowflake ID (Twitter-এর method) — timestamp + machine id + sequence
-- 3. Sequence range বরাদ্দ: shard 0 → 1-1B, shard 1 → 1B-2B...
```

### Step 6: Cross-Shard সমস্যা Handle করা

**❌ সমস্যা ১: Cross-shard JOIN**
```sql
-- user_id=101 (shard 0) এর orders এর সাথে user_id=205 (shard 2) এর orders JOIN করা যাবে না
```
**✅ সমাধান:**
- এমন query avoid করুন — data denormalize করে shard-এর ভেতরেই রাখুন
- প্রয়োজনে Application layer-এ দুই shard থেকে data এনে merge করুন (scatter-gather)

**❌ সমস্যা ২: Distributed Transaction**
- একটা operation-এ একাধিক shard লাগলে (money transfer: shard A থেকে কাটা → shard B-তে জমা)
**✅ সমাধান:**
- **Saga Pattern** — প্রতিটা step আলাদা transaction, fail হলে compensating action
- **2-Phase Commit (2PC)** — strong consistency দরকার হলে (slow)
- Practical-এ: এক user-এর সব ডেটা এক shard-এ রাখুন — তাহলে local transaction-ই যথেষ্ট

**❌ সমস্যা ৩: Global Secondary Queries**
- `SELECT * FROM users WHERE email = 'x@y.com'` — email দিয়ে shard বেছে নেওয়া যায় না!
**✅ সমাধান:**
- **Search index আলাদা রাখুন:** Elasticsearch / OpenSearch
- **Directory/Lookup table:** email → user_id → shard_id
- **Broadcast query:** সব shard-এ query পাঠিয়ে merge (শুধু rare admin query-তে)

### Step 7: Rebalancing / Resharding Plan
- ডেটা বাড়লে নতুন shard যোগ করতে হবে
- **Consistent Hashing** ব্যবহার করুন → মাত্র 1/N ডেটা move হবে (modulo hashing-এ সব move হয়)

```
Consistent Hashing Ring:

        shard A ──────► [ key1, key2 ]
       /       \
   key N         shard B ──► [ key3 ]
       \       /
        shard C ──────► [ key4, key5 ]

নতুন shard D যোগ → শুধু key N-র আশেপাশের key গুলো move হবে
```

**Online Resharding Steps:**
1. নতুন shard add করুন → replicate শুরু (dual-write / CDC)
2. Backfill শেষ হলে traffic shift করুন
3. পুরনো shard থেকে verify করে ডেটা remove

### Step 8: Monitoring & Alerting Setup
প্রতিটা shard-এর জন্য track করুন:

| Metric | কেন জরুরি |
|--------|-----------|
| QPS per shard | Load imbalance ধরা |
| Storage per shard | Resharding-এর সময় বুঝতে |
| Replication lag | Replica থেকে stale data |
| Slow queries | Partition pruning কাজ করছে কিনা |
| Shard CPU/Memory/Connections | Capacity planning |

**Tools:** Prometheus + Grafana, pg_stat_statements, Percona Monitoring

### Step 9: Backup & Disaster Recovery
- প্রতিটা shard-এর **আলাদা backup** — shard-গুলো আলাদা physical machine
- **Point-in-time recovery (PITR)** setup করুন
- Regularly **cross-shard restore test** করুন (backup আছে মানে restore হবে এমন না)

### Step 10: Documentation & Runbook
- Shard map কোথায় আছে
- কীভাবে নতুন shard যোগ করবেন
- কীভাবে একটা shard down হলে failover করবেন
- On-call engineer-দের জন্য step-by-step runbook

---

## 🌍 Real-Life Examples

| Company | Approach | শেখা |
|---------|----------|-------|
| **Instagram** | PostgreSQL + manually sharded (billions of users) | user_id-based sharding, application-level routing |
| **Twitter/X** | Snowflake ID + Manhattan (in-house DB) | custom distributed storage |
| **Uber** | Schema-based sharding (Schemaless) | per-entity isolation |
| **Google Spanner** | Global distributed SQL | strong consistency at global scale |
| **Discord** | Monolith → then sharded MongoDB | শুধুমাত্র প্রয়োজনে shard |

---

## ✅ Best Practices

1. **Shard key আবার ভাবুন** — একবার production-এ চলে গেলে shard key বদলানো প্রায় অসম্ভব
2. **Denormalize করুন** — JOIN কমাতে ডেটা duplicate রাখুন (NoSQL মনমতো)
3. **Per-shard connection pool** ব্যবহার করুন (PgBouncer)
4. **Consistent Hashing** → future resharding সহজ হবে
5. **আগে partition করুন, তারপর shard** — একই technique দুই লেভেলে use করা যায়
6. **Cross-shard query-কে "ভিলেন" বানান** — architecture-এ এমনভাবে ডিজাইন করুন যাতে প্রয়োজনই না পড়ে
7. **Load test করুন** — production traffic-এর ২–৩ গুণ দিয়ে
8. **ID generation centralize করুন** — Snowflake/UUID

---

## ⚠️ Common Mistakes / Pitfalls

| Mistake | পরিণতি | সমাধান |
|---------|---------|--------|
| খুব তাড়াতাড়ি shard করা | অযথা complexity | আগে partition + cache + replica |
| খারাপ shard key (যেমন: status, gender) | Hotspot, uneven data | High cardinality key (user_id) |
| Cross-shard JOIN ব্যবহার | Massive latency | Denormalization |
| Application-এ shard logic গুলিয়ে ফেলা | Wrong shard → data loss | Unit test + shard routing layer |
| Resharding plan না রাখা | Downtime ঝুঁকি | Consistent hashing + automation |
| সব shard-এর backup একসাথে রাখা | Single point of failure | Per-shard isolated backup |

---

## 🛠️ Tools & Technology Stack

| Purpose | Tools |
|---------|-------|
| **Distributed SQL (Sharding built-in)** | CockroachDB, TiDB, YugabyteDB, Google Spanner, Amazon Aurora Limitless |
| **MySQL Sharding** | Vitess (PlanetScale), ProxySQL, MySQL NDB Cluster |
| **PostgreSQL Sharding** | Citus, pg_partman (partitioning) |
| **MongoDB** | Built-in Sharding (mongos router, replica sets) |
| **Cache Layer** | Redis, Valkey |
| **Search (cross-shard query-এর বিকল্প)** | Elasticsearch, OpenSearch |
| **CDC / Replication** | Debezium, Kafka |
| **Monitoring** | Prometheus, Grafana, Datadog |

---

## 🎯 উপসংহার

| বিষয় | মূল কথা |
|------|---------|
| **Partitioning** | এক DB-র ভেতরে ডেটা ভাগ — performance-এর জন্য |
| **Sharding** | একাধিক DB server-এ ডেটা ভাগ — scalability-এর জন্য |
| **ক্রম** | Index → Partition → Cache → Replica → **Shard (শেষ অস্ত্র)** |
| **Shard Key** | সবচেয়ে গুরুত্বপূর্ণ সিদ্ধান্ত — ভুল হলে ভুল শেষ |
| **Professional Approach** | Consistent hashing, middleware/proxy, monitoring, runbook |

&gt; 💬 **Remember:** Sharding is a **scalability tool, not a performance magic wand**। সঠিকভাবে implement করলে billion-user system চালানো যায়; ভুলভাবে করলে রাতের ঘুম হারাম। 😅

---

## 📚 References

- [PostgreSQL Table Partitioning Docs](https://www.postgresql.org/docs/current/ddl-partitioning.html)
- [MongoDB Sharding Docs](https://www.mongodb.com/docs/sharding/)
- [Vitess Documentation](https://vitess.io/docs/)
- [Citus (PostgreSQL Extension)](https://www.citusdata.com/)
- Designing Data-Intensive Applications — Martin Kleppmann
- High Scalability Blog (sharding war stories)

---

**⭐ যদি এই গাইড কাজে লাগে, repository-তে একটা star দেবেন!**


⬅️ [CAP Theorem](09-cap-theorem.md) | ➡️ [Caching](11-caching.md)
