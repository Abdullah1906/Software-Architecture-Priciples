# Chapter ১১: Caching

## ১. Caching কী?

বারবার দরকার হয় এমন data **দ্রুত জায়গায় অস্থায়ীভাবে** রেখে দেওয়া, যাতে প্রতিবার ধীর উৎস (database, API) থেকে আনতে না হয়।

> উদাহরণ: প্রতিবার লাইব্রেরি থেকে বই আনার বদলে প্রয়োজনীয় বইটি টেবিলেই রেখে দেওয়া।

```
Request ─▶ Cache ── hit ✅ ──▶ দ্রুত উত্তর
              │
            miss ❌
              ▼
          Database ─▶ উত্তর + Cache-এ রাখো
```

- **Cache Hit** = cache-এ পাওয়া গেছে
- **Cache Miss** = পাওয়া যায়নি, উৎস থেকে আনতে হলো
- **Hit Ratio** = hit ÷ মোট request (যত বেশি, তত ভালো)

---

## ২. কেন ব্যবহার করি?

| সুবিধা | ব্যাখ্যা |
|---|---|
| **গতি** | Memory-র access, Database-এর চেয়ে বহুগুণ দ্রুত |
| **DB-র চাপ কম** | কম query → কম খরচ |
| **Scalability** | একই হার্ডওয়্যারে বেশি user |
| **সহনশীলতা** | উৎস ধীর/অনুপলব্ধ হলেও কিছু উত্তর দেওয়া যায় |

---

## ৩. Cache কোথায় থাকে? (স্তর)

```
Browser Cache → CDN → API Gateway/Reverse Proxy → Application Cache → DB Cache
   (client)    (edge)                              (In-memory/Redis)
```

| স্তর | উদাহরণ | উপযোগী |
|---|---|---|
| Browser | HTTP cache headers | CSS, JS, ছবি |
| CDN | Cloudflare, Azure CDN | Static ফাইল, বিশ্বব্যাপী |
| Application (In-Memory) | `IMemoryCache` | একটি server-এর মধ্যে |
| Distributed | **Redis**, Memcached | একাধিক server শেয়ার করে |
| Database | Query/Buffer cache | DB নিজেই সামলায় |

---

## ৪. Caching Pattern

### ক) Cache-Aside (Lazy Loading) ⭐ সবচেয়ে প্রচলিত
Application নিজে cache পরীক্ষা করে।

```csharp
public async Task<ProductDto?> GetProductAsync(int id)
{
    var key = $"product:{id}";

    var cached = await _cache.GetStringAsync(key);
    if (cached is not null)
        return JsonSerializer.Deserialize<ProductDto>(cached);   // HIT

    var product = await _repo.GetByIdAsync(id);                  // MISS → DB
    if (product is null) return null;

    await _cache.SetStringAsync(key, JsonSerializer.Serialize(product),
        new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10)
        });
    return product;
}
```

### খ) Read-Through
Cache নিজেই DB থেকে আনে (Application শুধু cache-কে জিজ্ঞেস করে)।

### গ) Write-Through
Write হলে **একই সাথে** cache ও DB আপডেট → সঙ্গতি ভালো, write ধীর।

### ঘ) Write-Behind (Write-Back)
আগে cache-এ লেখে, **পরে async-ভাবে** DB-তে → দ্রুত, কিন্তু crash-এ data হারানোর ঝুঁকি।

| Pattern | সুবিধা | ঝুঁকি |
|---|---|---|
| Cache-Aside | সরল, নমনীয় | প্রথম request ধীর, stale হতে পারে |
| Write-Through | সঙ্গত data | Write ধীর |
| Write-Behind | Write খুব দ্রুত | Data হারানোর সম্ভাবনা |

---

## ৫. ASP.NET Core-এ Cache

### In-Memory (এক server)
```csharp
builder.Services.AddMemoryCache();

var data = await _memoryCache.GetOrCreateAsync("categories", async entry =>
{
    entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
    return await _repo.GetCategoriesAsync();
});
```

### Distributed (Redis — একাধিক server)
```csharp
builder.Services.AddStackExchangeRedisCache(o =>
{
    o.Configuration = "localhost:6379";
    o.InstanceName = "myapp:";
});
```

### HTTP Response Caching / Output Caching
```csharp
builder.Services.AddOutputCache();
app.UseOutputCache();

[HttpGet]
[OutputCache(Duration = 60)]
public IActionResult GetCategories() => Ok(...);
```

