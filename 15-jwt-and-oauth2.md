# Chapter ১৫: JWT ও OAuth 2.0

## শুরুতেই পার্থক্য বুঝুন

| | কী | কাজ |
|---|---|---|
| **Authentication (AuthN)** | তুমি কে? | পরিচয় যাচাই (login) |
| **Authorization (AuthZ)** | তুমি কী করতে পারো? | অনুমতি যাচাই (role/permission) |
| **JWT** | একটি **token-এর ফরম্যাট** | তথ্য বহন ও স্বাক্ষরিত |
| **OAuth 2.0** | একটি **Authorization framework/প্রটোকল** | অন্যকে সীমিত অনুমতি দেওয়া |
| **OpenID Connect (OIDC)** | OAuth 2.0-র উপর **Authentication** স্তর | "login with Google" |

> ⚠️ **JWT ≠ OAuth**। OAuth 2.0 token হিসেবে JWT ব্যবহার করতে *পারে*, কিন্তু দুটি আলাদা জিনিস।

---


## 📚 Table of Contents

1. [Introduction](#1-introduction)
2. [Authentication vs Authorization](#2-authentication-vs-authorization)
3. [What is JWT?](#3-what-is-jwt)
4. [JWT Structure](#4-jwt-structure)
5. [How JWT Works](#5-how-jwt-works)
6. [Access Token vs Refresh Token](#6-access-token-vs-refresh-token)
7. [What is OAuth 2.0?](#7-what-is-oauth-20)
8. [OAuth 2.0 Flow](#8-oauth-20-flow)
9. [OAuth 2.0 Grant Types](#9-oauth-20-grant-types)
10. [JWT vs OAuth 2.0](#10-jwt-vs-oauth-20)
11. [What is OpenID Connect (OIDC)?](#11-what-is-openid-connect-oidc)
12. [Professional Architecture](#12-professional-architecture)
13. [ASP.NET Core 8 JWT Implementation](#13-aspnet-core-8-jwt-implementation)
14. [Protect API Endpoints](#14-protect-api-endpoints)
15. [Angular 20 JWT Integration](#15-angular-20-jwt-integration)
16. [Refresh Token Implementation](#16-refresh-token-implementation)
17. [Professional Security Best Practices](#17-professional-security-best-practices)
18. [Common JWT Errors](#18-common-jwt-errors)
19. [Real-World Example](#19-real-world-example)
20. [Interview Questions](#20-interview-questions)
21. [Conclusion](#21-conclusion)
22. [References](#22-references)

---

# 1. Introduction

বর্তমান সময়ে প্রায় প্রতিটি Web Application-এ Authentication এবং Authorization প্রয়োজন হয়।

উদাহরণ:

- Banking Application
- E-commerce Application
- Hospital Management System
- School Management System
- Madrasa Management System
- Enterprise Software
- Microservices Architecture

ধরো, আমাদের একটি Application আছে।

Frontend: Angular 20

Backend: ASP.NET Core 8 Web API

Database: SQL Server

এখন আমাদের প্রয়োজন:

- User Login করতে পারবে।
- Login করার পর API Access করতে পারবে।
- Admin শুধু Admin-এর কাজ করতে পারবে।
- Employee শুধু তার Permission অনুযায়ী কাজ করতে পারবে।
- Token Expire হলে নতুন Token পাওয়া যাবে।
- Logout করলে Refresh Token বাতিল হবে।

এই কাজগুলো বাস্তবায়নের জন্য JWT এবং OAuth 2.0 গুরুত্বপূর্ণ।

# 2. Authentication vs Authorization

দুটো বিষয় দেখতে একই রকম হলেও তাদের কাজ আলাদা।

## Authentication কী?

Authentication হলো User কে Verify করা।

অর্থাৎ, User আসলেই কে?

উদাহরণ:

```text
Email: admin@gmail.com
Password: ********

System:
এই User কি সত্যিই Admin?
```

Authentication সফল হলে User Identity প্রতিষ্ঠিত হয়।

## Authorization কী?

Authorization হলো Authenticated User কোন কোন Resource Access করতে পারবে তা নির্ধারণ করা।

উদাহরণ:

| User | Permission |
|---|---|
| SuperAdmin | সবকিছু |
| Admin | User Management |
| Accountant | Payment Management |
| Employee | Assigned Tasks |
| Guest | Public Pages |

### সহজ পার্থক্য

```text
Authentication = Who are you?

Authorization = What can you access?
```

# 3. What is JWT?

**JWT = JSON Web Token**

JWT হলো একটি Compact এবং URL-safe Token Format, যার মাধ্যমে দুইটি System-এর মধ্যে Claims বা তথ্য নিরাপদভাবে আদান-প্রদান করা যায়।

JWT সাধারণত Authentication এবং API Authorization-এর কাজে ব্যবহৃত হয়।

উদাহরণ:

একজন User Login করল।

Server যাচাই করল:

```text
Email: admin@gmail.com
Password: Correct
```

এরপর Server একটি JWT তৈরি করে Client-কে পাঠাল।

```text
Login Successful!

Access Token:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

এরপর Angular থেকে API Request পাঠানোর সময় Token ব্যবহার করা হবে।

```http
GET /api/admin/users

Authorization: Bearer YOUR_ACCESS_TOKEN
```

API Token Validate করে Request অনুমোদন করবে।

## JWT কেন ব্যবহার করব?

1. Stateless Authentication Support করে।
2. API Authentication সহজ করে।
3. Microservices-এর মধ্যে Token ব্যবহার করা যায়।
4. Role-based Authorization করা যায়।
5. Permission-based Authorization করা যায়।
6. Mobile এবং Web Application উভয় জায়গায় ব্যবহার করা যায়।
7. HTTP Header-এর মাধ্যমে সহজে পাঠানো যায়।

**Important:** JWT নিজে কোনো Authentication Protocol নয়। এটি একটি Token Format।

# 4. JWT Structure

একটি JWT সাধারণত তিনটি অংশ নিয়ে গঠিত।

```text
HEADER.PAYLOAD.SIGNATURE
```

উদাহরণ:

```text
xxxxx.yyyyy.zzzzz
```

## 4.1 Header

Header-এ Token-এর Type এবং Signing Algorithm থাকে।

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

এখানে:

- alg = Signing Algorithm
- typ = Token Type

Common Algorithms:

| Algorithm | Description |
|---|---|
| HS256 | HMAC SHA-256 |
| RS256 | RSA SHA-256 |
| ES256 | ECDSA SHA-256 |

## 4.2 Payload

Payload-এ User সম্পর্কিত Claims থাকে।

```json
{
  "sub": "101",
  "email": "admin@gmail.com",
  "role": "Admin",
  "iss": "Madrasa.API",
  "aud": "Madrasa.Angular",
  "iat": 1793400000,
  "exp": 1793403600
}
```

Common JWT Claims:

| Claim | Meaning |
|---|---|
| sub | Subject / User ID |
| iss | Issuer |
| aud | Audience |
| exp | Expiration Time |
| iat | Issued At |
| nbf | Not Before |
| jti | Unique Token ID |

## 4.3 Signature

Signature নিশ্চিত করে যে Token তৈরি হওয়ার পর সেটি পরিবর্তন করা হয়নি।

উদাহরণ:

```text
HMACSHA256(
    Base64Url(Header) + "." +
    Base64Url(Payload),
    SecretKey
)
```

যদি কেউ Payload পরিবর্তন করে, Signature Validation ব্যর্থ হবে।

### Important Security Note

JWT-এর Payload সাধারণত Base64Url encoded থাকে, Encrypted নয়।

অর্থাৎ, যে কেউ Token Decode করে Payload দেখতে পারে।

তাই JWT Payload-এ কখনো রাখবে না:

- Password
- Database Connection String
- Secret Key
- Credit Card Information
- Sensitive Personal Information

# 5. How JWT Works

JWT Authentication-এর সাধারণ Workflow:

```text
       Angular Application
               |
               |
          Login Request
               |
               v
       ASP.NET Core API
               |
               |
       Validate Credentials
               |
               v
       Generate JWT Token
               |
               |
        Return Access Token
               |
               v
       Angular Stores Token
               |
               |
       Send API Request
               |
               v
       Authorization Header
               |
               v
       JWT Bearer Middleware
               |
               |
       Validate Token Signature
       Validate Issuer
       Validate Audience
       Validate Expiration
               |
               v
       Authorization Check
               |
               v
          API Response
```

### Step-by-Step

**Step 1:** User Login Request পাঠায়।

**Step 2:** Backend Database থেকে User খুঁজে বের করে।

**Step 3:** Password Hash Verify করে।

**Step 4:** Credentials সঠিক হলে Access Token তৈরি করে।

**Step 5:** Token Angular-এ পাঠানো হয়।

**Step 6:** Angular পরবর্তী API Request-এ Token পাঠায়।

**Step 7:** Backend Token Validate করে।

**Step 8:** User-এর Role ও Permission অনুযায়ী Access দেয়।

# 6. Access Token vs Refresh Token

Professional Authentication System-এ সাধারণত দুই ধরনের Token ব্যবহার করা হয়।

## 6.1 Access Token

Access Token API Access করার জন্য ব্যবহৃত হয়।

Example:

```text
Access Token Expiry: 15 Minutes
```

এর কাজ:

- API Request Authenticate করা।
- User Identity বহন করা।
- Role এবং Scope অনুযায়ী Access দেওয়া।

## 6.2 Refresh Token

Refresh Token ব্যবহার করে নতুন Access Token পাওয়া যায়।

ধরো:

```text
Access Token Expired!

Refresh Token Still Valid!
```

এখন User-কে আবার Login করতে হবে না।

Angular Refresh Token পাঠাবে এবং Backend নতুন Access Token ও Refresh Token প্রদান করবে।

## Comparison

| Feature | Access Token | Refresh Token |
|---|---|---|
| Purpose | API Access | New Access Token |
| Lifetime | Short | Longer |
| API Access | Yes | No |
| Database Storage | সাধারণত প্রয়োজন হয় না | Server-side state রাখা হয় |
| Revocation | JWT হলে মেয়াদ শেষ হওয়া পর্যন্ত Valid থাকতে পারে | Server থেকে Revoke করা যায় |

### Example

```text
Access Token = 15 Minutes

Refresh Token = 7 Days
```

এগুলো শুধু Example Configuration। প্রকৃত Expiry Application-এর Security Requirement অনুযায়ী নির্ধারণ করতে হবে।

### Refresh Token Rotation

Professional System-এ প্রতিবার Refresh Request সফল হলে:

1. পুরোনো Refresh Token বাতিল হবে।
2. নতুন Access Token তৈরি হবে।
3. নতুন Refresh Token তৈরি হবে।
4. নতুন Refresh Token Database-এ সংরক্ষিত হবে।

এটিকে Refresh Token Rotation বলে।

পুরোনো Token আবার ব্যবহার হলে Token Reuse Detection করে সংশ্লিষ্ট Token Family Revoke করা যেতে পারে।

# 7. What is OAuth 2.0?

**OAuth 2.0 = Open Authorization 2.0**

OAuth 2.0 হলো একটি Authorization Framework।

এর মাধ্যমে কোনো Application অন্য একটি Application-এর Resource নির্দিষ্ট Permission বা Scope অনুযায়ী Access করতে পারে।

সহজ উদাহরণ:

ধরো, তুমি একটি Application তৈরি করেছ।

তুমি চাও User Google Account দিয়ে Login করুক।

এখন তোমার Application-এর Google Password সরাসরি জানার প্রয়োজন নেই।

Google একটি Authorization Server হিসেবে কাজ করবে।

User Google-এ Authentication এবং Consent সম্পন্ন করার পর তোমার Application প্রয়োজনীয় Authorization পাবে।

Example:

```text
User
 |
 |
 Angular Application
 |
 |
 Redirect to Google
 |
 |
 Google Authorization Server
 |
 |
 User Authentication
 |
 |
 Authorization Code
 |
 |
 Token Exchange
 |
 |
 Application Receives Tokens
```

## OAuth 2.0 কেন ব্যবহার করব?

1. Third-party Application Integration।
2. Delegated Authorization।
3. Scope-based Access Control।
4. Secure API Access।
5. Single Sign-On Architecture-এর ভিত্তি হিসেবে ব্যবহার।
6. Microservices এবং Distributed Systems-এর API Security।

### Real-World Examples

- Login with Google
- Login with Microsoft
- Login with GitHub
- Third-party Payment API
- Google Drive API Access

**Important:** OAuth 2.0 মূলত Authorization-এর জন্য। User Authentication-এর জন্য OAuth-এর উপর OpenID Connect (OIDC) ব্যবহার করা হয়।

# 8. OAuth 2.0 Flow

OAuth 2.0-এ সাধারণত চারটি গুরুত্বপূর্ণ Entity থাকে।

| Entity | Description |
|---|---|
| Resource Owner | User |
| Client | যে Application Access চায় |
| Authorization Server | Token প্রদান করে |
| Resource Server | Protected API / Resource |

## Authorization Code Flow with PKCE

Angular-এর মতো Public Client-এর জন্য Authorization Code Flow with PKCE ব্যবহার করা উচিত।

Workflow:

```text
             User
              |
              v
       Angular Application
              |
              |
       Generate PKCE Verifier
              |
              |
       Generate Code Challenge
              |
              v
      Authorization Server
              |
              |
       User Login / Consent
              |
              v
       Authorization Code
              |
              v
       Angular Callback
              |
              |
       Send Code + Verifier
              |
              v
      Authorization Server
              |
              |
       Validate PKCE
              |
              v
       Access Token
              |
              v
       Protected API
```

### PKCE কী?

PKCE = Proof Key for Code Exchange

এটি Authorization Code Flow-কে আরও নিরাপদ করে।

দুটি গুরুত্বপূর্ণ Value:

```text
code_verifier
code_challenge
```

Client প্রথমে একটি Random `code_verifier` তৈরি করে।

এরপর SHA-256 ব্যবহার করে `code_challenge` তৈরি করে।

Authorization Request-এ Challenge পাঠানো হয়।

Token Request-এর সময় Original Verifier পাঠানো হয়।

Authorization Server দুটো মিলিয়ে যাচাই করে।

এর ফলে অন্য কেউ Authorization Code চুরি করলেও Verifier ছাড়া সহজে Token নিতে পারবে না।

**Professional Recommendation:** Angular SPA-এর মতো Public Client-এর জন্য Authorization Code + PKCE ব্যবহার করো।

# 9. OAuth 2.0 Grant Types

OAuth 2.0-এ বিভিন্ন Grant Type রয়েছে।

| Grant Type | Use Case |
|---|---|
| Authorization Code + PKCE | Web SPA / Mobile App |
| Client Credentials | Machine-to-Machine API |
| Device Authorization | Input-limited devices |
| Refresh Token | Access Token Renew |

### 9.1 Authorization Code + PKCE

User-based Application-এর জন্য।

Example:

```text
Angular App → Identity Provider → Login → Token
```

### 9.2 Client Credentials

একটি Backend Service অন্য একটি Backend Service-এর API Access করবে।

Example:

```text
Order Service
     |
     v
Authorization Server
     |
     v
Client Credentials
     |
     v
Access Token
     |
     v
Payment Service
```

এখানে সাধারণত কোনো End User সরাসরি Login করে না।

### 9.3 Refresh Token Grant

Access Token Expire হলে Refresh Token ব্যবহার করে নতুন Token পাওয়া যায়।

### Deprecated / Insecure Grants

নিচের পুরোনো Flow ব্যবহার করা উচিত নয়:

- Implicit Grant
- Resource Owner Password Credentials Grant

বর্তমান OAuth Security Best Current Practice অনুযায়ী এগুলো এড়িয়ে চলতে হবে।

# 10. JWT vs OAuth 2.0

অনেক Developer JWT এবং OAuth 2.0-কে একই বিষয় মনে করেন। কিন্তু এদের কাজ আলাদা।

| Feature | JWT | OAuth 2.0 |
|---|---|---|
| Type | Token Format | Authorization Framework |
| Purpose | Claims বহন | Delegated Authorization |
| Token | JWT Token | JWT অথবা Opaque Token |
| Login | নিজে Login Protocol নয় | নিজে User Authentication Protocol নয় |
| API Security | Bearer Token হিসেবে ব্যবহার করা যায় | Access Delegation নিয়ন্ত্রণ করে |
| Scope | Claim হিসেবে থাকতে পারে | Scope-based Authorization |

### সহজভাবে মনে রাখো

```text
JWT = Token Format

OAuth 2.0 = Authorization Framework

OpenID Connect = Authentication Layer
```

OAuth 2.0-এর Access Token JWT হতে পারে, আবার Opaque Token-ও হতে পারে।

# 11. What is OpenID Connect (OIDC)?

OIDC = OpenID Connect

এটি OAuth 2.0-এর উপর তৈরি একটি Authentication Layer।

OAuth 2.0 বলে:

```text
এই Application নির্দিষ্ট Resource Access করতে পারবে।
```

OIDC বলে:

```text
এই User কে?
```

OIDC-এর মাধ্যমে সাধারণত ID Token পাওয়া যায়।

### ID Token vs Access Token

| Token | Purpose |
|---|---|
| ID Token | User Authentication Information |
| Access Token | API Access |
| Refresh Token | New Access Token Obtain |

OIDC-এর ID Token সাধারণত JWT Format-এ থাকে।

### Example Providers

- Microsoft Entra ID
- Google Identity
- Keycloak
- Auth0
- Okta

# 12. Professional Architecture

একটি Professional Angular + ASP.NET Core Application-এ Authentication Architecture এমন হতে পারে।

```text
                 Angular 20
                     |
                     |
              Login / Logout
                     |
                     v
             Identity Provider
                     |
                     |
              OAuth 2.0 + OIDC
                     |
                     |
              Access Token
                     |
                     v
             ASP.NET Core API
                     |
                     |
               JWT Validation
                     |
                     |
              Authorization
                     |
                     v
                Application
                     |
                     v
                 SQL Server
```

### Recommended Technologies

| Layer | Technology |
|---|---|
| Frontend | Angular 20 |
| Backend | ASP.NET Core 8 |
| Authentication | OpenID Connect |
| Authorization | OAuth 2.0 |
| Token Format | JWT |
| Identity Provider | Keycloak / Entra ID |
| Database | SQL Server |
| API Security | JWT Bearer Authentication |
| Transport Security | HTTPS / TLS |

**Architecture Decision:**

- নিজস্ব User Registration, Login, Role এবং Permission Management প্রয়োজন হলে ASP.NET Core Identity + OpenIddict বিবেচনা করতে পারো।
- Enterprise SSO প্রয়োজন হলে Microsoft Entra ID বা Keycloak ব্যবহার করতে পারো।
- শুধুমাত্র API-তে JWT Validate করতে হলে ASP.NET Core JWT Bearer Middleware ব্যবহার করা যায়।

---

# 13. ASP.NET Core 8 JWT Implementation

এবার বাস্তব উদাহরণ দেখি।

আমাদের Project:

```text
JwtDemo
 |
 |---- JwtDemo.Api
 |
 |---- Controllers
 |      |
 |      |---- AuthController.cs
 |      |---- UsersController.cs
 |
 |---- appsettings.json
 |
 |---- Program.cs
```

এই উদাহরণে একটি Trusted Identity Provider থেকে পাওয়া JWT Validate করব।

Production-এ নিজস্ব Authorization Server হিসেবে কাজ করার জন্য শুধু JWT Bearer Middleware যথেষ্ট নয়।

## Step 1: Create Project

Visual Studio 2022:

```text
Create New Project
      |
      v
ASP.NET Core Web API
      |
      v
Project Name: JwtDemo.Api
      |
      v
Framework: .NET 8
      |
      v
Create
```

## Step 2: Install NuGet Package

Package Manager Console:

```powershell
Install-Package Microsoft.AspNetCore.Authentication.JwtBearer
```

## Step 3: Configure appsettings.json

```json
{
  "Jwt": {
    "Issuer": "https://identity.example.com",
    "Audience": "Madrasa.Api"
  }
}
```

এখানে:

- Issuer = যে Trusted Server Token তৈরি করেছে।
- Audience = যে API-এর জন্য Token তৈরি হয়েছে।

**Note:** Production-এ Signing Key বা Client Secret সরাসরি appsettings.json কিংবা GitHub-এ রাখবে না। Trusted Identity Provider-এর Signing Key সাধারণত OpenID Connect Discovery/JWKS-এর মাধ্যমে পাওয়া যায়।

## Step 4: Configure Program.cs

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services.AddAuthentication(
    JwtBearerDefaults.AuthenticationScheme
)
.AddJwtBearer(options =>
{
    options.Authority =
        builder.Configuration["Jwt:Issuer"];

    options.Audience =
        builder.Configuration["Jwt:Audience"];

    options.RequireHttpsMetadata = true;

    options.TokenValidationParameters =
        new TokenValidationParameters
        {
            ValidateIssuer = true,

            ValidateAudience = true,

            ValidateLifetime = true,

            ValidateIssuerSigningKey = true,

            ClockSkew = TimeSpan.FromSeconds(30)
        };
});

builder.Services.AddAuthorization();

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

app.UseCors("AngularPolicy");

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();

app.Run();
```

### Important: Middleware Order

```csharp
app.UseAuthentication();

app.UseAuthorization();
```

প্রথমে Authentication হবে, তারপর Authorization হবে।

এগুলো ভুল Order-এ রাখলে Authentication এবং Authorization সমস্যা হতে পারে।

## Step 5: Create UsersController.cs

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

namespace JwtDemo.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpGet("public")]
    [AllowAnonymous]
    public IActionResult Public()
    {
        return Ok("Public API");
    }

    [HttpGet("profile")]
    [Authorize]
    public IActionResult Profile()
    {
        var userId = User.FindFirstValue(
            ClaimTypes.NameIdentifier
        );

        var email = User.FindFirstValue(
            ClaimTypes.Email
        );

        return Ok(new
        {
            UserId = userId,
            Email = email,
            Message = "Authenticated User"
        });
    }

    [HttpGet("admin")]
    [Authorize(Roles = "Admin")]
    public IActionResult Admin()
    {
        return Ok("Welcome Admin!");
    }
}
```

### এখানে কী হচ্ছে?

`[AllowAnonymous]`

সবাই API Access করতে পারবে।

`[Authorize]`

শুধু Authenticated User API Access করতে পারবে।

`[Authorize(Roles = "Admin")]`

শুধু Admin Role-এর User API Access করতে পারবে।

**Important:** Token-এর Claim Mapping Identity Provider অনুযায়ী ভিন্ন হতে পারে। তাই `sub`, `role`, `roles`, `ClaimTypes.NameIdentifier` ইত্যাদি প্রয়োজন অনুযায়ী Mapping Configure করতে হবে।

## Step 6: Test Using Swagger

Public API:

```http
GET /api/users/public
```

Response:

```json
{
  "message": "Public API"
}
```

Protected API:

```http
GET /api/users/profile
Authorization: Bearer YOUR_ACCESS_TOKEN
```

Response:

```json
{
  "userId": "101",
  "email": "admin@gmail.com",
  "message": "Authenticated User"
}
```

Admin API:

```http
GET /api/users/admin
Authorization: Bearer YOUR_ACCESS_TOKEN
```

যদি User-এর Role Admin হয়:

```text
200 OK
```

অন্যথায়:

```text
403 Forbidden
```

---

# 14. Protect API Endpoints

ASP.NET Core-এ Authorization-এর জন্য কয়েকটি গুরুত্বপূর্ণ Attribute রয়েছে।

## 14.1 Authorize

```csharp
[Authorize]
[HttpGet("dashboard")]
public IActionResult Dashboard()
{
    return Ok("Dashboard Data");
}
```

## 14.2 Role-Based Authorization

```csharp
[Authorize(Roles = "Admin")]
[HttpGet("users")]
public IActionResult GetUsers()
{
    return Ok();
}
```

একাধিক Role:

```csharp
[Authorize(Roles = "Admin,SuperAdmin")]
```

## 14.3 Policy-Based Authorization

Program.cs:

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy(
        "ManageUsers",
        policy =>
        {
            policy.RequireAuthenticatedUser();

            policy.RequireClaim(
                "permission",
                "users.manage"
            );
        }
    );
});
```

Controller:

```csharp
[Authorize(Policy = "ManageUsers")]
[HttpPost("users")]
public IActionResult CreateUser()
{
    return Ok("User Created");
}
```

### Role vs Permission

Role বলে User কোন Group-এর অন্তর্ভুক্ত।

Permission বলে User নির্দিষ্ট কোন কাজ করতে পারবে।

উদাহরণ:

```text
Role: Accountant

Permissions:
- payment.view
- payment.create
- payment.export
```

Enterprise Application-এ Role এবং Permission একসঙ্গে ব্যবহার করা যায়।

---

# 15. Angular 20 JWT Integration

এবার Angular Frontend থেকে Protected API Access করব।

## Step 1: Create Auth Service

`auth.service.ts`

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class AuthService {

  private readonly TOKEN_KEY = 'access_token';

  setToken(token: string): void {
    sessionStorage.setItem(
      this.TOKEN_KEY,
      token
    );
  }

  getToken(): string | null {
    return sessionStorage.getItem(
      this.TOKEN_KEY
    );
  }

  isAuthenticated(): boolean {
    return !!this.getToken();
  }

  logout(): void {
    sessionStorage.removeItem(
      this.TOKEN_KEY
    );
  }
}
```

**Security Note:** এই উদাহরণটি শেখার জন্য। `sessionStorage`-এ রাখা Token JavaScript-এর মাধ্যমে Access করা যায়, তাই XSS হলে চুরি হতে পারে। Production SPA-তে Backend-for-Frontend (BFF) এবং Secure, HttpOnly, SameSite Cookie Architecture বিবেচনা করা উচিত।

## Step 2: Create JWT Interceptor

Angular 20-এ Functional Interceptor ব্যবহার করা যায়।

`auth.interceptor.ts`

```typescript
import {
  HttpInterceptorFn
} from '@angular/common/http';

import { inject } from '@angular/core';

import { AuthService } from './auth.service';

export const authInterceptor: HttpInterceptorFn =
  (req, next) => {

    const authService = inject(AuthService);

    const token = authService.getToken();

    if (token) {

      const authReq = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`
        }
      });

      return next(authReq);
    }

    return next(req);
  };
```

### Interceptor কেন ব্যবহার করব?

প্রতিটি API Request-এ Manually Token পাঠাতে হবে না।

Interceptor Automatically Token যোগ করে।

## Step 3: Register Interceptor

`app.config.ts`

```typescript
import {
  ApplicationConfig
} from '@angular/core';

import {
  provideHttpClient,
  withInterceptors
} from '@angular/common/http';

import {
  authInterceptor
} from './core/interceptors/auth.interceptor';

export const appConfig: ApplicationConfig = {

  providers: [

    provideHttpClient(
      withInterceptors([
        authInterceptor
      ])
    )

  ]

};
```

## Step 4: Call Protected API

`user.service.ts`

```typescript
import { Injectable } from '@angular/core';

import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class UserService {

  private apiUrl =
    'https://localhost:7100/api/users';

  constructor(
    private http: HttpClient
  ) {}

  getProfile() {

    return this.http.get(
      `${this.apiUrl}/profile`
    );

  }
}
```

এখন Angular Request পাঠাবে:

```http
GET /api/users/profile

Authorization: Bearer eyJhbGciOi...
```

Backend Token Validate করবে।

---

# 16. Refresh Token Implementation

Professional Authentication System-এ Refresh Token খুব গুরুত্বপূর্ণ।

## Workflow

```text
Angular Application
        |
        v
API Request
        |
        v
Access Token Expired
        |
        v
401 Unauthorized
        |
        v
Refresh Token Request
        |
        v
ASP.NET Core API
        |
        v
Validate Refresh Token
        |
        v
Revoke Old Refresh Token
        |
        v
Generate New Token Pair
        |
        v
Return New Tokens
        |
        v
Retry Original API Request
```

## Database Design

SQL Server:

```sql
CREATE TABLE RefreshTokens
(
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,

    UserId BIGINT NOT NULL,

    TokenHash NVARCHAR(256) NOT NULL,

    ExpiresAt DATETIME2 NOT NULL,

    CreatedAt DATETIME2 NOT NULL
        DEFAULT SYSUTCDATETIME(),

    RevokedAt DATETIME2 NULL,

    ReplacedByTokenHash NVARCHAR(256) NULL,

    CONSTRAINT FK_RefreshTokens_Users
        FOREIGN KEY (UserId)
        REFERENCES Users(Id)
);
```

Recommended Index:

```sql
CREATE UNIQUE INDEX
IX_RefreshTokens_TokenHash
ON RefreshTokens(TokenHash);
```

### Database-এ কেন Refresh Token Hash রাখব?

Refresh Token হলো একটি Sensitive Credential।

তাই Plain Text Token Database-এ না রেখে Cryptographic Hash সংরক্ষণ করা উচিত।

উদাহরণ:

```text
Original Refresh Token:
abc123xyz...

Database:
SHA256(abc123xyz...)
```

Refresh Request এলে Server Token Hash করে Database-এর সঙ্গে Match করবে।

## Refresh Endpoint Example

```csharp
[HttpPost("refresh")]
[AllowAnonymous]
public async Task<IActionResult> Refresh(
    [FromBody] RefreshTokenRequest request)
{
    var result = await _authService.RefreshAsync(
        request.RefreshToken
    );

    if (result is null)
    {
        return Unauthorized();
    }

    return Ok(result);
}
```

এখানে `_authService.RefreshAsync()`-এর ভিতরে:

1. Refresh Token Hash করতে হবে।
2. Database থেকে Token খুঁজতে হবে।
3. Expiration Check করতে হবে।
4. Revocation Check করতে হবে।
5. Token User এবং Client Binding Verify করতে হবে।
6. নতুন Access Token Generate করতে হবে।
7. নতুন Refresh Token Generate করতে হবে।
8. Database Transaction-এর মাধ্যমে পুরোনো Token Revoke এবং নতুন Token Insert করতে হবে।

**Important:** Refresh Token Rotation অবশ্যই Atomic Transaction-এর মাধ্যমে করতে হবে। Concurrent Refresh Request এবং Token Reuse Detection-ও Handle করতে হবে।

---

# 17. Professional Security Best Practices

Production Application-এ নিচের Security Practices অনুসরণ করা উচিত।

## 17.1 Always Use HTTPS

```text
HTTP ❌

HTTPS ✅
```

Token কখনো HTTP-এর মাধ্যমে পাঠাবে না।

## 17.2 Use Short-Lived Access Tokens

Recommended Example:

```text
Access Token: 5 - 15 Minutes
```

প্রকৃত Expiry Security Requirement অনুযায়ী নির্ধারণ করতে হবে।

## 17.3 Secure Refresh Token

- Refresh Token Hash করে Database-এ রাখবে।
- Refresh Token Rotation Implement করবে।
- Expiration Maintain করবে।
- Logout হলে Revoke করবে।
- Password Change হলে Active Sessions Revoke করার Policy রাখবে।

## 17.4 Validate JWT Properly

সবসময় Validate করবে:

```text
Signature
Issuer
Audience
Expiration
Not Before
Algorithm
```

শুধু Token Decode করে বিশ্বাস করা যাবে না।

## 17.5 Never Store Sensitive Information in JWT

JWT Payload Encrypted নয়।

তাই Password বা Secret রাখবে না।

## 17.6 Use Strong Signing Algorithm

HS256 ব্যবহার করলে শক্তিশালী Random Secret প্রয়োজন।

Distributed Enterprise System-এ RS256 বা ES256-এর মতো Asymmetric Signing বিবেচনা করতে পারো।

এক্ষেত্রে:

```text
Private Key = Sign Token

Public Key = Verify Token
```

Private Key কখনো Public Repository-তে রাখবে না।

## 17.7 Use Role and Permission

```text
[Authorize(Roles = "Admin")]
```

অথবা:

```text
[Authorize(Policy = "payment.create")]
```

## 17.8 Prevent XSS and CSRF

- Angular Application-এ Strict Content Security Policy বিবেচনা করো।
- Untrusted HTML Render করা এড়িয়ে চলো।
- HttpOnly Cookie ব্যবহার করলে CSRF Protection প্রয়োজন।
- Cookie-এর Secure এবং SameSite Configuration ঠিক রাখো।
- Production-এ BFF Pattern বিবেচনা করো।

## 17.9 Token Revocation

JWT Stateless হলেও Token Revocation-এর প্রয়োজন হতে পারে।

সমাধান:

- Short Token Lifetime
- Refresh Token Revocation
- Token Version / Security Stamp
- Denylist (প্রয়োজন হলে)
- Identity Provider Session Revocation

## 17.10 Never Use These in Production

```text
❌ Hardcoded JWT Secret

❌ HTTP for Token Transmission

❌ Long-lived Access Token

❌ Plain Text Refresh Token Storage

❌ Implicit Grant

❌ Resource Owner Password Grant

❌ Trusting Decoded JWT Without Validation

❌ Storing Password in JWT Payload
```

---

# 18. Common JWT Errors

| Status Code | Meaning | Possible Reason |
|---|---|---|
| 401 | Unauthorized | Token Missing |
| 401 | Unauthorized | Invalid Signature |
| 401 | Unauthorized | Expired Token |
| 401 | Unauthorized | Invalid Issuer |
| 401 | Unauthorized | Invalid Audience |
| 403 | Forbidden | Insufficient Permission |
| 500 | Internal Server Error | Backend Configuration / Code Error |

## Common Problems

### Problem 1: 401 Unauthorized

সম্ভাব্য কারণ:

- Authorization Header পাঠানো হয়নি।
- Token Expire হয়েছে।
- Signing Key ভুল।
- Issuer Mismatch।
- Audience Mismatch।

### Problem 2: 403 Forbidden

Token Valid কিন্তু User-এর Permission নেই।

Example:

```text
User Role = Employee

Required Role = Admin

Result = 403 Forbidden
```

### Problem 3: User.Claims Count = 0

সম্ভাব্য কারণ:

- Authentication Middleware Configure করা হয়নি।
- `UseAuthentication()` Missing।
- Middleware Order ভুল।
- JWT Validation Failed।
- Bearer Token পাঠানো হয়নি।

### Problem 4: Role Authorization Not Working

সম্ভাব্য কারণ:

- Role Claim Mapping ভুল।
- Identity Provider `roles` পাঠাচ্ছে, কিন্তু API `role` খুঁজছে।
- Role Claim Type Configure করা হয়নি।

### Debugging Checklist

```text
[ ] Token Present?
[ ] Token Signature Valid?
[ ] Issuer Correct?
[ ] Audience Correct?
[ ] Expiration Valid?
[ ] Authentication Middleware Enabled?
[ ] Authorization Middleware Enabled?
[ ] Role Claim Mapping Correct?
[ ] Permission Claim Available?
```

---

# 19. Real-World Example

ধরো, আমাদের একটি Madrasa Donation Platform রয়েছে।

Technology:

```text
Frontend: Angular 20

Backend: ASP.NET Core 8 Web API

Database: SQL Server

Authentication: OAuth 2.0 / OpenID Connect

Token: JWT

Authorization: Role + Permission
```

আমাদের System-এ চার ধরনের User আছে।

| Role | Access |
|---|---|
| SuperAdmin | Full Access |
| Admin | Organization Management |
| Accountant | Donation & Payment |
| Employee | Assigned Features |

### Example API Endpoints

| Endpoint | Authorization |
|---|---|
| GET /api/public/notices | Anonymous |
| GET /api/admin/dashboard | Admin |
| POST /api/donations | Authenticated |
| GET /api/payments | payment.view |
| POST /api/payments | payment.create |
| GET /api/reports | report.view |
| GET /api/superadmin/organizations | SuperAdmin |

### Example JWT Claims

```json
{
  "sub": "101",
  "email": "admin@example.com",
  "role": "Admin",
  "permissions": [
    "donation.view",
    "donation.create",
    "payment.view",
    "report.view"
  ],
  "organizationId": "10",
  "iss": "https://identity.example.com",
  "aud": "Madrasa.Api",
  "exp": 1793400900
}
```

**Multi-Tenant Security:**

Multi-Tenant Application-এ শুধু JWT-এর `organizationId` Claim-এর উপর নির্ভর করা যাবে না।

প্রতিটি Request-এ:

1. Token Validate করতে হবে।
2. Authenticated User Identify করতে হবে।
3. User-এর Organization Membership Verify করতে হবে।
4. Database Query-তে Tenant Isolation Enforce করতে হবে।
5. Unauthorized Tenant-এর Data Access Block করতে হবে।

এভাবে একটি Secure Multi-Tenant Application তৈরি করা যায়।

---

# 20. Interview Questions

### Q1. What is JWT?

JWT হলো একটি Compact, URL-safe Token Format, যা Claims আদান-প্রদানের জন্য ব্যবহৃত হয়।

### Q2. What are the three parts of JWT?

Header, Payload এবং Signature।

### Q3. Is JWT Encrypted?

না। সাধারণ Signed JWT-এর Payload Base64Url Encoded থাকে, Encrypted নয়।

### Q4. What is the purpose of JWT Signature?

Token-এর Integrity এবং Authenticity Verify করা।

### Q5. What is the difference between Authentication and Authorization?

Authentication User Identity Verify করে।

Authorization User-এর Access Permission Verify করে।

### Q6. What is OAuth 2.0?

OAuth 2.0 একটি Authorization Framework, যার মাধ্যমে Application অন্য Resource Access করার অনুমতি পায়।

### Q7. What is OpenID Connect?

OIDC হলো OAuth 2.0-এর উপর তৈরি Authentication Layer।

### Q8. What is the difference between Access Token and Refresh Token?

Access Token API Access করে।

Refresh Token ব্যবহার করে নতুন Access Token পাওয়া যায়।

### Q9. What is Bearer Token?

Bearer Token হলো এমন একটি Token, যার Possession থাকলে এবং Token Valid হলে সেটি ব্যবহার করে Protected Resource Access করা যায়।

### Q10. What is PKCE?

PKCE হলো Authorization Code Flow-এর Security Extension, যা Code Verifier এবং Code Challenge ব্যবহার করে Authorization Code-এর অপব্যবহার প্রতিরোধ করে।

### Q11. What is the difference between 401 and 403?

401 = Authentication Failed বা Valid Authentication নেই।

403 = Authentication সফল, কিন্তু Access Permission নেই।

### Q12. What is JWT Stateless Authentication?

Server প্রতিটি Request-এ Token Validate করে। সাধারণত প্রতিটি Access Token-এর জন্য Server-side Session Lookup প্রয়োজন হয় না।

### Q13. What is JWT Expiration?

`exp` Claim Token-এর Expiration Time নির্দেশ করে।

### Q14. Why should we use Refresh Token Rotation?

পুরোনো Refresh Token বাতিল করে নতুন Token প্রদান করা হয়। Token Reuse Detect করতেও সাহায্য করে।

### Q15. Can OAuth 2.0 use JWT?

হ্যাঁ। OAuth 2.0 Access Token হিসেবে JWT অথবা Opaque Token ব্যবহার করতে পারে।

### Q16. What is the difference between JWT and Session Authentication?

Session Authentication-এ Server-side Session State থাকে।

JWT Authentication-এ Signed Token-এর Claims Validate করে Authentication করা যায়।

### Q17. What is the purpose of Issuer and Audience?

Issuer Token কে তৈরি করেছে তা নির্দেশ করে।

Audience Token কোন Resource Server-এর জন্য তৈরি হয়েছে তা নির্দেশ করে।

### Q18. What is Client Credentials Flow?

একটি Machine বা Backend Service অন্য একটি Service-এর API Access করার জন্য Client Credentials ব্যবহার করে Token নেয়।

### Q19. Should we store JWT in LocalStorage?

LocalStorage সুবিধাজনক হলেও XSS-এর কারণে Token চুরি হতে পারে। Production Architecture-এ BFF এবং Secure HttpOnly Cookie বিবেচনা করা উচিত।

### Q20. Is JWT enough for Enterprise Authentication?

সবসময় নয়। Enterprise Authentication-এর জন্য Identity Provider, OIDC, OAuth 2.0, MFA, Token Lifecycle, Role/Permission এবং Session Management প্রয়োজন হতে পারে।

---

# 21. Conclusion

JWT এবং OAuth 2.0 আধুনিক Web Application Security-এর গুরুত্বপূর্ণ অংশ।

মনে রাখবে:

```text
Authentication
      |
      v
OpenID Connect
      |
      v
OAuth 2.0 Authorization
      |
      v
Access Token
      |
      v
JWT Validation
      |
      v
Authorization
      |
      v
Protected API
```

Professional Developer হিসেবে তোমার যেসব বিষয় ভালোভাবে জানা উচিত:

- Authentication vs Authorization
- JWT Structure and Claims
- JWT Signature Validation
- Access Token and Refresh Token
- OAuth 2.0 Authorization Code + PKCE
- OpenID Connect
- Role-Based Authorization
- Policy-Based Authorization
- Refresh Token Rotation
- Token Revocation
- HTTPS / TLS
- XSS / CSRF Protection
- Multi-Tenant Authorization
- Identity Provider Integration

**Final Recommendation:**

ছোট বা Medium Application-এর ক্ষেত্রে ASP.NET Core Identity + JWT Authentication দিয়ে শুরু করতে পারো।

Enterprise Application-এর ক্ষেত্রে OpenID Connect + OAuth 2.0 + Dedicated Identity Provider ব্যবহার করো।

---

## ✅ সারসংক্ষেপ

- **JWT** = স্বাক্ষরিত token ফরম্যাট (Header.Payload.Signature); এনক্রিপ্টেড নয়
- **OAuth 2.0** = সীমিত অনুমতি দেওয়ার framework; **OIDC** = তার উপর login
- Access (ছোট) + Refresh (দীর্ঘ, বাতিলযোগ্য) token
- **Auth Code + PKCE** (user), **Client Credentials** (service)


**Author:** Full Stack .NET Developer

**Technology Stack:** C# | ASP.NET Core | Angular 20 | SQL Server | JWT | OAuth 2.0 | OpenID Connect

⭐ If this guide helps you, consider giving this repository a Star!

⬅️ [HTTPS ও TLS](14-https-and-tls.md) | ➡️ [SQL Injection ও XSS](16-sql-injection-and-xss.md)
