# Chapter ১২: gRPC ও GraphQL

REST-এর বিকল্প বা পরিপূরক দুটি জনপ্রিয় API পদ্ধতি। দুটির সমস্যা ও লক্ষ্য আলাদা।

---

## ভাগ ১: gRPC

### ১. gRPC কী?

Google-এর তৈরি **high-performance RPC (Remote Procedure Call)** framework। অন্য server-এর method-কে এমনভাবে call করা যায় যেন সেটি local method।

- **Protocol:** HTTP/2
- **Data format:** **Protocol Buffers (Protobuf)** — binary, ছোট ও দ্রুত
- **Contract-first:** `.proto` ফাইলে API সংজ্ঞায়িত, সেখান থেকে client/server কোড তৈরি হয়

### ২. কেন ব্যবহার করি?

| সুবিধা | ব্যাখ্যা |
|---|---|
| **দ্রুত** | Binary + HTTP/2 → JSON/REST-এর চেয়ে ছোট ও দ্রুত |
| **Strongly typed** | Contract থেকে কোড তৈরি, ভুল কম |
| **Streaming** | Client/Server/দ্বিমুখী stream |
| **Multi-language** | C#, Java, Go, Python... সব একই `.proto` থেকে |
| **Multiplexing** | HTTP/2-তে এক connection-এ অনেক call |

### ৩. `.proto` ফাইল

```protobuf
syntax = "proto3";

option csharp_namespace = "MyApp.Grpc";

service OrderService {
  rpc GetOrder (GetOrderRequest) returns (OrderReply);                 // Unary
  rpc StreamOrders (StreamRequest) returns (stream OrderReply);        // Server streaming
}

message GetOrderRequest { int32 id = 1; }

message OrderReply {
  int32  id = 1;
  string customer_name = 2;
  double total = 3;
}
```

### ৪. ASP.NET Core Server

```csharp
public class OrderGrpcService : OrderService.OrderServiceBase
{
    private readonly IOrderRepository _repo;
    public OrderGrpcService(IOrderRepository repo) => _repo = repo;

    public override async Task<OrderReply> GetOrder(GetOrderRequest request, ServerCallContext context)
    {
        var order = await _repo.GetByIdAsync(request.Id)
            ?? throw new RpcException(new Status(StatusCode.NotFound, "Order নেই"));

        return new OrderReply { Id = order.Id, CustomerName = order.CustomerName, Total = (double)order.Total };
    }
}

// Program.cs
builder.Services.AddGrpc();
app.MapGrpcService<OrderGrpcService>();
```

### Client
```csharp
using var channel = GrpcChannel.ForAddress("https://localhost:5001");
var client = new OrderService.OrderServiceClient(channel);
var reply = await client.GetOrderAsync(new GetOrderRequest { Id = 10 });
```

### ৫. gRPC-র ৪ ধরনের call

| ধরন | বর্ণনা | উদাহরণ |
|---|---|---|
| Unary | ১টি request → ১টি response | Order আনা |
| Server streaming | ১টি request → অনেক response | Live price feed |
| Client streaming | অনেক request → ১টি response | বড় ফাইল upload |
| Bidirectional | দুই দিকেই stream | Chat |

### ৬. কোথায় সেরা?
- **Microservices-এর ভেতরের** service-to-service যোগাযোগ ⭐
- কম latency/বেশি throughput দরকার
- Real-time streaming

### ৭. সীমাবদ্ধতা
- Browser থেকে সরাসরি কঠিন (**gRPC-Web** লাগে)
- Human-readable নয় (debug কঠিন; Postman এখন সমর্থন করে)
- Public API হিসেবে REST-এর মতো সার্বজনীন নয়

---

## ভাগ ২: GraphQL

### ১. GraphQL কী?

Facebook-এর তৈরি **query language ও runtime**। **Client নিজেই বলে দেয় কোন data চাই** — একটি endpoint, নির্দিষ্ট আকারের উত্তর।

### ২. কোন সমস্যার সমাধান?

REST-এ সাধারণ দুটি সমস্যা:

| সমস্যা | মানে |
|---|---|
| **Over-fetching** | দরকার নাম, কিন্তু পুরো user object আসে |
| **Under-fetching** | একটি screen-এর জন্য ৩–৪টি আলাদা API call (`/users`, `/orders`, `/items`) |

### ৩. উদাহরণ

**একটি request, ঠিক যা দরকার:**
```graphql
query {
  order(id: 10) {
    id
    total
    customer { name phone }
    items { testName price }
  }
}
```
**Response (আকার query-র মতোই):**
```json
{
  "data": {
    "order": {
      "id": 10,
      "total": 1500,
      "customer": { "name": "রহিম", "phone": "017..." },
      "items": [{ "testName": "CBC", "price": 500 }]
    }
  }
}
```

### ৪. মূল ধারণা

| ধারণা | কাজ |
|---|---|
| **Query** | data পড়া |
| **Mutation** | data বদলানো (create/update/delete) |
| **Subscription** | Real-time আপডেট (WebSocket) |
| **Schema** | Type-এর সংজ্ঞা (strongly typed) |
| **Resolver** | প্রতিটি field-এর data কোথা থেকে আসবে |

```graphql
type Order { id: Int! total: Float! customer: Customer! items: [OrderItem!]! }
type Query { order(id: Int!): Order  orders: [Order!]! }
type Mutation { placeOrder(input: PlaceOrderInput!): Order! }
```

### ৫. ASP.NET Core (Hot Chocolate)

```csharp
public class Query
{
    public Task<Order?> GetOrder([Service] IOrderRepository repo, int id) => repo.GetByIdAsync(id);

    [UsePaging] [UseFiltering] [UseSorting]
    public IQueryable<Order> GetOrders([Service] AppDbContext db) => db.Orders;
}

builder.Services.AddGraphQLServer().AddQueryType<Query>().AddFiltering().AddSorting();
app.MapGraphQL();
```

### ৬. সুবিধা
- একটি request-এ সব দরকারি data
- Mobile-এ নেটওয়ার্ক ব্যবহার কম
- Frontend স্বাধীনভাবে বদলাতে পারে (backend বদলাতে হয় না)
- Self-documenting schema

### ৭. চ্যালেঞ্জ ও ঝুঁকি ⚠️

| সমস্যা | সমাধান |
|---|---|
| **N+1 query** (resolver-এ বারবার DB call) | **DataLoader** / batching |
| অতিরিক্ত গভীর/ভারী query (DoS ঝুঁকি) | Query depth/complexity limit |
| HTTP caching কঠিন (সব POST `/graphql`) | Persisted queries, ক্লায়েন্ট-সাইড cache |
| Authorization জটিল (field-level) | প্রতিটি resolver-এ নিয়ন্ত্রণ |
| শেখার বক্ররেখা | — |

---

## ভাগ ৩: REST বনাম gRPC বনাম GraphQL

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

⬅️ [Caching](11-caching.md) | ➡️ [CI/CD](13-ci-cd.md)
