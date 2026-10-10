# Chapter ০১: REST API

## ১. REST API কী?

**REST** = **RE**presentational **S**tate **T**ransfer।

এটি একটি **architectural style** (নিয়মের সমষ্টি), যার মাধ্যমে client (browser, mobile app, Angular) ও server HTTP দিয়ে কথা বলে। Server **resource** (যেমন: User, Order, Product) expose করে, আর client HTTP method দিয়ে সেগুলো ব্যবহার করে।

> উদাহরণ: রেস্টুরেন্টে আপনি (client) ওয়েটারকে (API) বলেন "একটা বিরিয়ানি দাও"। রান্নাঘরে কী হচ্ছে আপনার জানার দরকার নেই — শুধু menu (API contract) জানলেই চলে।

---

## ২. কেন ব্যবহার করি?

| সুবিধা | ব্যাখ্যা |
|---|---|
| Frontend-Backend আলাদা | Angular/React/Mobile একই API ব্যবহার করতে পারে |
| Standard | HTTP সবাই বোঝে, নতুন শেখার খরচ কম |
| Stateless | Server আগের request মনে রাখে না → সহজে scale করা যায় |
| Cacheable | GET response cache করে speed বাড়ানো যায় |
| Language-independent | C#, Java, JS — যে কোনো ভাষায় client বানানো যায় |

---

## ৩. REST-এর ৬টি মূল নীতি

1. **Client-Server** — frontend ও backend আলাদা
2. **Stateless** — প্রতিটি request-এ প্রয়োজনীয় সব তথ্য (যেমন JWT token) থাকবে
3. **Cacheable** — response cache করা যাবে কিনা বলে দেওয়া
4. **Uniform Interface** — সব resource-এ একই ধরনের URL/method নিয়ম
5. **Layered System** — client জানে না মাঝে load balancer/gateway আছে কিনা
6. **Code on Demand** *(optional)*

---

## ৪. কীভাবে কাজ করে?

```
Client ──(HTTP Request)──▶ Server ──▶ Database
Client ◀─(HTTP Response)── Server ◀──
```

একটি request-এর অংশ:

```http
POST /api/orders HTTP/1.1
Host: api.qisslab.com
Authorization: Bearer <jwt>
Content-Type: application/json

{ "customerId": 12, "items": [{ "productId": 5, "qty": 2 }] }
```

### HTTP Method ও CRUD

| Method | কাজ | উদাহরণ | Idempotent? |
|---|---|---|---|
| `GET` | তথ্য পড়া | `GET /api/orders/10` | ✅ |
| `POST` | নতুন তৈরি | `POST /api/orders` | ❌ |
| `PUT` | পুরো replace | `PUT /api/orders/10` | ✅ |
| `PATCH` | আংশিক update | `PATCH /api/orders/10` | ❌ (সাধারণত) |
| `DELETE` | মুছে ফেলা | `DELETE /api/orders/10` | ✅ |

> **Idempotent** = একই request ১০ বার পাঠালেও ফলাফল ১ বারের মতোই।

### Status Code

| Code | অর্থ | কখন |
|---|---|---|
| 200 OK | সফল | GET/PUT সফল |
| 201 Created | তৈরি হয়েছে | POST সফল |
| 204 No Content | সফল, body নেই | DELETE সফল |
| 400 Bad Request | ভুল input | Validation fail |
| 401 Unauthorized | login নেই | Token নেই/ভুল |
| 403 Forbidden | permission নেই | Role মেলেনি |
| 404 Not Found | পাওয়া যায়নি | ভুল id |
| 409 Conflict | সংঘাত | Duplicate data |
| 422 Unprocessable | business rule ভঙ্গ | Stock নেই |
| 500 Server Error | server-এর ভুল | Unhandled exception |

---

## ৫. URL Design — Professional নিয়ম

✅ **ভালো:**
```
GET    /api/v1/orders            → সব order
GET    /api/v1/orders/10         → একটি order
POST   /api/v1/orders            → নতুন order
PUT    /api/v1/orders/10         → update
DELETE /api/v1/orders/10         → delete
GET    /api/v1/customers/5/orders → customer ৫-এর order
```

