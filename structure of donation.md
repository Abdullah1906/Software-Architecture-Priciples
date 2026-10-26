 পুরো Project-কে ছোট ছোট Step-এ ভাগ করা
আমরা প্রতিটি Step শেষ করব, build/test করব, তারপর পরের Step-এ যাব। একসঙ্গে অনেকগুলো module লিখব না।

Phase 1 — Project Foundation
প্রথমে এটি
- Step 1.1 — Solution ও project structure
- Step 1.2 — Clean Architecture reference
- Step 1.3 — Dependency Injection ও configuration
- Step 1.4 — API response, error handling ও validation
- Step 1.5 — Swagger, health check ও API versioning
- Step 1.6 — Unit testing setup




Phase 2 — Database & ADO.NET
- Step 2.1 — Database ও schema design
- Step 2.2 — Tables, primary/foreign keys ও constraints
- Step 2.3 — Indexes, audit columns ও rowversion
- Step 2.4 — Stored procedure conventions
- Step 2.5 — ADO.NET connection factory
- Step 2.6 — Transaction ও repository implementation
- Step 2.7 — Database script versioning ও deployment




Phase 3 — Authentication & Multi-Tenancy
- Step 3.1 — Organization/Institution registration
- Step 3.2 — User, role ও permission
- Step 3.3 — JWT ও refresh-token rotation
- Step 3.4 — Organization isolation
- Step 3.5 — Tenant-scoped authorization tests
- Step 3.6 — API key ও rate limiting




Phase 4 — Donation Core
- Step 4.1 — Donation categories: Zakat, Sadqah, Fitra, General
- Step 4.2 — Fund management: Building, Orphan, Lillah, Qurbani
- Step 4.3 — Campaign ও target amount
- Step 4.4 — Donor profile
- Step 4.5 — Donation create API
- Step 4.6 — Anonymous donation
- Step 4.7 — Donation history ও pagination
- Step 4.8 — Recurring monthly donation
- Step 4.9 — Unique donation/transaction reference




Phase 5 — Payment Integration
- Step 5.1 — Payment provider abstraction
- Step 5.2 — bKash integration
- Step 5.3 — Nagad/Rocket integration
- Step 5.4 — SSLCommerz বা aamarPay
- Step 5.5 — Stripe/PayPal integration
- Step 5.6 — Currency ও exchange-rate management
- Step 5.7 — Verified webhook ও duplicate-event protection
- Step 5.8 — Manual bank deposit ও slip verification
- Step 5.9 — Payment reconciliation




Phase 6 — Receipt & Notification
- Step 6.1 — PDF donation receipt
- Step 6.2 — Email notification
- Step 6.3 — SMS notification
- Step 6.4 — Receipt verification ও download
- Step 6.5 — Notification retry ও background processing




Phase 7 — Accounting & Fund Management
- Step 7.1 — Chart of Accounts
- Step 7.2 — Income/expense ledger
- Step 7.3 — Double-entry accounting
- Step 7.4 — Bank accounts ও cash-in-hand
- Step 7.5 — Fund-wise balance tracking
- Step 7.6 — Expense category ও voucher upload
- Step 7.7 — Entry → Review → Approval workflow
- Step 7.8 — Budget বনাম actual expense
- Step 7.9 — Donation settlement reconciliation




Phase 8 — Reports & Transparency
- Step 8.1 — Daily/monthly/yearly reports
- Step 8.2 — Fund-wise income ও expense
- Step 8.3 — Balance sheet ও fund statement
- Step 8.4 — Public transparency dashboard
- Step 8.5 — Fund target/progress bar
- Step 8.6 — PDF/Excel export
- Step 8.7 — Report pagination ও filtering




Phase 9 — Audit, Security & Reliability
- Step 9.1 — Centralized audit logging
- Step 9.2 — Who changed what, when ও before/after values
- Step 9.3 — Immutable financial records ও correction entries
- Step 9.4 — Concurrency handling ও rowversion
- Step 9.5 — Idempotency ও duplicate payment protection
- Step 9.6 — Security, integration ও load tests




Phase 10 — Angular 20 Frontend
- Step 10.1 — Angular project ও folder structure
- Step 10.2 — Login, roles ও route guards
- Step 10.3 — Public donation page
- Step 10.4 — Donation form ও payment flow
- Step 10.5 — Donor profile/history
- Step 10.6 — Admin fund/donation management
- Step 10.7 — Expense approval ও accounting UI
- Step 10.8 — Dashboard, charts ও reports
- Step 10.9 — Pagination, filters ও export




