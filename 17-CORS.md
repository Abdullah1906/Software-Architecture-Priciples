# 🌐 CORS (Cross-Origin Resource Sharing) 

>
> CORS কী, কেন প্রয়োজন, কীভাবে কাজ করে, Preflight Request, ASP.NET Core Web API Configuration, Angular 20 Integration, JWT Authentication, Production Best Practices এবং Troubleshooting – Basic to Advanced.

---

## 📌 Table of Contents

1. [CORS কী?](#1-cors-কী)
2. [CORS কেন প্রয়োজন?](#2-cors-কেন-প্রয়োজন)
3. [Origin কী?](#3-origin-কী)
4. [Same-Origin Policy কী?](#4-same-origin-policy-কী)
5. [CORS কীভাবে কাজ করে?](#5-cors-কীভাবে-কাজ-করে)
6. [Simple Request](#6-simple-request)
7. [Preflight Request](#7-preflight-request)
8. [Important CORS Headers](#8-important-cors-headers)
9. [CORS Configuration in ASP.NET Core](#9-cors-configuration-in-aspnet-core)
10. [Angular 20 + ASP.NET Core Integration](#10-angular-20--aspnet-core-integration)
11. [CORS with JWT Authentication](#11-cors-with-jwt-authentication)
12. [AllowAnyOrigin vs WithOrigins](#12-allowanyorigin-vs-withorigins)
13. [AllowCredentials কী?](#13-allowcredentials-কী)
14. [Production Grade CORS Configuration](#14-production-grade-cors-configuration)
15. [CORS Middleware Order](#15-cors-middleware-order)
16. [CORS vs HTTPS vs Authentication](#16-cors-vs-https-vs-authentication)
17. [CORS vs CSRF](#17-cors-vs-csrf)
18. [Common CORS Errors & Solutions](#18-common-cors-errors--solutions)
19. [Testing CORS](#19-testing-cors)
20. [Professional Best Practices](#20-professional-best-practices)
21. [Interview Questions & Answers](#21-interview-questions--answers)
22. [Final Checklist](#22-final-checklist)
23. [References](#23-references)

---

# 1. CORS কী?

**CORS = Cross-Origin Resource Sharing**

CORS হলো একটি Browser Security Mechanism, যার মাধ্যমে একটি Website অন্য Origin-এর Server/API থেকে Resource Access করতে পারবে কি না, তা নিয়ন্ত্রণ করা হয়।

সহজভাবে:

ধরো, তোমার দুটি Application আছে।

```text
Frontend:
Angular 20
http://localhost:4200

Backend:
ASP.NET Core Web API
https://localhost:7100
```

Angular থেকে ASP.NET Core API-তে Request পাঠানো হচ্ছে।

কিন্তু Browser যাচাই করবে Backend কি Angular-এর Origin-কে অনুমতি দিয়েছে?

যদি অনুমতি না দেয়, তাহলে Browser CORS Error দেখাতে পারে।

### CORS Architecture

```text
+---------------------------+
|       Angular 20          |
|   localhost:4200          |
+-------------+-------------+
              |
              |
              | HTTP Request
              |
              v
+---------------------------+
|    ASP.NET Core API       |
|    localhost:7100         |
+-------------+-------------+
              |
              | CORS Policy
              |
              v
+---------------------------+
|       SQL Server          |
|       MadrasaDb           |
+---------------------------+
```

**Important:** CORS মূলত Browser-এর Security Policy। এটি Server-to-Server Communication বা Postman Request-এ Browser-এর মতো প্রয়োগ হয় না।

---

# 2. CORS কেন প্রয়োজন?

ধরো, তোমার Angular Application থেকে API Call করছ।

```typescript
this.http.get(
  'https://localhost:7100/api/notices'
).subscribe({
  next: response => console.log(response),
  error: error => console.log(error)
});
```

Browser Request পাঠানোর সময় Origin Header পাঠাতে পারে।

```http
GET /api/notices HTTP/1.1
Host: localhost:7100

Origin: http://localhost:4200
```

Backend যদি অনুমতি না দেয়, Browser JavaScript-কে Response Access করতে দেবে না।

Example Error:

```text
Access to XMLHttpRequest at
'https://localhost:7100/api/notices'

from origin 'http://localhost:4200'

has been blocked by CORS policy.
```

অর্থাৎ, API Server Cross-Origin Access-এর প্রয়োজনীয় Permission দেয়নি।

### CORS-এর উদ্দেশ্য

- Cross-Origin Resource Access Control.
- Browser-based Security.
- Trusted Frontend Application-কে API Access দেওয়া।
- Unwanted Cross-Origin JavaScript Access সীমিত করা।

মনে রাখবে, CORS সম্পূর্ণ API Security Solution নয়।

---

# 3. Origin কী?

Origin = Protocol + Host + Port

Example:

```text
http://localhost:4200
```

এখানে:

| Component | Value |
|---|---|
| Protocol | HTTP |
| Host | localhost |
| Port | 4200 |

### Same Origin vs Different Origin

| URL | Origin |
|---|---|
| http://localhost:4200 | Original |
| http://localhost:5000 | Different |
| https://localhost:4200 | Different |
| http://127.0.0.1:4200 | Different |
| http://localhost:4200/products | Same |

**মনে রাখবে:** URL-এর Path পরিবর্তন হলে Origin পরিবর্তন হয় না। কিন্তু Protocol, Host অথবা Port পরিবর্তন হলে Origin পরিবর্তন হয়।

---

# 4. Same-Origin Policy কী?

Same-Origin Policy হলো Browser Security Mechanism, যা একটি Origin-এর Script-কে অন্য Origin-এর Resource Access করা থেকে সীমিত করে।

Example:

```text
Website A:
https://example.com

Website B:
https://another.com
```

Website A সরাসরি Website B-এর Response পড়তে পারবে না, যদি প্রয়োজনীয় Browser Security Permission না থাকে।

CORS হলো Server-এর মাধ্যমে নির্দিষ্ট Cross-Origin Access অনুমোদনের একটি পদ্ধতি।

```text
Same-Origin Policy
        |
        v
Cross-Origin Access Restricted
        |
        v
CORS Permission
        |
        v
Allowed Cross-Origin Access
```

---

# 5. CORS কীভাবে কাজ করে?

CORS প্রধানত দুই ধরনের Request-এর ক্ষেত্রে দেখা যায়।

1. Simple Request
2. Preflight Request

---

# 6. Simple Request

কিছু Cross-Origin Request-এর ক্ষেত্রে Browser সরাসরি মূল Request পাঠায়।

Example:

```http
GET /api/notices HTTP/1.1

Origin: http://localhost:4200
```

Backend Response:

```http
HTTP/1.1 200 OK

Access-Control-Allow-Origin: http://localhost:4200

Content-Type: application/json
```

এখানে Browser Response Header যাচাই করে।

Origin অনুমোদিত হলে JavaScript Response Access করতে পারবে।

### Simple Request-এর সাধারণ শর্ত

- GET, HEAD অথবা POST Method.
- কিছু নির্দিষ্ট Safe Request Header.
- POST-এর ক্ষেত্রে নির্দিষ্ট Safelisted Content-Type.

সব Request Simple Request নয়।

---

# 7. Preflight Request

**Preflight Request** হলো একটি OPTIONS Request, যা Browser কিছু Cross-Origin Request-এর আগে পাঠায়।

ধরো, Angular থেকে POST Request পাঠাচ্ছ।

```typescript
this.http.post(
  'https://localhost:7100/api/auth/login',
  {
    email: 'admin@gmail.com',
    password: '123456'
  }
);
```

Browser প্রথমে OPTIONS Request পাঠাতে পারে।

### Step 1: OPTIONS Request

```http
OPTIONS /api/auth/login HTTP/1.1

Origin: http://localhost:4200

Access-Control-Request-Method: POST

Access-Control-Request-Headers: content-type
```

### Step 2: Server Response

```http
HTTP/1.1 204 No Content

Access-Control-Allow-Origin: http://localhost:4200

Access-Control-Allow-Methods: GET, POST, PUT, DELETE

Access-Control-Allow-Headers: content-type
```

### Step 3: Actual Request

Preflight সফল হলে Browser মূল POST Request পাঠায়।

```http
POST /api/auth/login HTTP/1.1

Content-Type: application/json
```

### Preflight Flow Diagram

```text
+------------------+
|    Angular       |
+--------+---------+
         |
         | POST Request Required
         |
         v
+------------------+
|     Browser      |
+--------+---------+
         |
         | OPTIONS Request
         |
         v
+------------------+
| ASP.NET Core API |
+--------+---------+
         |
         | CORS Policy Check
         |
         v
+------------------+
| Permission Given?|
+--------+---------+
         |
      YES|       NO
         |        |
         v        v
     POST       Blocked
         |
         v
     Response
```

**Important:** সব Cross-Origin Request-এর জন্য Preflight হয় না।

---

# 8. Important CORS Headers

CORS-এর গুরুত্বপূর্ণ HTTP Headers:

| Header | কাজ |
|---|---|
| Origin | Request কোন Origin থেকে এসেছে |
| Access-Control-Allow-Origin | কোন Origin অনুমোদিত |
| Access-Control-Allow-Methods | কোন HTTP Method অনুমোদিত |
| Access-Control-Allow-Headers | কোন Request Header অনুমোদিত |
| Access-Control-Allow-Credentials | Credentials পাঠানোর অনুমতি |
| Access-Control-Expose-Headers | JavaScript কোন Response Header পড়তে পারবে |
| Access-Control-Max-Age | Preflight Result কতক্ষণ Cache থাকবে |
| Access-Control-Request-Method | Preflight-এ চাওয়া Method |
| Access-Control-Request-Headers | Preflight-এ চাওয়া Headers |

### Example

```http
Access-Control-Allow-Origin: https://mydonation.com

Access-Control-Allow-Methods: GET, POST, PUT, DELETE

Access-Control-Allow-Headers: Authorization, Content-Type
```

---

# 9. CORS Configuration in ASP.NET Core

এখন ASP.NET Core Web API-তে CORS Configure করব।

### Step 1: Create CORS Policy

`Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services.AddCors(options =>
{
    options.AddPolicy("AngularPolicy", policy =>
    {
        policy
            .WithOrigins("http://localhost:4200")
            .AllowAnyHeader()
            .AllowAnyMethod();
    });
});

var app = builder.Build();

app.UseHttpsRedirection();

app.UseRouting();

app.UseCors("AngularPolicy");

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();

app.Run();
```

### Step 2: Explanation

| Method | কাজ |
|---|---|
| AddCors() | CORS Service Register |
| AddPolicy() | CORS Policy তৈরি |
| WithOrigins() | কোন Origin Access পাবে |
| AllowAnyHeader() | যেকোনো Request Header Allow |
| AllowAnyMethod() | যেকোনো HTTP Method Allow |
| UseCors() | CORS Middleware চালু |

### Step 3: Test API

```text
Frontend:
http://localhost:4200

Backend:
https://localhost:7100
```

Angular থেকে API Call করো।

```typescript
this.http.get(
  'https://localhost:7100/api/notices'
).subscribe({
  next: data => console.log(data),
  error: error => console.error(error)
});
```

এখন Browser-এর Network Tab থেকে Response Header Check করো।

```http
Access-Control-Allow-Origin: http://localhost:4200
```

---

# 10. Angular 20 + ASP.NET Core Integration

ধরো তোমার Project:

```text
Frontend:
Angular 20

Backend:
ASP.NET Core 8 Web API

Database:
SQL Server

Architecture:
Clean Architecture
```

### Angular Environment Configuration

`environment.ts`

```typescript
export const environment = {
  production: false,
  apiUrl: 'https://localhost:7100/api'
};
```

`environment.prod.ts`

```typescript
export const environment = {
  production: true,
  apiUrl: 'https://api.mydonation.com/api'
};
```

### Angular API Service

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { environment } from '../../../environments/environment';

@Injectable({
  providedIn: 'root'
})
export class NoticeService {

  private apiUrl = environment.apiUrl;

  constructor(private http: HttpClient) {}

  getNotices() {
    return this.http.get(
      `${this.apiUrl}/notices`
    );
  }
}
```

### ASP.NET Core CORS Configuration

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AngularPolicy", policy =>
    {
        policy
            .WithOrigins("http://localhost:4200")
            .WithMethods("GET", "POST", "PUT", "DELETE")
            .WithHeaders("Content-Type", "Authorization");
    });
});
```

Middleware:

```csharp
app.UseRouting();

app.UseCors("AngularPolicy");

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

এখন Angular এবং ASP.NET Core-এর মধ্যে Cross-Origin Communication সম্ভব হবে।

---

# 11. CORS with JWT Authentication

ধরো তোমার Application JWT Authentication ব্যবহার করে।

Angular Interceptor:

```typescript
import {
  HttpInterceptorFn
} from '@angular/common/http';

export const authInterceptor: HttpInterceptorFn =
  (req, next) => {

    const token =
      localStorage.getItem('access_token');

    if (token) {
      req = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`
        }
      });
    }

    return next(req);
  };
```

এখন Angular API Request-এর সঙ্গে Authorization Header পাঠাবে।

```http
GET /api/admin/notices HTTP/1.1

Authorization: Bearer YOUR_JWT_TOKEN

Origin: http://localhost:4200
```

Backend CORS Configuration:

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("JwtPolicy", policy =>
    {
        policy
            .WithOrigins("http://localhost:4200")
            .WithMethods(
                "GET",
                "POST",
                "PUT",
                "DELETE"
            )
            .WithHeaders(
                "Authorization",
                "Content-Type"
            );
    });
});
```

**Important:**

- Authorization Header-এর কারণে Preflight হতে পারে।
- JWT Authentication-এর জন্য `AllowCredentials()` সবসময় প্রয়োজন হয় না।
- Token URL Query String-এ পাঠাবে না।
- HTTPS ব্যবহার করবে।

---

# 12. AllowAnyOrigin vs WithOrigins

## AllowAnyOrigin()

```csharp
policy
    .AllowAnyOrigin()
    .AllowAnyHeader()
    .AllowAnyMethod();
```

এর অর্থ যেকোনো Origin থেকে Browser-based Cross-Origin Access Allow করা।

## WithOrigins()

```csharp
policy
    .WithOrigins(
        "https://mydonation.com",
        "https://www.mydonation.com"
    )
    .AllowAnyHeader()
    .AllowAnyMethod();
```

এখানে শুধু নির্দিষ্ট Origin Allow হবে।

### Comparison

| বিষয় | AllowAnyOrigin | WithOrigins |
|---|---|---|
| All Origins | Yes | No |
| Specific Origins | No | Yes |
| Production | সাধারণত Avoid | Recommended |
| Security Control | কম | বেশি |

**Professional Recommendation:** Production-এ Specific Trusted Origin ব্যবহার করবে।

---

# 13. AllowCredentials কী?

Cross-Origin Request-এর সঙ্গে Browser Credentials পাঠানোর ক্ষেত্রে এটি ব্যবহৃত হয়।

Credentials-এর উদাহরণ:

- Cookies
- HTTP Authentication Credentials
- TLS Client Certificates

ASP.NET Core:

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("CookiePolicy", policy =>
    {
        policy
            .WithOrigins("https://mydonation.com")
            .AllowAnyHeader()
            .AllowAnyMethod()
            .AllowCredentials();
    });
});
```

Angular:

```typescript
this.http.get(
  'https://api.mydonation.com/api/profile',
  {
    withCredentials: true
  }
);
```

### Important Rules

- `AllowCredentials()` ব্যবহার করলে `AllowAnyOrigin()` ব্যবহার করা যাবে না।
- Specific Origin দিতে হবে।
- Cross-Site Cookie-এর জন্য প্রয়োজন অনুযায়ী `SameSite=None; Secure` Configure করতে হবে।
- CSRF Protection আলাদাভাবে Implement করতে হবে।

---

# 14. Production Grade CORS Configuration

Production-এ Development এবং Production Origin আলাদা রাখবে।

### Step 1: appsettings.json

```json
{
  "Cors": {
    "AllowedOrigins": [
      "http://localhost:4200",
      "https://mydonation.com",
      "https://www.mydonation.com"
    ]
  }
}
```

### Step 2: Program.cs

```csharp
var allowedOrigins =
    builder.Configuration
        .GetSection("Cors:AllowedOrigins")
        .Get<string[]>() ?? [];

builder.Services.AddCors(options =>
{
    options.AddPolicy("ProductionPolicy", policy =>
    {
        policy
            .WithOrigins(allowedOrigins)
            .WithMethods(
                "GET",
                "POST",
                "PUT",
                "DELETE",
                "PATCH"
            )
            .WithHeaders(
                "Authorization",
                "Content-Type"
            );
    });
});
```

### Step 3: Middleware

```csharp
app.UseRouting();

app.UseCors("ProductionPolicy");

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

### Production Domain Example

```text
Frontend:
https://mydonation.com

Backend:
https://api.mydonation.com
```

এখন শুধু Configured Origin-গুলো Browser-based Cross-Origin Response Access করতে পারবে।

**Note:** Production Configuration-এ localhost Origin বাদ রাখবে। Environment-specific Configuration ব্যবহার করা ভালো।

---

# 15. CORS Middleware Order

ASP.NET Core-এ Middleware Order গুরুত্বপূর্ণ।

Recommended Order:

```csharp
app.UseRouting();

app.UseCors("AngularPolicy");

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

Architecture:

```text
Incoming Request
       |
       v
UseRouting()
       |
       v
UseCors()
       |
       v
UseAuthentication()
       |
       v
UseAuthorization()
       |
       v
MapControllers()
       |
       v
Response
```

ভুল Middleware Order-এর কারণে Preflight Request বা CORS Response Header নিয়ে সমস্যা হতে পারে।

---

# 16. CORS vs HTTPS vs Authentication

এই তিনটি বিষয় এক নয়।

| Technology | কাজ |
|---|---|
| CORS | Browser-এর Cross-Origin Response Access Control |
| HTTPS | Network Communication Encrypt করে |
| Authentication | User কে তা যাচাই করে |
| Authorization | User কী করতে পারবে তা নির্ধারণ করে |

Example:

```text
+----------------------+
|     Angular 20       |
+----------+-----------+
           |
           | HTTPS
           |
           v
+----------------------+
|  ASP.NET Core API    |
+----------------------+
           |
           v
      CORS Policy
           |
           v
   JWT Authentication
           |
           v
   Role Authorization
           |
           v
      SQL Server
```

CORS থাকলেই API Secure হয় না।

API-তে Authentication এবং Authorization আলাদাভাবে Implement করতে হবে।

---

# 17. CORS vs CSRF

| বিষয় | CORS | CSRF |
|---|---|---|
| Full Form | Cross-Origin Resource Sharing | Cross-Site Request Forgery |
| Type | Browser Security Mechanism | Security Attack |
| Purpose | Cross-Origin Response Access Control | Unauthorized Request প্রতিরোধ |
| Protection | CORS Policy | CSRF Token, SameSite Cookie |
| Authentication | Authentication নয় | Authenticated User-কে Target করে |

**Important:** CORS নিজে CSRF Protection নয়। কিছু Cross-Origin Request Response Access Block হলেও Request Server-এ পৌঁছাতে পারে। তাই Cookie-based Authentication ব্যবহার করলে CSRF Protection প্রয়োজন।

---

# 18. Common CORS Errors & Solutions

| Error | Cause | Solution |
|---|---|---|
| No Access-Control-Allow-Origin | Origin Allow করা নেই | WithOrigins() |
| Method not allowed | HTTP Method Allow করা নেই | WithMethods() |
| Header not allowed | Request Header Allow করা নেই | WithHeaders() |
| Preflight Failed | OPTIONS Request সফল হয়নি | Middleware Order Check |
| Redirect on Preflight | OPTIONS Request Redirect হচ্ছে | HTTPS/Proxy/CORS Configuration Check |
| Credentials not supported | Wildcard Origin-এর সঙ্গে Credentials | Specific Origin ব্যবহার |
| Multiple Allow-Origin Headers | একাধিক জায়গায় CORS Header যোগ হচ্ছে | Duplicate Configuration Remove |

### Special Case: 307 Redirect Problem

ধরো:

```text
Angular:
http://localhost:4200

API:
http://localhost:5100
```

কিন্তু API HTTP থেকে HTTPS Redirect করছে।

```text
http://localhost:5100
          |
          v
https://localhost:7100
```

OPTIONS Preflight Request Redirect হলে Browser Request Block করতে পারে।

### Solution

1. Angular থেকে সরাসরি HTTPS API URL ব্যবহার করবে।
2. CORS Middleware সঠিক জায়গায় রাখবে।
3. Reverse Proxy-এর Forwarded Headers Configure করবে।
4. Preflight Request যেন Authentication Challenge বা Redirect-এর কারণে ব্যর্থ না হয় তা নিশ্চিত করবে।

---

# 19. Testing CORS

### Method 1: Browser Developer Tools

1. Browser Open করবে।
2. F12 Press করবে।
3. Network Tab Open করবে।
4. API Request পাঠাবে।
5. OPTIONS Request আছে কি না দেখবে।
6. Response Headers Check করবে।

Expected Header:

```http
Access-Control-Allow-Origin: http://localhost:4200
```

### Method 2: cURL

Simple Request:

```bash
curl -i \
  -H "Origin: http://localhost:4200" \
  https://localhost:7100/api/notices
```

Preflight Request:

```bash
curl -i -X OPTIONS \
  https://localhost:7100/api/notices \
  -H "Origin: http://localhost:4200" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: authorization,content-type"
```

> cURL শুধু Server-এর Response Header দেখায়। এটি Browser-এর CORS Enforcement নিজে থেকে প্রয়োগ করে না।

### Method 3: Postman

Postman দিয়ে API Request Test করা যায়।

কিন্তু Postman Browser-এর CORS Policy Enforce করে না। তাই Postman-এ সফল Request মানেই Browser-এ CORS ঠিক আছে এমন নয়।

---

# 20. Professional Best Practices

Production Application-এ নিচের Best Practices Follow করবে।

### CORS Configuration

- Specific Origin Allow করবে।
- `AllowAnyOrigin()` Avoid করবে।
- Required HTTP Methods-ই Allow করবে।
- Required Headers-ই Allow করবে।
- Development এবং Production Configuration আলাদা রাখবে।
- Duplicate CORS Middleware Avoid করবে।

### Authentication & Security

- HTTPS ব্যবহার করবে।
- JWT Token Securely Handle করবে।
- Authentication ও Authorization আলাদা রাখবে।
- Cookie Authentication-এ CSRF Protection ব্যবহার করবে।
- Secure Cookie Attributes Configure করবে।
- API Rate Limiting Implement করবে।

### Deployment

- Reverse Proxy Configuration Check করবে।
- Forwarded Headers Trusted Proxy-এর জন্য Configure করবে।
- OPTIONS Preflight Test করবে।
- Production Browser Testing করবে।
- Error Logs Monitor করবে।

---

# 21. Interview Questions & Answers

### Q1. What is CORS?

**Answer:** CORS হলো Browser Security Mechanism, যা Cross-Origin HTTP Request-এর Response Access Control করে।

### Q2. What is an Origin?

**Answer:** Origin হলো Protocol, Host এবং Port-এর Combination।

### Q3. What is a Preflight Request?

**Answer:** কিছু Cross-Origin Request-এর আগে Browser OPTIONS Method ব্যবহার করে Server-এর Permission যাচাই করে। এটিই Preflight Request।

### Q4. What is Access-Control-Allow-Origin?

**Answer:** এটি একটি HTTP Response Header, যা কোন Origin Response Access করতে পারবে তা নির্ধারণ করে।

### Q5. Does Postman follow CORS?

**Answer:** না। Postman Browser-এর CORS Enforcement-এর অধীন নয়।

### Q6. Can CORS prevent SQL Injection?

**Answer:** না। SQL Injection প্রতিরোধ করতে Parameterized Queries, Input Validation এবং নিরাপদ Database Access প্রয়োজন।

### Q7. What is the difference between CORS and CSRF?

**Answer:** CORS Browser-এর Cross-Origin Response Access নিয়ন্ত্রণ করে। CSRF হলো Authenticated User-এর Browser ব্যবহার করে অননুমোদিত Request করানোর Attack।

### Q8. Why should we avoid AllowAnyOrigin in production?

**Answer:** এটি যেকোনো Website-কে Browser-এর মাধ্যমে API Response Access করার অনুমতি দিতে পারে। তাই Specific Trusted Origin ব্যবহার করা ভালো।

### Q9. Is CORS enforced by the server or browser?

**Answer:** Server CORS Headers দিয়ে Permission জানায়, কিন্তু Browser সেই Policy Enforce করে।

### Q10. Does CORS protect APIs from Postman or cURL?

**Answer:** না। CORS Browser-based Security Mechanism। Postman এবং cURL সরাসরি API Call করতে পারে।

### Q11. Why does Angular generate OPTIONS requests?

**Answer:** কিছু Cross-Origin Request-এর ক্ষেত্রে Browser মূল Request পাঠানোর আগে Preflight OPTIONS Request পাঠিয়ে Server-এর Permission যাচাই করে।

### Q12. Does JWT require AllowCredentials?

**Answer:** সাধারণ Authorization Bearer Header-এর জন্য `AllowCredentials()` প্রয়োজন হয় না। এটি মূলত Browser-managed Credentials যেমন Cookies ব্যবহারের ক্ষেত্রে গুরুত্বপূর্ণ।

### Q13. What happens if CORS Middleware is placed incorrectly?

**Answer:** CORS Headers সঠিকভাবে Apply না হওয়া, Preflight Failure বা Authentication-related সমস্যার সৃষ্টি হতে পারে।

### Q14. Is CORS a replacement for Authentication?

**Answer:** না। CORS এবং Authentication সম্পূর্ণ আলাদা Security Mechanism।

---

# 22. Final Checklist – Professional Developer

### Development

- [ ] CORS Service Register করা।
- [ ] Specific Origin Configure করা।
- [ ] Required Methods Allow করা।
- [ ] Required Headers Allow করা।
- [ ] Angular API URL Configure করা।
- [ ] OPTIONS Preflight Test করা।

### Production

- [ ] Production Origin Configure করা।
- [ ] Development Origin বাদ দেওয়া।
- [ ] Wildcard Origin Avoid করা।
- [ ] Credentials Configuration Verify করা।
- [ ] HTTPS Enabled রাখা।
- [ ] JWT Authentication Implement করা।
- [ ] Role-Based Authorization Implement করা।
- [ ] CSRF Protection (যেখানে প্রয়োজন)।
- [ ] Reverse Proxy Configuration Verify করা।
- [ ] Browser Network Tab দিয়ে Test করা।

---

# 23. References & Official Documentation

1. MDN – CORS Guide  
   https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS

2. Microsoft – CORS in ASP.NET Core  
   https://learn.microsoft.com/en-us/aspnet/core/security/cors

3. MDN – Same-Origin Policy  
   https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy

4. OWASP – CSRF Prevention Cheat Sheet  
   https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html

5. MDN – HTTP Headers  
   https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers

---

## 🎯 Final Learning Summary

একজন Professional Full Stack .NET Developer হিসেবে তোমার বুঝতে হবে:

- CORS কেন Browser-এ প্রয়োজন।
- Same-Origin Policy কীভাবে কাজ করে।
- Simple Request ও Preflight Request-এর পার্থক্য।
- OPTIONS Request কীভাবে Handle হয়।
- ASP.NET Core-এ CORS Policy কীভাবে Configure করতে হয়।
- Angular 20 থেকে Secure API Request কীভাবে পাঠাতে হয়।
- JWT এবং Cookie Authentication-এর সঙ্গে CORS-এর পার্থক্য।
- Production Environment-এ CORS কীভাবে Secure করতে হয়।

**Author:** Full Stack .NET Developer  
**Technologies:** C# | ASP.NET Core 8 | Angular 20 | Web API | JWT | SQL Server  
**Category:** Web Security  
**Level:** Basic to Advanced  
**Last Updated:** October 2026