---

## ৬. Eviction ও Expiration

Cache-এর জায়গা সীমিত; পুরনোকে সরাতে হয়।

| নীতি | মানে |
|---|---|
| **TTL (Time To Live)** | নির্দিষ্ট সময় পরে মেয়াদোত্তীর্ণ |
| **Sliding Expiration** | শেষ ব্যবহারের পর নির্দিষ্ট সময় |
| **LRU** | সবচেয়ে কম সম্প্রতি ব্যবহৃত বাদ |
| **LFU** | সবচেয়ে কম ঘন ব্যবহৃত বাদ |

---

## ৭. 🔥 সবচেয়ে কঠিন সমস্যা: Cache Invalidation

> *"কম্পিউটার বিজ্ঞানে দুটি কঠিন সমস্যা: cache invalidation এবং নামকরণ।"*

DB বদলালে cache-এর পুরনো (stale) data কীভাবে ঠিক করবেন?

| কৌশল | ব্যাখ্যা |
|---|---|
| **TTL** | সময় শেষে নিজে মুছে যাবে (সবচেয়ে সরল) |
| **Write-এ মুছে ফেলা** | Update/Delete হলে সংশ্লিষ্ট key মুছুন |
| **Event-based** | Domain Event থেকে cache invalidate |
| **Versioned key** | `product:5:v3` |

```csharp
public async Task UpdateProductAsync(Product p)
{
    await _repo.UpdateAsync(p);
    await _cache.RemoveAsync($"product:{p.Id}");   // মুছে ফেলুন
}
```

---

## ৮. সাধারণ সমস্যা ও সমাধান

| সমস্যা | মানে | সমাধান |
|---|---|---|
| **Cache Stampede** (Thundering Herd) | জনপ্রিয় key মেয়াদ শেষ হতেই হাজার request একসাথে DB-তে | Lock/single-flight, TTL-এ random jitter |
| **Cache Penetration** | অস্তিত্বহীন key বারবার চাওয়া → সব DB-তে | খালি ফলাফলও (null) ছোট TTL-এ cache; Bloom filter |
| **Cache Avalanche** | অনেক key একসাথে মেয়াদোত্তীর্ণ | TTL-এ এলোমেলো পার্থক্য |
| **Stale Data** | পুরনো data দেখানো | সঠিক TTL/invalidation |
| **Hot Key** | একটি key-তেই বিপুল চাপ | Local cache + Redis, replication |

---

## ৯. কী cache করবেন, কী নয়?

| ✅ উপযোগী | ❌ সাবধান |
|---|---|
| বারবার পড়া, কম বদলায় (category, config, dropdown) | ঘন ঘন বদলায় এমন data |
| ব্যয়বহুল গণনা/report | ব্যক্তিগত/সংবেদনশীল data (বিনা বিবেচনায়) |
| ধীর external API-র উত্তর | Strong consistency দরকার (ব্যাংক balance) |
| Session, token | Cache-এর খরচ বেশি, লাভ কম |

---

## ১০. Professional Practices

1. **আগে মাপুন**, তারপর cache করুন — অকারণ জটিলতা বাড়াবেন না
2. সবসময় **TTL** রাখুন
3. **Key naming convention** (`entity:id:variant`)
4. **Hit ratio** monitor করুন
5. Cache **কখনোই একমাত্র উৎস নয়** — হারালে system যেন চলে (graceful fallback)
6. User-ভিত্তিক data-র key-তে user/tenant Id রাখুন (অন্যের data leak এড়াতে) ⚠️
7. বড় object এড়ান; শুধু প্রয়োজনীয় DTO cache করুন
8. Stampede ঠেকাতে lock বা `GetOrCreate` + semaphore

---

## ✅ সারসংক্ষেপ

- Cache = দ্রুত অস্থায়ী সংরক্ষণ; DB-র চাপ কমায়, গতি বাড়ায়
- **Cache-Aside** + **TTL** = সবচেয়ে সাধারণ ও নিরাপদ শুরু
- আসল কঠিন কাজ **invalidation**; stampede/penetration/avalanche সামলান
- একাধিক server হলে **Redis** (distributed cache)

⬅️ [Sharding ও Partitioning](10-sharding-and-partitioning.md) | ➡️ [gRPC ও GraphQL](12-grpc-and-graphql.md)
