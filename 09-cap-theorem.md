# Chapter ০৯: CAP Theorem

## ১. CAP Theorem কী?

Distributed system (একাধিক server/node) সম্পর্কে একটি তত্ত্ব (Eric Brewer, ২০০০):

> নেটওয়ার্ক partition ঘটলে, **Consistency** ও **Availability** — এই দুটির মধ্যে **একটি** বেছে নিতে হবে; দুটো একসাথে পুরোপুরি পাওয়া যায় না।

### তিনটি অক্ষর

| অক্ষর | নাম | মানে |
|---|---|---|
| **C** | Consistency | সব node একই সময়ে **একই (সর্বশেষ) data** দেখায় |
| **A** | Availability | প্রতিটি request **সবসময় সাড়া** পায় (error নয়) |
| **P** | Partition Tolerance | node-গুলোর মধ্যে নেটওয়ার্ক বিচ্ছিন্ন হলেও system **চলতে থাকে** |

> ⚠️ CAP-এর **C**, ACID-এর **C** নয়। CAP-এর C মানে সব replica-র data মিল; ACID-এর C মানে constraint মানা।

---

## ২. কেন "দুটি বেছে নাও"?

বাস্তবে নেটওয়ার্ক ব্যর্থ হয় — তাই **P বাধ্যতামূলক**। আসল সিদ্ধান্ত: **partition হলে C বেছে নেবেন, না A?**

### উদাহরণ

দুটি server (A ও B) একই data রাখে। দুজনের মাঝের নেটওয়ার্ক কেটে গেল। এখন একজন user A-তে balance বদলালেন, আরেকজন B থেকে পড়ছেন।

```
 [Server A] ✂️ নেটওয়ার্ক বিচ্ছিন্ন ✂️ [Server B]
  balance=৫০০ (নতুন)                     balance=১০০০ (পুরনো)
```

- **CP পছন্দ:** B বলবে "এখন উত্তর দিতে পারছি না" (error) → data **ভুল হবে না**, কিন্তু **available নয়**
- **AP পছন্দ:** B পুরনো `১০০০` দেখাবে → **available**, কিন্তু data **পুরনো/অসঙ্গত**

---

## ৩. CP বনাম AP

| | CP (Consistency + Partition) | AP (Availability + Partition) |
|---|---|---|
| Partition-এ আচরণ | কিছু request বন্ধ/error | সাড়া দেয়, কিন্তু পুরনো data হতে পারে |
| Consistency | Strong | **Eventual** |
| উপযোগী | ব্যাংক, inventory, booking | Social feed, like count, cache, catalog |
| উদাহরণ | HBase, MongoDB (default), etcd, ZooKeeper, SQL Server (single primary) | Cassandra, DynamoDB (default), CouchDB |

### সিদ্ধান্ত নেবেন কীভাবে?
প্রশ্ন করুন: **"ভুল/পুরনো data দেখানো খারাপ, নাকি সাড়া না দেওয়া খারাপ?"**

- ব্যাংক balance: ভুল দেখানোর চেয়ে সাময়িক error ভালো → **CP**
- Facebook like count: ১০টি কম দেখালে ক্ষতি নেই, কিন্তু site বন্ধ থাকা ক্ষতি → **AP**

---

## ৪. Eventual Consistency

AP system-এ data **তাৎক্ষণিক নয়, কিছুক্ষণ পরে** সব node-এ মিলে যায়।

```
সময় ০:  Node A = ৫০০,  Node B = ১০০০   (অসঙ্গত)
সময় ২:  Node A = ৫০০,  Node B = ৫০০    (মিলে গেছে ✅)
```

### সংঘাত (conflict) সমাধানের কৌশল
- **Last Write Wins** — শেষ timestamp জেতে
- **Vector clock / CRDT** — আরো নিখুঁত merge
- **Application-level merge** — business logic দিয়ে

---

## ৫. PACELC — CAP-এর বাস্তবসম্মত সম্প্রসারণ

CAP শুধু **partition** হলে প্রযোজ্য। কিন্তু স্বাভাবিক সময়েও একটি লেনদেন আছে:

> **P**artition হলে → **A** না **C**? ; **E**lse (স্বাভাবিক সময়ে) → **L**atency না **C**onsistency?

- দ্রুত সাড়া চাইলে → replica থেকে পুরনো data আসতে পারে (Latency ↑, Consistency ↓)
- Strong consistency চাইলে → সব replica-র সাথে মিলিয়ে সাড়া দিতে সময় বেশি লাগে

---

## ৬. বাস্তব প্রকল্পে প্রয়োগ

### Microservices-এ
প্রতিটি service-এর নিজস্ব DB → service-গুলোর মধ্যে সাধারণত **eventual consistency**। তাই:
- **Saga Pattern** (ক্ষতিপূরণমূলক action)
- **Outbox Pattern**
- UI-তে "প্রক্রিয়াধীন" অবস্থা দেখানো

### Database Replication-এ
- Primary-তে write, Replica থেকে read → replica **কিছুটা পিছিয়ে** থাকতে পারে
- "নিজের লেখা সাথে সাথে পড়া" দরকার হলে (read-your-writes) সেই request primary-তে পাঠান

### একই system-এ মিশ্র সিদ্ধান্ত ⭐
সব কিছুতে একই পছন্দ লাগে না:

| ডেটা | পছন্দ |
|---|---|
| Payment/Balance | CP / Strong consistency |
| Product catalog, search | AP / Eventual |
| Session/Cache | AP |

---

## ৭. সাধারণ ভুল ধারণা

| ভুল ধারণা | বাস্তব |
|---|---|
| "CAP-এর যেকোনো ২টি বেছে নেওয়া যায়" | P এড়ানো যায় না; আসল পছন্দ C বনাম A |
| "AP মানে data ভুল" | সাময়িক অসঙ্গত, পরে ঠিক হয় |
| "NoSQL = AP, SQL = CP" | নির্ভর করে configuration ও deployment-এর উপর |
| "CAP-এর C = ACID-এর C" | ভিন্ন অর্থ |
| "সবসময়ই partition ঘটে" | বিরল, কিন্তু ঘটলে সিদ্ধান্ত আগেই থাকতে হয় |

---

## ✅ সারসংক্ষেপ

- Partition অনিবার্য → পছন্দ **Consistency বনাম Availability**
- **CP** = নির্ভুলতা আগে | **AP** = সবসময় সাড়া, eventual consistency
- ডেটার গুরুত্ব অনুযায়ী আলাদা আলাদা সিদ্ধান্ত নিন

⬅️ [CQRS](08-cqrs.md) | ➡️ [Sharding ও Partitioning](10-sharding-and-partitioning.md)
