# Chapter ১২: gRPC ও GraphQL

REST-এর বিকল্প বা পরিপূরক দুটি জনপ্রিয় API পদ্ধতি। দুটির সমস্যা ও লক্ষ্য আলাদা।

**Topics:** gRPC, GraphQL, REST API, ASP.NET Core, Microservices, Clean Architecture, Security, Performance

---

## 📚 Table of Contents

- [1. Introduction](#1-introduction)
- [2. What is gRPC?](#2-what-is-grpc)
- [3. How gRPC Works](#3-how-grpc-works)
- [4. Build a gRPC Service in .NET 8](#4-build-a-grpc-service-in-net-8)
- [5. What is GraphQL?](#5-what-is-graphql)
- [6. Build a GraphQL API in ASP.NET Core](#6-build-a-graphql-api-in-aspnet-core)
- [7. REST vs gRPC vs GraphQL](#7-rest-vs-grpc-vs-graphql)
- [8. Professional Architecture](#8-professional-architecture)
- [9. Security and Performance](#9-security-and-performance)
- [10. Real-World Use Cases](#10-real-world-use-cases)
- [11. Best Practices](#11-best-practices)
- [12. Learning Roadmap](#12-learning-roadmap)
- [13. Official Documentation](#13-official-documentation)

---

## 1. Introduction

আধুনিক Software Development-এ একাধিক application, service এবং frontend-এর মধ্যে communication করার জন্য বিভিন্ন API technology ব্যবহার করা হয়।

সবচেয়ে পরিচিত একটি technology হলো REST API। তবে কিছু ক্ষেত্রে gRPC অথবা GraphQL আরও কার্যকর হতে পারে।

### সহজ উদাহরণ

ধরো, তুমি একটি Madrasa Donation Platform তৈরি করছ।

- **Angular 20:** Frontend
- **ASP.NET Core 8 Web API:** Backend
- **SQL Server:** Database
- **gRPC:** Internal microservice communication
- **GraphQL:** Frontend-এর প্রয়োজন অনুযায়ী একাধিক resource থেকে data সংগ্রহ

মনে রাখতে হবে, একই application-এ REST, gRPC ও GraphQL ব্যবহার করা সম্ভব। তবে প্রত্যেকটির নির্দিষ্ট দায়িত্ব থাকা উচিত।

---

## 2. What is gRPC?

**gRPC (Google Remote Procedure Call)** হলো একটি high-performance communication framework, যার মাধ্যমে একটি application অন্য application-এর method দূর থেকে call করার মতো করে ব্যবহার করতে পারে।

সহজ ভাষায়, একটি service অন্য service-কে request পাঠিয়ে কাজ করাতে পারে।

উদাহরণ:

```text
Donation API
     |
     | gRPC call
     v
Payment Service
     |
     v
Payment Result
```

Donation API সরাসরি Payment Service-এর internal method call করছে না। বরং network-এর মাধ্যমে একটি নির্ধারিত gRPC contract অনুসারে request পাঠাচ্ছে।

### gRPC-এর প্রধান বৈশিষ্ট্য

- HTTP/2-এর ওপর communication।
- Protocol Buffers (Protobuf) দিয়ে compact binary message serialization।
- `.proto` file থেকে strongly typed client ও server code generation।
- Unary, server streaming, client streaming ও bidirectional streaming।
- C#, Java, Go, Python-সহ বিভিন্ন programming language support।
- Authentication, authorization, logging ও dependency injection-এর সঙ্গে integration।

Official documentation: [Microsoft Learn — gRPC](https://learn.microsoft.com/aspnet/core/grpc/)

### কেন gRPC ব্যবহার করব?

ধরো, তোমার system-এ পাঁচটি microservice আছে:

```text
API Gateway
   |
   +----> User Service
   |
   +----> Donation Service
   |
   +----> Payment Service
   |
   +----> Notification Service
   |
   +----> Reporting Service
```

এই service-গুলোর মধ্যে ঘন ঘন communication হলে gRPC উপকারী হতে পারে।

**গুরুত্বপূর্ণ:** gRPC মানেই সব পরিস্থিতিতে REST-এর চেয়ে দ্রুত নয়। Performance নির্ভর করে payload, network latency, connection reuse, serialization, database এবং workload-এর ওপর।

---

## 3. How gRPC Works

gRPC-এর communication সাধারণত চারটি ধাপে হয়।

1. `.proto` file-এ service ও message contract define করা হয়।
2. Code generator থেকে client/server types তৈরি হয়।
3. Server generated base class implement করে।
4. Client generated method call করে এবং server response দেয়।

### Example: Greeting Service

**File:** `Protos/greet.proto`

```protobuf
syntax = "proto3";

option csharp_namespace = "GrpcGreeter";

package greet;

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

এখানে:

- `Greeter` হলো service।
- `SayHello` হলো remote procedure।
- `HelloRequest` হলো request message।
- `HelloReply` হলো response message।
- `name = 1` হলো field number, যা Protobuf wire format-এ ব্যবহৃত হয়।

Field number পরিবর্তন বা reuse করলে backward compatibility নষ্ট হতে পারে। তাই production-এ field number স্থিতিশীল রাখতে হবে।

### Communication Flow

```text
Client Application
       |
       | SayHello("Abdullah")
       v
Generated gRPC Client
       |
       | HTTP/2 + Protobuf
       v
GreeterService
       |
       v
HelloReply("Hello Abdullah")
```

---

## 4. Build a gRPC Service in .NET 8

### Step 1: Create the Project

Visual Studio 2022 ব্যবহার করলে:

1. `Create a new project` নির্বাচন করো।
2. `ASP.NET Core gRPC Service` নির্বাচন করো।
3. Project name দাও `GrpcGreeter`।
4. Framework হিসেবে `.NET 8.0` নির্বাচন করো।
5. Project তৈরি করো।

.NET CLI ব্যবহার করলে:

```bash
dotnet new grpc -o GrpcGreeter -f net8.0
```

তারপর:

```bash
cd GrpcGreeter
dotnet restore
```

### Step 2: Understand the Project Structure

```text
GrpcGreeter/
├── Protos/
│   └── greet.proto
├── Services/
│   └── GreeterService.cs
├── Program.cs
└── GrpcGreeter.csproj
```

### Step 3: Define the Contract

`Protos/greet.proto` ফাইলের content হবে:

```protobuf
syntax = "proto3";

option csharp_namespace = "GrpcGreeter";

package greet;

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

### Step 4: Implement the Service

**File:** `Services/GreeterService.cs`

```csharp
using Grpc.Core;

namespace GrpcGreeter.Services;

public sealed class GreeterService(
    ILogger<GreeterService> logger)
    : Greeter.GreeterBase
{
    public override Task<HelloReply> SayHello(
        HelloRequest request,
        ServerCallContext context)
    {
        logger.LogInformation(
            "Greeting requested for {Name}",
            request.Name);

        return Task.FromResult(new HelloReply
        {
            Message = $"Hello {request.Name}"
        });
    }
}
```

এখানে `Greeter.GreeterBase` generated base class। Method signature `.proto` contract থেকে তৈরি হয়।

### Step 5: Configure the Server

**File:** `Program.cs`

```csharp
using GrpcGreeter.Services;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddGrpc();

var app = builder.Build();

app.MapGrpcService<GreeterService>();

app.MapGet("/", () =>
    "This server exposes a gRPC endpoint.");

app.Run();
```

### Step 6: Verify the Project Configuration

`GrpcGreeter.csproj` ফাইলে gRPC service এবং Protobuf tooling-এর configuration থাকা প্রয়োজন। Template ব্যবহার করলে এগুলো সাধারণত আগে থেকেই থাকে।

```xml
<ItemGroup>
  <PackageReference Include="Grpc.AspNetCore"
                    Version="2.*" />
</ItemGroup>

<ItemGroup>
  <Protobuf Include="Protos\greet.proto"
            GrpcServices="Server" />
</ItemGroup>
```

**Note:** এটি configuration বোঝানোর উদাহরণ। Production project-এ NuGet package-এর নির্দিষ্ট compatible version pin করবে; `2.*`-কে production version হিসেবে ব্যবহার করবে না। Template-এর বিদ্যমান package reference অকারণে duplicate করবে না।

### Step 7: Run the Application

```bash
dotnet run
```

Server চালু হলে terminal-এ HTTPS endpoint দেখাবে।

Browser-এ root URL খুললে সাধারণ text পাওয়া যেতে পারে। এটি স্বাভাবিক; gRPC endpoint সাধারণ browser-এর address bar দিয়ে সাধারণ REST endpoint-এর মতো test করা যায় না।

gRPC call করার জন্য একটি compatible client প্রয়োজন।

### Step 8: Create a .NET gRPC Client

একটি Console App তৈরি করো:

```bash
dotnet new console -o GrpcGreeterClient -f net8.0
```

Client project-এ package install করো:

```bash
dotnet add GrpcGreeterClient.csproj package Grpc.Net.Client
dotnet add GrpcGreeterClient.csproj package Google.Protobuf
dotnet add GrpcGreeterClient.csproj package Grpc.Tools
```

Client `.csproj`-এ server-এর একই `.proto` contract reference করো:

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto"
            GrpcServices="Client" />
</ItemGroup>
```

`Protos/greet.proto` ফাইলটি client project-এ কপি করতে হবে।

**File:** `Program.cs`

```csharp
using Grpc.Net.Client;
using GrpcGreeter;

using var channel = GrpcChannel.ForAddress(
    "https://localhost:7001");

var client = new Greeter.GreeterClient(channel);

var response = await client.SayHelloAsync(
    new HelloRequest
    {
        Name = "Abdullah"
    });

Console.WriteLine(response.Message);
```

`7001`-এর জায়গায় server-এর প্রকৃত HTTPS port ব্যবহার করবে। Server ও client-এর generated contract এবং namespace অবশ্যই মিলতে হবে।

Expected output:

```text
Hello Abdullah
```

Development certificate trusted না হলে certificate trust configuration ঠিক করতে হবে। Production-এ certificate validation বন্ধ করা যাবে না।

**Official tutorial:** [Create a .NET gRPC client and server](https://learn.microsoft.com/aspnet/core/tutorials/grpc/grpc-start?view=aspnetcore-8.0)

---

## 5. What is GraphQL?

**GraphQL হলো API-এর জন্য একটি query language এবং runtime**, যার মাধ্যমে client নির্দিষ্ট করে বলতে পারে সে কোন fields এবং related data চায়।

REST API-তে সাধারণত বিভিন্ন resource-এর জন্য একাধিক endpoint থাকে। GraphQL-এ একটি endpoint থেকে client নিজের প্রয়োজন অনুযায়ী data query করতে পারে।

### Example

ধরো, একটি Donation Platform-এ donation-এর সঙ্গে donor এবং payment information দেখাতে হবে।

REST API:

```http
GET /api/donations/101
GET /api/donors/15
GET /api/payments/501
```

এই তিনটি endpoint প্রয়োজন হতে পারে, যদি প্রথম response-এ related data না থাকে।

GraphQL query:

```graphql
query {
  donation(id: 101) {
    id
    amount
    status
    donor {
      name
    }
    payment {
      method
      status
    }
  }
}
```

Client শুধু প্রয়োজনীয় fields চায়।

Possible response:

```json
{
  "data": {
    "donation": {
      "id": 101,
      "amount": 5000,
      "status": "Completed",
      "donor": {
        "name": "Abdullah"
      },
      "payment": {
        "method": "bKash",
        "status": "Paid"
      }
    }
  }
}
```

এটি illustrative response। প্রকৃত result schema, database data এবং authorization-এর ওপর নির্ভর করবে।

### GraphQL-এর মূল concepts

| Concept | কাজ |
|---|---|
| Schema | API-এর available types ও operations define করে |
| Query | Data read করে |
| Mutation | Data create/update/delete করে |
| Resolver | Field বা operation-এর data resolve করে |
| Subscription | Supported transport-এ ongoing event update দেয় |
| Type | Field-এর structure ও data type নির্ধারণ করে |

### GraphQL কেন ব্যবহার হয়?

- Client-এর প্রয়োজন অনুযায়ী fields নির্বাচন।
- Related data এক query-তে fetch করা।
- বিভিন্ন frontend-এর জন্য flexible data access।
- Strongly typed schema এবং introspection-based tooling।
- Mobile application-এ অপ্রয়োজনীয় data transfer কমানোর সুযোগ।

**সতর্কতা:** GraphQL ব্যবহার করলেই network request বা database query স্বয়ংক্রিয়ভাবে কমে যায় না। Resolver design খারাপ হলে N+1 query সমস্যা, অতিরিক্ত database load এবং resource exhaustion হতে পারে।

Official documentation: [GraphQL .NET](https://graphql-dotnet.github.io/docs/guides/aspnetcore/)

---

## 6. Build a GraphQL API in ASP.NET Core

এই উদাহরণে GraphQL-এর basic concepts বোঝানো হয়েছে। Production application-এ query resolver-এর মাধ্যমে Application Service বা Repository call করবে; সরাসরি database access নয়।

### Step 1: Create the Project

Visual Studio 2022-এ `ASP.NET Core Web API` project তৈরি করো।

Project name:

```text
DonationGraphQL.Api
```

Framework:

```text
.NET 8.0
```

### Step 2: Install GraphQL Packages

GraphQL.NET ব্যবহার করে এই example তৈরি করা হবে।

```bash
dotnet add package GraphQL
dotnet add package GraphQL.SystemTextJson
```

Package versions তোমার target framework এবং dependency compatibility অনুযায়ী pin করবে।

### Step 3: Create a Model

**File:** `Models/Donation.cs`

```csharp
namespace DonationGraphQL.Api.Models;

public sealed class Donation
{
    public int Id { get; init; }

    public decimal Amount { get; init; }

    public string Status { get; init; } = string.Empty;
}
```

### Step 4: Create the Query

**File:** `GraphQL/DonationQuery.cs`

```csharp
using DonationGraphQL.Api.Models;
using GraphQL.Types;

namespace DonationGraphQL.Api.GraphQL;

public sealed class DonationQuery : ObjectGraphType
{
    public DonationQuery()
    {
        Field<ListGraphType<DonationType>>("donations")
            .Resolve(_ => new List<Donation>
            {
                new()
                {
                    Id = 1,
                    Amount = 5000,
                    Status = "Completed"
                },
                new()
                {
                    Id = 2,
                    Amount = 2500,
                    Status = "Pending"
                }
            });
    }
}

public sealed class DonationType : ObjectGraphType<Donation>
{
    public DonationType()
    {
        Field(x => x.Id);
        Field(x => x.Amount);
        Field(x => x.Status);
    }
}
```

এই উদাহরণে static sample data ব্যবহার করা হয়েছে, যাতে GraphQL query এবং type-এর কাজ সহজে বোঝা যায়। এটি production database implementation নয়।

### Step 5: Register the GraphQL Schema

**File:** `GraphQL/DonationSchema.cs`

```csharp
using GraphQL.Types;

namespace DonationGraphQL.Api.GraphQL;

public sealed class DonationSchema : Schema
{
    public DonationSchema(IServiceProvider provider)
        : base(provider)
    {
        Query = provider.GetRequiredService<DonationQuery>();
    }
}
```

### Step 6: Configure Program.cs

**File:** `Program.cs`

```csharp
using DonationGraphQL.Api.GraphQL;
using GraphQL;
using GraphQL.Types;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSingleton<DonationQuery>();

builder.Services.AddSingleton<ISchema, DonationSchema>();

builder.Services.AddGraphQL(options =>
    options.AddSystemTextJson());

var app = builder.Build();

app.UseGraphQL<ISchema>("/graphql");

app.MapGet("/", () =>
    "Donation GraphQL API is running.");

app.Run();
```

GraphQL.NET-এর package version অনুযায়ী middleware ও DI configuration-এ সামান্য পার্থক্য থাকতে পারে। Build error হলে installed version-এর official guide অনুসরণ করবে।

### Step 7: Send a GraphQL Query

GraphQL endpoint:

```http
POST /graphql
Content-Type: application/json
```

Request body:

```json
{
  "query": "{ donations { id amount status } }"
}
```

Expected response:

```json
{
  "data": {
    "donations": [
      {
        "id": 1,
        "amount": 5000,
        "status": "Completed"
      },
      {
        "id": 2,
        "amount": 2500,
        "status": "Pending"
      }
    ]
  }
}
```

GraphQL client হিসেবে GraphiQL, GraphQL IDE বা Postman ব্যবহার করা যায়। IDE চালু করতে চাইলে server configuration এবং compatible UI package আলাদাভাবে সেট করতে হবে।

### Step 8: Connect SQL Server Professionally

Production-এ static list বাদ দিয়ে Application Service ব্যবহার করবে:

```text
GraphQL Resolver
      |
      v
Application Service
      |
      v
Repository Interface
      |
      v
ADO.NET Repository
      |
      v
Stored Procedure
      |
      v
SQL Server
```

তোমার project-এ যদি ADO.NET ও stored procedure ব্যবহার করা হয়, তাহলে GraphQL resolver-এর ভেতরে SQL query না লিখে existing service/repository layer reuse করবে।

---

## 7. REST vs gRPC vs GraphQL

| Feature | REST API | gRPC | GraphQL |
|---|---|---|---|
| Main purpose | Resource-based API | Service-to-service RPC | Flexible data querying |
| Common protocol | HTTP | HTTP/2 | Usually HTTP |
| Data format | Usually JSON | Protobuf binary by default | Usually JSON response |
| Contract | OpenAPI optional | `.proto` contract | GraphQL schema |
| Browser support | Excellent | Native gRPC সীমিত; gRPC-Web দরকার হতে পারে | Excellent |
| Streaming | SSE/WebSockets ইত্যাদি | Built-in streaming patterns | Subscription implementation অনুযায়ী |
| Best fit | Public APIs, CRUD | Internal microservices, streaming | Frontend-specific data requirements |
| Main trade-off | Over-fetching/under-fetching হতে পারে | Tooling ও transport requirements | Resolver complexity ও query cost |

### কোনটি কখন ব্যবহার করব?

**REST API বেছে নাও যখন:**

- Public API তৈরি করছ।
- Standard CRUD endpoint প্রয়োজন।
- Angular বা third-party client-কে সহজ integration দিতে হবে।
- HTTP caching, status codes এবং OpenAPI documentation গুরুত্বপূর্ণ।

**gRPC বেছে নাও যখন:**

- Internal microservices-এর মধ্যে frequent communication আছে।
- Strongly typed contract প্রয়োজন।
- Streaming দরকার।
- Low-latency, high-throughput communication একটি গুরুত্বপূর্ণ requirement।

**GraphQL বেছে নাও যখন:**

- UI-তে অনেক related resource একসঙ্গে দেখাতে হয়।
- Mobile এবং web client-এর data requirement আলাদা।
- Client-এর fields নির্বাচনের flexibility প্রয়োজন।
- Schema-based frontend/backend collaboration গুরুত্বপূর্ণ।

একই application-এ তিনটি technology-ই থাকতে পারে। Technology নির্বাচন করবে measured requirements অনুযায়ী, শুধু popularity দেখে নয়।

---

## 8. Professional Architecture

একটি maintainable .NET application-এ API protocol-কে business logic থেকে আলাদা রাখা উচিত।

### Example Architecture

```text
                    Angular 20
                         |
                  REST / GraphQL
                         |
                 ASP.NET Core API
                         |
                 Application Layer
                 /               \
         Domain Layer       Service Contracts
                                  |
                         Infrastructure Layer
                                  |
                       ADO.NET / Repositories
                                  |
                              SQL Server

Internal Service Communication:
Donation Service -------- gRPC -------- Payment Service
```

### Recommended Project Structure

```text
DonationPlatform/
├── DonationPlatform.Domain/
│   ├── Entities/
│   └── ValueObjects/
├── DonationPlatform.Application/
│   ├── Interfaces/
│   ├── Services/
│   ├── DTOs/
│   └── Validators/
├── DonationPlatform.Infrastructure/
│   ├── Repositories/
│   ├── Persistence/
│   └── GrpcClients/
├── DonationPlatform.Api/
│   ├── Controllers/
│   ├── GraphQL/
│   ├── Middleware/
│   └── Program.cs
└── DonationPlatform.Contracts/
    └── Protos/
        └── payment.proto
```

এই structure একটি illustrative example। সব project-এ GraphQL, REST এবং gRPC রাখার প্রয়োজন নেই।

### Professional gRPC Contract Example

`payment.proto`:

```protobuf
syntax = "proto3";

option csharp_namespace = "Payment.Contracts";

package payment;

service PaymentService {
  rpc GetPaymentStatus (PaymentStatusRequest)
      returns (PaymentStatusResponse);
}

message PaymentStatusRequest {
  string payment_id = 1;
}

message PaymentStatusResponse {
  string payment_id = 1;
  string status = 2;
}
```

এই contract-এর মাধ্যমে Donation Service payment status জানতে পারবে।

Production implementation-এ অবশ্যই নিচের বিষয়গুলো যোগ করতে হবে:

- Authentication এবং authorization।
- Deadline ও cancellation handling।
- Structured logging এবং tracing।
- Transient failure-এর জন্য সীমিত retry।
- Error status mapping।
- Contract compatibility tests।
- Service health checks।
- Network failure-এর জন্য timeout এবং resilience policy।

### Professional GraphQL Design

GraphQL resolver-কে thin রাখবে।

```text
GraphQL Query
     |
     v
Donation Query Resolver
     |
     v
IDonationService
     |
     v
IDonationRepository
     |
     v
Stored Procedure
```

Resolver-এর দায়িত্ব হলো request resolve করা। Business rules Application/Domain layer-এ থাকবে।

GraphQL schema-তে database entity সরাসরি expose না করে উপযুক্ত DTO এবং schema type ব্যবহার করা ভালো।

---

## 9. Security and Performance

### 9.1 Authentication and Authorization

শুধু endpoint-এ authentication চালু করলেই সব data নিরাপদ হয় না।

- JWT authentication সঠিকভাবে configure করতে হবে।
- Role এবং permission অনুযায়ী access নিয়ন্ত্রণ করতে হবে।
- প্রতিটি request-এ tenant/organization isolation enforce করতে হবে।
- GraphQL-এর প্রতিটি sensitive field ও operation-এ authorization যাচাই করতে হবে।
- gRPC method-এও authentication ও authorization প্রযোজ্য করতে হবে।

উদাহরণ:

```text
User A -> Organization A -> Donation A
User B -> Organization B -> Donation B
```

Organization B-এর user যেন Organization A-এর donation দেখতে না পারে—এটি service/repository level-এও enforce করতে হবে।

### 9.2 gRPC Performance

- Reusable channel/client ব্যবহার করো।
- প্রতিটি request-এর জন্য নতুন channel তৈরি কোরো না।
- Deadline এবং cancellation token ব্যবহার করো।
- Protobuf field number স্থিতিশীল রাখো।
- Retry policy এমনভাবে তৈরি করো যাতে duplicate payment-এর মতো সমস্যা না হয়।
- Large message এবং streaming workload পরীক্ষা করো।

### 9.3 GraphQL Performance

- N+1 query সমস্যা এড়াতে batching/DataLoader pattern ব্যবহার করো।
- Query depth ও complexity limit নির্ধারণ করো।
- Pagination বাধ্যতামূলক করো যেখানে list বড় হতে পারে।
- Expensive query-তে timeout এবং resource limit রাখো।
- Resolver-এর database access ও execution plan monitor করো।
- প্রয়োজন অনুযায়ী caching এবং persisted query ব্যবহার করো।

### 9.4 Observability

Production environment-এ তিনটি technology-র ক্ষেত্রেই দরকার:

- Structured logs।
- Distributed tracing।
- Request correlation/trace ID।
- Latency ও error-rate metrics।
- Health checks।
- Alerting এবং dashboards।

Metrics ও tracing দিয়ে actual bottleneck মাপবে; শুধু protocol-এর নাম দেখে performance সিদ্ধান্ত নেবে না।

---

## 10. Real-World Use Cases

### Use Case 1: Donation Platform

```text
Angular
   |
   v
REST / GraphQL API
   |
   +----> Donation Service
   |
   +----> Donor Service
   |
   +----> Reporting Service

Donation Service ---- gRPC ---- Payment Service
```

- REST: Donation create ও management endpoint।
- GraphQL: Dashboard-এ donation, donor এবং payment information fetch।
- gRPC: Internal service communication, যেখানে synchronous RPC উপযুক্ত।

**Payment processing-এর জন্য অতিরিক্ত সতর্কতা:** Gateway callback যাচাই, idempotency, transaction management এবং audit trail প্রয়োজন। শুধু gRPC ব্যবহার করলেই payment transaction নিরাপদ বা atomic হয় না।

### Use Case 2: E-commerce

- REST: Product CRUD এবং public integration।
- GraphQL: Product details, variants, reviews এবং availability এক query-তে আনা।
- gRPC: Internal inventory ও pricing service communication।

### Use Case 3: Live Monitoring

- gRPC server streaming: Server থেকে continuous updates।
- GraphQL subscription: Client-কে supported subscription transport-এর মাধ্যমে updates পাঠানো।
- REST: সাধারণ resource retrieval এবং management operations।

সব streaming mechanism-এর transport requirements একই নয়; browser এবং infrastructure support আগে যাচাই করতে হবে।

---

## 11. Best Practices

### gRPC Checklist

- [ ] `.proto` contract version control-এ রাখো।
- [ ] Breaking change এড়িয়ে backward compatibility বজায় রাখো।
- [ ] HTTP/2 ও deployment configuration যাচাই করো।
- [ ] Deadline, cancellation ও error handling যোগ করো।
- [ ] Secure transport এবং authorization ব্যবহার করো।
- [ ] Generated client ও server contract test করো।
- [ ] Retry এবং idempotency carefully design করো।

### GraphQL Checklist

- [ ] Schema-তে শুধু অনুমোদিত fields expose করো।
- [ ] Resolver-এ business logic জমা কোরো না।
- [ ] Query depth ও complexity limit করো।
- [ ] N+1 query সমস্যা পরীক্ষা করো।
- [ ] Pagination এবং input validation যোগ করো।
- [ ] Field-level এবং object-level authorization পরীক্ষা করো।
- [ ] Production-এ development-only IDE বা verbose errors সীমিত করো।

### Common Mistakes

| Mistake | Better approach |
|---|---|
| সব API-তে gRPC ব্যবহার | Client ও workload অনুযায়ী protocol বেছে নেওয়া |
| সব সমস্যার জন্য GraphQL | Query flexibility সত্যিই প্রয়োজন কি না যাচাই |
| Resolver-এ SQL লেখা | Application ও repository layer ব্যবহার |
| Unlimited GraphQL query | Depth/complexity limit ও pagination |
| সব gRPC call-এ unlimited retry | Deadline, retry budget ও idempotency |
| Schema পরিবর্তনে compatibility উপেক্ষা | Versioning এবং automated contract tests |

---

## 12. Learning Roadmap

নিচের ধাপগুলো অনুসরণ করে ধীরে ধীরে production-grade implementation শিখতে পারো।

### Phase 1 — Fundamentals

- [ ] REST API, HTTP ও JSON বুঝে নেওয়া।
- [ ] RPC concept শেখা।
- [ ] Protobuf এবং `.proto` file শেখা।
- [ ] GraphQL Query, Mutation ও Schema বোঝা।

### Phase 2 — Hands-on .NET

- [ ] .NET 8 gRPC Greeter service তৈরি।
- [ ] .NET gRPC client তৈরি।
- [ ] GraphQL query endpoint তৈরি।
- [ ] SQL Server repository-এর সঙ্গে GraphQL যুক্ত করা।
- [ ] REST, gRPC ও GraphQL endpoint test করা।

### Phase 3 — Professional Development

- [ ] Clean Architecture ও Dependency Injection।
- [ ] JWT, role/permission ও tenant isolation।
- [ ] Structured logging ও distributed tracing।
- [ ] Unit, integration এবং contract tests।
- [ ] Docker-based deployment।
- [ ] Performance testing এবং query optimization।

### Phase 4 — Portfolio Project

একটি ছোট Donation Platform তৈরি করো যেখানে:

1. REST API দিয়ে donation create করবে।
2. GraphQL দিয়ে dashboard data fetch করবে।
3. gRPC দিয়ে Donation Service থেকে Payment Service-এর সঙ্গে যোগাযোগ করবে।
4. SQL Server-এ donation data রাখবে।
5. JWT authorization এবং organization isolation enforce করবে।
6. Docker ও automated tests দিয়ে deployment প্রস্তুত করবে।

---

## Final Summary

**gRPC** মূলত efficient, strongly typed service-to-service communication-এর জন্য উপকারী।

**GraphQL** মূলত client-কে নিজের প্রয়োজন অনুযায়ী data query করার flexibility দেয়।

**REST API** এখনও public API, সাধারণ CRUD এবং Angular frontend-এর সঙ্গে যোগাযোগের জন্য একটি কার্যকর ও সহজে maintainable choice।

Professional developer হিসেবে লক্ষ্য হওয়া উচিত তিনটি technology-র পার্থক্য বোঝা এবং বাস্তব requirement অনুযায়ী সঠিকটি বেছে নেওয়া—অকারণে architecture জটিল করা নয়।

---




## REST বনাম gRPC বনাম GraphQL

| বিষয় | REST | gRPC | GraphQL |
|---|---|---|---|
| Protocol | HTTP/1.1 (বা 2) | HTTP/2 | HTTP (সাধারণত POST) |
| Format | JSON | Protobuf (binary) | JSON |
| Contract | OpenAPI (ঐচ্ছিক) | `.proto` (বাধ্যতামূলক) | Schema (বাধ্যতামূলক) |
| গতি | ভালো | **সবচেয়ে দ্রুত** | ভালো |
| Over/Under-fetching | আছে | — | **নেই** |
| Streaming | সীমিত | **চমৎকার** | Subscription |
| Browser সমর্থন | **চমৎকার** | সীমিত (gRPC-Web) | চমৎকার |
| Caching | **সহজ** (HTTP) | কঠিন | কঠিন |
| সেরা ব্যবহার | Public API, সাধারণ CRUD | Internal microservice | জটিল UI, mobile, অনেক client |

### বাস্তবে মিশ্র ব্যবহার ⭐
```
Mobile/Web ──(GraphQL বা REST)──▶ API Gateway ──(gRPC)──▶ ভেতরের Microservices
```

---

## ✅ সারসংক্ষেপ

- **gRPC** = দ্রুত, binary, service-to-service-এর জন্য আদর্শ
- **GraphQL** = client চায় যা, পায় তাই; জটিল frontend-এর জন্য ভালো
- **REST** = সরল, সর্বজনীন; এখনো বেশিরভাগ ক্ষেত্রে ডিফল্ট পছন্দ
- একটিই সব ক্ষেত্রে সেরা নয় — প্রয়োজন অনুযায়ী বাছুন

**Author:** Abdullah Hossain  
**Focus:** C# | ASP.NET Core | Angular | SQL Server | Clean Architecture | Microservices

⬅️ [Caching](11-caching.md) | ➡️ [CI/CD](13-ci-cd.md)
