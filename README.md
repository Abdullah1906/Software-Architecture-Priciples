# 📚 Backend Architecture & Security — Professional Series (বাংলায়)

এই সিরিজে backend development-এর গুরুত্বপূর্ণ ধারণাগুলো সহজ বাংলায় ব্যাখ্যা করা হয়েছে — **কী**, **কেন**, **কীভাবে কাজ করে**, **কোন পরিস্থিতিতে লাগে**, এবং **professional project-এ কীভাবে ব্যবহার হয়**।

কোড উদাহরণগুলো **ASP.NET Core (C#) + SQL Server** দিয়ে দেওয়া হয়েছে।

---

## 📖 Chapter সূচি

### 🧱 Part ১: মৌলিক ভিত্তি

| Chapter | বিষয় | ফাইল |
|---|---|---|
| ০১ | REST API | [01-rest-api.md](01-rest-api.md) |
| ০২ | ACID (Database Transaction) | [02-acid.md](02-acid.md) |
| ০৩ | DDD (Domain-Driven Design) | [03-ddd.md](03-ddd.md) |
| ০৪ | REST + ACID + DDD একসাথে | [04-putting-it-together.md](04-putting-it-together.md) |

### 🏗️ Part ২: Design ও Architecture

| Chapter | বিষয় | ফাইল |
|---|---|---|
| ০৫ | SOLID, DRY, KISS, YAGNI | [05-design-principles.md](05-design-principles.md) |
| ০৬ | Monolithic বনাম Microservices | [06-monolith-vs-microservices.md](06-monolith-vs-microservices.md) |
| ০৭ | Message Queue ও Event-Driven Architecture | [07-message-queue-and-eda.md](07-message-queue-and-eda.md) |
| ০৮ | CQRS | [08-cqrs.md](08-cqrs.md) |

### 📈 Part ৩: Data ও Scalability

| Chapter | বিষয় | ফাইল |
|---|---|---|
| ০৯ | CAP Theorem | [09-cap-theorem.md](09-cap-theorem.md) |
| ১০ | Database Sharding ও Partitioning | [10-sharding-and-partitioning.md](10-sharding-and-partitioning.md) |
| ১১ | Caching & Read Replica | [11-caching.md](11-caching.md) |

### 🔌 Part ৪: API ও Delivery

| Chapter | বিষয় | ফাইল |
|---|---|---|
| ১২ | gRPC ও GraphQL | [12-grpc-and-graphql.md](12-grpc-and-graphql.md) |
| ১৩ | CI/CD | [13-ci-cd.md](13-ci-cd.md) |

### 🔐 Part ৫: Security

| Chapter | বিষয় | ফাইল |
|---|---|---|
| ১৪ | HTTPS ও TLS | [14-https-and-tls.md](14-https-and-tls.md) |
| ১৫ | JWT ও OAuth 2.0 | [15-jwt-and-oauth2.md](15-jwt-and-oauth2.md) |
| ১৬ | SQL Injection ও XSS Protection | [16-sql-injection-and-xss.md](16-sql-injection-and-xss.md) |

---

## 🎯 কীভাবে পড়বেন

1. **Part ১** দিয়ে শুরু করুন — API, Transaction ও Domain modeling-এর ভিত্তি
2. **Part ২** — কোড ও system কীভাবে সাজাবেন
3. **Part ৩** — data বাড়লে কীভাবে scale করবেন
4. **Part ৪** — API-র বিকল্প ও স্বয়ংক্রিয় deployment
5. **Part ৫** — নিরাপত্তা (প্রতিটি প্রকল্পের জন্য আবশ্যক)

> 💡 সময় কম থাকলে Part ১ ও Part ৫ আগে পড়ুন।

## 🧰 Prerequisites

- C# ও ASP.NET Core-এর basic ধারণা
- SQL (SELECT, INSERT, UPDATE) জানা
- Git ও HTTP-র প্রাথমিক ধারণা

## 📝 Note

- প্রতিটি chapter-এর শেষে **Best Practices** ও **সাধারণ ভুল** আছে — interview-এর আগে ঝালাই করার জন্য
- এই ধারণাগুলো **সব প্রকল্পে সব একসাথে লাগে না**। ছোট প্রকল্পে সরল রাখুন (KISS, YAGNI) এবং প্রয়োজন অনুযায়ী যোগ করুন

## 🤝 Contribution

ভুল পেলে বা উন্নতির পরামর্শ থাকলে Issue/Pull Request করুন।

## 📜 License

MIT