Phase 11 — Production Deployment
- Step 11.1 — Docker ও environment configuration
- Step 11.2 — Secrets management
- Step 11.3 — HTTPS ও reverse proxy
- Step 11.4 — Database backup/restore test
- Step 11.5 — Logging, metrics ও alerts
- Step 11.6 — CI/CD pipeline
- Step 11.7 — Production readiness checklist



৩. Professional-grade design-এর বাধ্যতামূলক নিয়ম
বিষয়	আমাদের implementation
Architecture	Domain → Application → Infrastructure → API
Database access	ADO.NET + stored procedures
Pagination	OFFSET/FETCH, maximum page size, total count
ACID	Financial operations-এ database transactions
Audit log	Actor, action, timestamp, entity ও change details
Multi-tenancy	প্রতিটি organization-এর data আলাদা authorization scope-এ
Payment	Server-side verification; শুধু redirect দেখে Paid নয়
Duplicate protection	Idempotency key, unique constraints, webhook event tracking
Fund accounting	Fund-wise ledger এবং অনুমোদিত transfer/expense
Security	Least privilege, validation, rate limiting, secure secrets
Testing	Unit tests, integration tests ও payment failure scenarios

বিশেষ করে Donation এবং Accounting আলাদা bounded module হবে। Donation সফল হলেই টাকা সরাসরি খরচের হিসাবে দেখানো যাবে না; payment settlement, fund allocation এবং accounting entry-র মধ্যে স্পষ্ট সম্পর্ক রাখতে হবে। Zakat-এর মতো restricted fund অন্য fund-এ ব্যবহার হয়েছে কি না, সেটিও ledger ও approval rules দিয়ে যাচাই করা যাবে।
৪. API কীভাবে অন্য প্রতিষ্ঠান ব্যবহার করবে?
মাদ্রাসা / NGO / ওয়েবসাইট

Angular, React, WordPress বা অন্য client


Donation REST API

Authentication · Tenant isolation · Validation · Versioning


Application & Domain

Donation rules · Payment workflow · Fund ledger


ADO.NET + SQL Server

Stored procedures · ACID · Audit · Indexes


উদাহরণ হিসেবে একটি client donation তৈরি করতে পারে:
POST /api/v1/donations
Authorization: Bearer <access_token>
Idempotency-Key: 8d4c...unique-key
Content-Type: application/json


{
  "fundId": 12,
  "amount": 1000.00,
  "currency": "BDT",
  "donationType": "Sadqah",
  "isAnonymous": false
}


এটি একটি illustrative contract; actual request DTO ও payment lifecycle আমরা পরবর্তী ধাপে নির্ধারণ করব। টাকা নেওয়ার ক্ষেত্রে client-এর request-কে একাই payment success হিসেবে বিশ্বাস করা হবে না।
৫. এখন থেকে কীভাবে কাজ করব?
প্রতিটি Step-এ থাকবে:
1. কেন এই কাজ প্রয়োজন এবং architecture decision।
2. Visual Studio-তে solution/project তৈরি বা পরিবর্তনের নির্দেশনা।
3. সম্পূর্ণ file path এবং copy-paste করার মতো code।
4. SQL Server-এর সম্পূর্ণ script, যেখানে প্রয়োজন।
5. Swagger/SSMS দিয়ে কীভাবে test করবে।
6. Expected result, error handling এবং পরবর্তী Step-এর dependency।
আমরা শুরু করব Phase 1, Step 1.1 — Donation API Solution Structure দিয়ে। প্রাথমিকভাবে .NET 8 LTS, ASP.NET Core Web API, ADO.NET এবং SQL Server ধরে এগোনো যায়; version চূড়ান্ত করার আগে নতুন project-এর deployment target অনুযায়ী যাচাই করা উচিত।

একটি নাম প্রস্তাব করছি: DonationPlatform.sln এবং project-গুলো হবে Donation.Domain, Donation.Application, Donation.Infrastructure, Donation.Api, Donation.UnitTests ও Donation.IntegrationTests।

পরবর্তী ধাপে এই solution-এর folder structure, Visual Studio project creation, project references এবং build verification সম্পূর্ণ করব।