❌ **খারাপ:**
```
GET  /api/getAllOrders
POST /api/createOrder
POST /api/deleteOrder?id=10
```

**নিয়ম:**
- URL-এ **noun** (বিশেষ্য), verb নয় — কাজ বোঝায় HTTP method
- **Plural** ব্যবহার করুন (`orders`, `users`)
- **lowercase + hyphen** (`order-items`)
- Version রাখুন (`/v1/`)

---

## ৬. ASP.NET Core উদাহরণ

```csharp
[ApiController]
[Route("api/v1/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _service;
    public OrdersController(IOrderService service) => _service = service;

    [HttpGet("{id:int}")]
    public async Task<ActionResult<OrderDto>> GetById(int id)
    {
        var order = await _service.GetByIdAsync(id);
        return order is null ? NotFound() : Ok(order);
    }

    [HttpPost]
    public async Task<ActionResult<OrderDto>> Create(CreateOrderRequest request)
    {
        var created = await _service.CreateAsync(request);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }

    [HttpDelete("{id:int}")]
    public async Task<IActionResult> Delete(int id)
    {
        await _service.DeleteAsync(id);
        return NoContent();
    }
}
```

---

## ৭. কোন পরিস্থিতিতে ব্যবহার হয়?

- Web app (Angular/React) ↔ Backend
- Mobile app ↔ Backend
- এক service থেকে আরেক service-এ call (Microservices)
- তৃতীয় পক্ষকে API দেওয়া (Payment gateway, SMS, Public API)

**কখন REST সেরা নয়?**
- Real-time chat/live data → **SignalR / WebSocket**
- অনেক nested data এক call-এ → **GraphQL**
- Service-to-service high performance → **gRPC**

---

## ৮. Professional-grade Practices

### ক) Pagination, Filtering, Sorting
```
GET /api/v1/orders?page=2&pageSize=20&status=pending&sort=-createdAt
```
সব data একসাথে কখনো return করবেন না।

### খ) DTO ব্যবহার করুন — Entity সরাসরি নয়
```csharp
public record OrderDto(int Id, string CustomerName, decimal Total, string Status);
```
Entity expose করলে internal field (password hash ইত্যাদি) leak হতে পারে।

### গ) Consistent Error Format (RFC 7807 — ProblemDetails)
```json
{
  "type": "https://httpstatuses.com/422",
  "title": "Insufficient stock",
  "status": 422,
  "detail": "Product 5 has only 1 unit left.",
  "traceId": "00-abc123"
}
```

### ঘ) Global Exception Handling
Controller-এ try/catch ছড়ানোর বদলে একটি middleware রাখুন।

### ঙ) Authentication ও Authorization
- JWT Bearer token
- `[Authorize(Roles = "Admin")]`

### চ) Versioning
`/api/v1`, `/api/v2` — পুরনো client যেন ভেঙে না যায়।

### ছ) অন্যান্য
- **HTTPS** বাধ্যতামূলক
- **Rate limiting** — abuse ঠেকাতে
- **Swagger/OpenAPI** — documentation
- **CORS** সঠিকভাবে configure
- **Logging + Correlation ID** — request trace করতে
- **Idempotency-Key** header — payment-এর মতো POST-এ double submit ঠেকাতে

---

## ৯. সাধারণ ভুল

| ভুল | সমাধান |
|---|---|
| সব কিছুতে `POST` | সঠিক method ব্যবহার করুন |
| সব error-এ `200 OK` + `{success:false}` | সঠিক status code দিন |
| URL-এ verb (`/getUser`) | noun ব্যবহার করুন |
| Pagination নেই | সবসময় page/pageSize রাখুন |
| Entity সরাসরি return | DTO ব্যবহার করুন |
| Version নেই | `/v1` দিয়ে শুরু করুন |

---

## ✅ সারসংক্ষেপ

- REST = HTTP + Resource + Stateless
- URL-এ noun, কাজ বোঝায় HTTP method
- সঠিক status code, DTO, pagination, versioning, ProblemDetails — এগুলোই professional API-র পরিচয়

➡️ পরের chapter: [ACID](02-acid.md)
