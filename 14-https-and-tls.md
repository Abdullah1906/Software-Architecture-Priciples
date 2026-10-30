# Chapter ১৪: HTTPS ও TLS

> Concepts, Architecture, SSL/TLS Handshake, Certificates, ASP.NET Core, Angular, IIS, Nginx, Production Deployment এবং Security Best Practices.

---

## 📌 Table of Contents

1. [HTTPS কী?](#1-https-কী)
2. [HTTP vs HTTPS](#2-http-vs-https)
3. [SSL ও TLS কী?](#3-ssl-ও-tls-কী)
4. [HTTPS কেন ব্যবহার করা হয়?](#4-https-কেন-ব্যবহার-করা-হয়)
5. [HTTPS কীভাবে কাজ করে?](#5-https-কীভাবে-কাজ-করে)
6. [TLS Handshake কী?](#6-tls-handshake-কী)
7. [Digital Certificate কী?](#7-digital-certificate-কী)
8. [Public Key ও Private Key](#8-public-key-ও-private-key)
9. [Symmetric vs Asymmetric Encryption](#9-symmetric-vs-asymmetric-encryption)
10. [HTTPS Setup – Local Development](#10-https-setup--local-development)
11. [ASP.NET Core-এ HTTPS Setup](#11-aspnet-core-এ-https-setup)
12. [Angular 20 + .NET API HTTPS](#12-angular-20--net-api-https)
13. [Production Server-এ HTTPS Setup](#13-production-server-এ-https-setup)
14. [Let's Encrypt Certificate](#14-lets-encrypt-certificate)
15. [Nginx Reverse Proxy Configuration](#15-nginx-reverse-proxy-configuration)
16. [IIS-এ HTTPS Configuration](#16-iis-এ-https-configuration)
17. [HTTP to HTTPS Redirect](#17-http-to-https-redirect)
18. [HSTS কী?](#18-hsts-কী)
19. [Professional Security Best Practices](#19-professional-security-best-practices)
20. [Common HTTPS Errors](#20-common-https-errors)
21. [Useful Commands](#21-useful-commands)
22. [Interview Questions & Answers](#22-interview-questions--answers)

---

# 1. HTTPS কী?

**HTTPS = HyperText Transfer Protocol Secure**

HTTPS হলো HTTP-এর secure version। এটি Browser এবং Web Server-এর মধ্যে আদান-প্রদান করা Data-কে TLS (Transport Layer Security) ব্যবহার করে Encrypt করে।

সহজভাবে:

ধরো, একজন User তোমার Website-এ Login করছে।

```text
Username: abdullah
Password: 123456
```

HTTP ব্যবহার করলে Network-এর মধ্যে Data Plain Text হিসেবে যেতে পারে।

কিন্তু HTTPS ব্যবহার করলে Data Encrypt হয়ে Server-এ যায়।

```text
User Browser
     |
     | Encrypted Data
     |
     v
HTTPS / TLS
     |
     v
Web Server
     |
     v
ASP.NET Core API
```

অর্থাৎ, HTTPS Internet-এর মাধ্যমে Data Securely Transfer করতে সাহায্য করে।

### HTTPS-এর Default Port

```text
HTTP  = Port 80
HTTPS = Port 443
```

Example:

```text
http://example.com
https://example.com
```

---

# 2. HTTP vs HTTPS

| বিষয় | HTTP | HTTPS |
|---|---|---|
| Full Form | HyperText Transfer Protocol | HyperText Transfer Protocol Secure |
| Security | Encrypted নয় | TLS Encrypted |
| Default Port | 80 | 443 |
| Data Protection | নেই | আছে |
| Authentication | Server identity verify করে না | Certificate দিয়ে Server verify করে |
| Data Integrity | TLS protection নেই | Integrity protection আছে |
| Login Security | ঝুঁকিপূর্ণ | Secure connection |
| SEO | তুলনামূলকভাবে দুর্বল | Search engines HTTPS-কে গুরুত্ব দেয় |

### Example

HTTP:

```http
POST /api/auth/login HTTP/1.1
Host: example.com
Content-Type: application/json

{
    "email": "admin@gmail.com",
    "password": "123456"
}
```

HTTPS:

```text
Browser
   |
   | TLS Encrypted Connection
   |
   v
Server
```

HTTPS-এ Network-এর মধ্যে একজন Attacker সহজে Original Request-এর Password পড়তে বা Data পরিবর্তন করতে পারবে না।

**মনে রাখবে:** HTTPS Data Transmission Secure করে। কিন্তু এটি তোমার Application-এর SQL Injection, XSS, Weak Password বা Authorization Vulnerability নিজে থেকে ঠিক করে না।

---

# 3. SSL ও TLS কী?

## SSL – Secure Sockets Layer

SSL হলো Internet Communication Secure করার একটি পুরোনো Cryptographic Protocol।

SSL-এর Version:

- SSL 1.0 – Publicly Released হয়নি।
- SSL 2.0 – Deprecated.
- SSL 3.0 – Deprecated.

SSL বর্তমানে Insecure হিসেবে বিবেচিত।

## TLS – Transport Layer Security

TLS হলো SSL-এর Successor এবং আধুনিক Secure Communication Protocol।

TLS Version:

| Version | Status |
|---|---|
| SSL 2.0 | Insecure |
| SSL 3.0 | Insecure |
| TLS 1.0 | Deprecated |
| TLS 1.1 | Deprecated |
| TLS 1.2 | Widely Supported |
| TLS 1.3 | Modern Recommended |

**Professional Recommendation:**

- TLS 1.2 এবং TLS 1.3 Support রাখবে।
- সম্ভব হলে TLS 1.3 Prefer করবে।
- SSL, TLS 1.0 ও TLS 1.1 Disable করবে।

### SSL Certificate vs TLS

অনেক Hosting Provider এখনো SSL Certificate বলে।

কিন্তু Technicalভাবে Modern Website-এ TLS Certificate ব্যবহৃত হয়।

অর্থাৎ, SSL Certificate কথাটি প্রচলিত হলেও Connection Security-এর জন্য আধুনিক Protocol হলো TLS।

---

# 4. HTTPS কেন ব্যবহার করা হয়?

HTTPS-এর প্রধান তিনটি Security Goal রয়েছে।

## 4.1 Confidentiality

Data যেন Unauthorized ব্যক্তি পড়তে না পারে।

Example:

```text
User Password: MyPassword123
```

TLS Encryption-এর মাধ্যমে Network-এ Password Plain Text হিসেবে দৃশ্যমান হয় না।

## 4.2 Integrity

Data Transmission-এর সময় কেউ যেন Data পরিবর্তন করতে না পারে।

Example:

```text
Original Amount: 1000 BDT
Modified Amount: 100 BDT
```

TLS-এর Integrity Protection Unauthorized Modification Detect করতে সাহায্য করে।

## 4.3 Authentication

User যেন নিশ্চিত হতে পারে যে সে সঠিক Website-এর Server-এর সঙ্গে Connected।

Example:

```text
https://mywebsite.com
```

Browser Server-এর Certificate যাচাই করে।

### Real-Life Example

ধরো, তোমার Madrasa Donation Platform-এ একজন User 5,000 BDT Donation করছে।

Payment Information:

```json
{
    "donorName": "Abdullah",
    "amount": 5000,
    "paymentMethod": "bKash"
}
```

HTTPS না থাকলে Network Attacker Sensitive Data Intercept বা Modify করার সুযোগ পেতে পারে।

HTTPS ব্যবহার করলে Transmission Layer Secure হয়।

---

# 5. HTTPS কীভাবে কাজ করে?

HTTPS-এর কাজ বোঝার জন্য নিচের Architecture দেখো।

```text
+-------------------------+
|       Web Browser       |
|     Angular 20 App      |
+------------+------------+
             |
             | 1. HTTPS Request
             |
             v
+-------------------------+
|      TLS Protocol       |
| Encryption + Integrity  |
+------------+------------+
             |
             | 2. Secure Channel
             |
             v
+-------------------------+
|       Web Server        |
|    Nginx / IIS          |
+------------+------------+
             |
             | 3. Forward Request
             |
             v
+-------------------------+
|     ASP.NET Core API    |
+------------+------------+
             |
             v
+-------------------------+
|       SQL Server        |
+-------------------------+
```

### Step-by-Step Process

1. User Browser-এ HTTPS Website Open করে।
2. Browser Server-এর সঙ্গে TLS Handshake শুরু করে।
3. Server তার Digital Certificate পাঠায়।
4. Browser Certificate Validate করে।
5. Client ও Server Cryptographic Keys Establish করে।
6. Secure Connection তৈরি হয়।
7. Browser Encrypted HTTP Request পাঠায়।
8. Server Request Decrypt করে Application-এ পাঠায়।
9. Server Response Encrypt করে Browser-এ পাঠায়।

---

# 6. TLS Handshake কী?

TLS Handshake হলো Client এবং Server-এর মধ্যে Secure Connection Establish করার Process।

এখানে TLS 1.3-এর Simplified Handshake দেখানো হয়েছে।

### TLS 1.3 Handshake Flow

```text
        CLIENT (Browser)
              |
              | ClientHello
              | Supported TLS Version
              | Cipher Suites
              | Key Share
              |
              v
        SERVER
              |
              | ServerHello
              | Selected Cipher Suite
              | Key Share
              |
              | EncryptedExtensions
              | Certificate
              | CertificateVerify
              | Finished
              |
              v
        CLIENT
              |
              | Certificate Validation
              | Finished
              |
              v
       SECURE CONNECTION
              |
              | Encrypted HTTP Data
              |
              v
           SERVER
```

### Step Explanation

**Step 1: ClientHello**

Browser Server-কে জানায়:

- Supported TLS Versions
- Supported Cipher Suites
- Key Share
- অন্যান্য TLS Parameters

**Step 2: ServerHello**

Server জানায়:

- Selected TLS Version
- Selected Cipher Suite
- Key Share

**Step 3: Certificate**

Server তার Digital Certificate পাঠায়।

**Step 4: Certificate Validation**

Browser যাচাই করে:

- Certificate Trusted CA দ্বারা Signed কি না।
- Domain Name Match করে কি না।
- Certificate Expired কি না।
- Certificate Chain Valid কি না।

**Step 5: Key Establishment**

Client এবং Server Ephemeral Key Exchange ব্যবহার করে Shared Secret তৈরি করে।

**Step 6: Secure Communication**

Handshake Complete হলে Encrypted Communication শুরু হয়।

> TLS 1.3 সাধারণত 1-RTT Handshake ব্যবহার করে। Session Resumption-এর ক্ষেত্রে 0-RTT Early Data সম্ভব, তবে এতে Replay Risk রয়েছে।

---

# 7. Digital Certificate কী?

Digital Certificate হলো একটি Electronic Document, যা Website বা Server-এর Identity Verify করতে সাহায্য করে।

একটি Certificate-এ সাধারণত থাকে:

- Domain Name
- Subject / Organization Information
- Public Key
- Issuer (Certificate Authority)
- Valid From
- Valid To
- Signature Algorithm
- Digital Signature

### Certificate Example

```text
Domain: example.com

Issuer: Let's Encrypt

Subject: example.com

Public Key: RSA / ECC

Valid From: 01-01-2026

Valid To: 01-04-2026

Signature: CA Digital Signature
```

### Certificate Authority (CA) কী?

Certificate Authority হলো Trusted Organization, যারা Digital Certificate Issue করে।

Examples:

- Let's Encrypt
- DigiCert
- GlobalSign
- Sectigo

### Certificate Chain

```text
Root CA
   |
   v
Intermediate CA
   |
   v
Website Certificate
   |
   v
example.com
```

Browser Trusted Root CA Store ব্যবহার করে Certificate Chain Validate করে।

---

# 8. Public Key ও Private Key

TLS-এ Cryptographic Key গুরুত্বপূর্ণ ভূমিকা রাখে।

## Public Key

- Publicly Share করা যায়।
- Certificate-এর মধ্যে থাকে।
- Cryptographic Operations-এর জন্য ব্যবহৃত হয়।

## Private Key

- Secret রাখতে হয়।
- Server-এর নিয়ন্ত্রণে থাকে।
- Unauthorized ব্যক্তির কাছে গেলে Certificate-এর Security Compromised হতে পারে।

Example:

```text
             SERVER

       +----------------+
       | Public Key     |
       | Certificate    |
       +----------------+

       +----------------+
       | Private Key    |
       | SECRET         |
       +----------------+
```

**Important:** TLS 1.3-এর সাধারণ Handshake-এ Public/Private Key সরাসরি Session Data Encrypt করার জন্য ব্যবহৃত হয় না। এখানে Ephemeral Key Exchange দিয়ে Shared Secret তৈরি হয় এবং সেই Secret থেকে Symmetric Encryption Key Derive করা হয়।

---

# 9. Symmetric vs Asymmetric Encryption

| বিষয় | Symmetric | Asymmetric |
|---|---|---|
| Keys | Same Secret Key | Public + Private Key |
| Speed | Fast | তুলনামূলক Slow |
| Use Case | Bulk Data Encryption | Key Exchange / Authentication |
| Example | AES-GCM | RSA, ECDSA, X25519 |

### Symmetric Encryption

```text
Original Data
     |
     v
Secret Key
     |
     v
Encrypted Data
     |
     v
Secret Key
     |
     v
Original Data
```

### Asymmetric Cryptography

```text
Public Key + Private Key
          |
          v
  Cryptographic Operations
```

TLS সাধারণত Handshake-এর মাধ্যমে Shared Secret Establish করে এবং পরবর্তী Data Encryption-এর জন্য Symmetric AEAD Cipher ব্যবহার করে।

---

# 10. HTTPS Setup – Local Development

ধরো তুমি ASP.NET Core API এবং Angular 20 ব্যবহার করছ।

প্রথমে Local Development Certificate Trust করতে হবে।

### Step 1: Check .NET SDK

Visual Studio 2022 অথবা Terminal-এ:

```bash
dotnet --version
```

### Step 2: Check HTTPS Development Certificate

```bash
dotnet dev-certs https --check
```

### Step 3: Trust Development Certificate

Windows:

```bash
dotnet dev-certs https --trust
```

### Step 4: Run ASP.NET Core API

```bash
dotnet run
```

Example:

```text
https://localhost:7100
http://localhost:5100
```

### Step 5: Browser Test

```text
https://localhost:7100
```

Browser Trusted Development Certificate ব্যবহার করে Secure Connection Establish করবে।

> Local Development Certificate Production-এ ব্যবহার করা যাবে না।

---

# 11. ASP.NET Core-এ HTTPS Setup

এখন ASP.NET Core Web API-তে HTTPS Configure করব।

### Step 1: Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

builder.Services.AddHttpsRedirection(options =>
{
    options.HttpsPort = 7100;
});

var app = builder.Build();

if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}

app.UseHttpsRedirection();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();
```

### Explanation

`AddHttpsRedirection()`:

HTTP Request-কে HTTPS-এ Redirect করার Configuration।

`UseHttpsRedirection()`:

HTTP Request এলে HTTPS URL-এ Redirect করে।

`UseHsts()`:

Browser-কে নির্দিষ্ট সময় পর্যন্ত HTTPS ব্যবহার করার Policy জানায়।

### Step 2: launchSettings.json

```json
{
  "profiles": {
    "Madrasa.Api": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "https://localhost:7100;http://localhost:5100",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

### Step 3: Test API

```http
GET https://localhost:7100/api/notices
```

এখন HTTPS ব্যবহার করে API Request পাঠাতে পারবে।

### Important Production Note

Production-এ যদি Nginx বা Load Balancer TLS Terminate করে, তাহলে ASP.NET Core-এর সামনে Trusted Forwarded Headers Configuration করতে হবে।

শুধু `UseHttpsRedirection()` ব্যবহার করলেই Reverse Proxy Environment-এ সবসময় সঠিক Redirect হবে না।

---

# 12. Angular 20 + .NET API HTTPS

ধরো তোমার Application:

```text
Frontend: Angular 20
Backend: ASP.NET Core 8
Database: SQL Server
```

Production Domain:

```text
https://www.mydonation.com
https://api.mydonation.com
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

### API Service

```typescript
@Injectable({
  providedIn: 'root'
})
export class AuthService {

  private apiUrl = environment.apiUrl;

  constructor(private http: HttpClient) {}

  login(data: LoginRequest) {

    return this.http.post(
      `${this.apiUrl}/auth/login`,
      data
    );
  }
}
```

### Angular Production Build

```bash
ng build --configuration production
```

Build Output:

```text
dist/
  madrasa-web/
```

এই Build Files Nginx বা অন্য Web Server-এ Deploy করবে।

### CORS Configuration in .NET

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AngularPolicy", policy =>
    {
        policy
            .WithOrigins("https://www.mydonation.com")
            .AllowAnyHeader()
            .AllowAnyMethod();
    });
});
```

Middleware:

```csharp
app.UseCors("AngularPolicy");
```

**Important:**

- CORS এবং HTTPS আলাদা বিষয়।
- CORS HTTPS-এর বিকল্প নয়।
- Production-এ `AllowAnyOrigin()` Avoid করবে।
- Cookie Authentication ব্যবহার করলে `AllowCredentials()` এবং Explicit Origin প্রয়োজন।
- JWT Bearer Authentication-এর ক্ষেত্রে HTTPS বাধ্যতামূলক রাখবে।

---

# 13. Production Server-এ HTTPS Setup

Production Environment-এ সাধারণ Architecture:

```text
                  INTERNET
                     |
                     |
                HTTPS :443
                     |
                     v
           +------------------+
           |  Nginx / IIS     |
           |  TLS Termination |
           +--------+---------+
                    |
                    | HTTP (Private Network)
                    |
                    v
           +------------------+
           | ASP.NET Core API|
           | Port 5000       |
           +--------+---------+
                    |
                    v
              SQL SERVER
```

### Production Deployment Steps

1. Domain Name Purchase করবে।
2. Domain DNS Record Configure করবে।
3. Server-এ Nginx বা IIS Install করবে।
4. TLS Certificate Issue করবে।
5. Certificate এবং Private Key Configure করবে।
6. HTTPS Port 443 Open করবে।
7. HTTP Port 80 থেকে HTTPS Redirect করবে।
8. Backend API Secure Network-এ রাখবে।
9. HSTS Configure করবে।
10. Certificate Auto Renewal Configure করবে।
11. HTTPS Connection Test করবে।

### DNS Example

```text
A Record:

mydonation.com       -> SERVER_PUBLIC_IP
www                  -> SERVER_PUBLIC_IP
api                  -> SERVER_PUBLIC_IP
```

---

# 14. Let's Encrypt Certificate

Let's Encrypt হলো একটি Free Certificate Authority, যা Domain Validation-এর মাধ্যমে TLS Certificate প্রদান করে।

Certificate সাধারণত 90 দিনের জন্য Valid থাকে।

### Ubuntu + Nginx Example

Step 1: Install Nginx

```bash
sudo apt update

sudo apt install nginx -y
```

Step 2: Install Certbot

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Step 3: DNS Configure

```text
mydonation.com -> SERVER_PUBLIC_IP
```

Step 4: Issue Certificate

```bash
sudo certbot --nginx -d mydonation.com -d www.mydonation.com
```

API Subdomain-এর জন্য:

```bash
sudo certbot --nginx -d api.mydonation.com
```

Step 5: Test Certificate Renewal

```bash
sudo certbot renew --dry-run
```

Step 6: Check Certificate

```bash
sudo certbot certificates
```

**Important:** Domain DNS সঠিকভাবে Configure থাকতে হবে এবং Certificate Validation-এর জন্য প্রয়োজনীয় Port/Challenge Access থাকতে হবে।

---

# 15. Nginx Reverse Proxy Configuration

ধরো:

```text
Frontend: Angular
Backend: ASP.NET Core API

Domain: mydonation.com
API: api.mydonation.com

Backend Internal Port: 5000
```

### Nginx Configuration

File:

```bash
sudo nano /etc/nginx/sites-available/mydonation
```

Configuration:

```nginx
server {
    listen 80;
    server_name mydonation.com www.mydonation.com;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name mydonation.com www.mydonation.com;

    root /var/www/madrasa-web;
    index index.html;

    ssl_certificate /etc/letsencrypt/live/mydonation.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/mydonation.com/privkey.pem;

    location / {
        try_files $uri $uri/ /index.html;
    }
}

server {
    listen 443 ssl;
    server_name api.mydonation.com;

    ssl_certificate /etc/letsencrypt/live/api.mydonation.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.mydonation.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:5000;

        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;

        proxy_set_header X-Forwarded-For
            $proxy_add_x_forwarded_for;

        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Enable Configuration

```bash
sudo ln -s /etc/nginx/sites-available/mydonation \
/etc/nginx/sites-enabled/
```

Test:

```bash
sudo nginx -t
```

Reload:

```bash
sudo systemctl reload nginx
```

এখন:

```text
https://mydonation.com
https://api.mydonation.com
```

দুটি Domain HTTPS ব্যবহার করবে।

---

# 16. IIS-এ HTTPS Configuration

Windows Server-এ ASP.NET Core Application Host করলে IIS ব্যবহার করা যায়।

### Steps

1. Windows Server-এ IIS Install করবে।
2. Domain-এর জন্য Certificate সংগ্রহ করবে।
3. IIS Manager Open করবে।
4. Server Certificates-এ Certificate Import করবে।
5. Website-এর Bindings Open করবে।
6. Add Binding নির্বাচন করবে।
7. Type: HTTPS
8. Port: 443
9. Host Name: api.mydonation.com
10. সঠিক SSL Certificate Select করবে।
11. Apply করবে।

Example:

```text
Website: Madrasa API

Binding:
Type: HTTPS
Port: 443
Host: api.mydonation.com
Certificate: api.mydonation.com
```

এখন IIS-এর মাধ্যমে ASP.NET Core API HTTPS-এ Access করা যাবে।

---

# 17. HTTP to HTTPS Redirect

যদি User HTTP দিয়ে Website Access করে:

```text
http://mydonation.com
```

তাহলে তাকে Redirect করতে হবে:

```text
https://mydonation.com
```

Nginx:

```nginx
server {
    listen 80;

    server_name mydonation.com;

    return 301 https://$host$request_uri;
}
```

ASP.NET Core:

```csharp
app.UseHttpsRedirection();
```

**301 Redirect:** Permanent Redirect.

Production-এ Redirect সঠিকভাবে Configure করতে হবে, যাতে Redirect Loop তৈরি না হয়।

---

# 18. HSTS কী?

HSTS = HTTP Strict Transport Security

HSTS হলো একটি HTTP Security Response Header, যা Browser-কে নির্দিষ্ট সময় পর্যন্ত Website-এর সঙ্গে শুধু HTTPS ব্যবহার করতে নির্দেশ দেয়।

Example:

```http
Strict-Transport-Security:
max-age=31536000; includeSubDomains
```

### HSTS Parameters

| Parameter | Meaning |
|---|---|
| max-age | কত সেকেন্ড Policy কার্যকর থাকবে |
| includeSubDomains | Subdomain-গুলোর জন্যও Policy |
| preload | Browser Preload List-এর জন্য অনুরোধ |

ASP.NET Core:

```csharp
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}
```

### HSTS কেন প্রয়োজন?

ধরো, User ভুল করে HTTP Website Open করল।

HSTS আগে থেকে Browser-এ Stored থাকলে Browser নিজেই HTTPS ব্যবহার করবে।

**Warning:** `includeSubDomains` এবং `preload` ব্যবহার করার আগে নিশ্চিত হবে সব সংশ্লিষ্ট Subdomain HTTPS Support করে। ভুল Configuration দীর্ঘস্থায়ী Access Problem তৈরি করতে পারে।

---

# 19. Professional Security Best Practices

Production Application-এ নিচের বিষয়গুলো Follow করা উচিত।

### 19.1 Modern TLS Versions

- TLS 1.2 এবং TLS 1.3 Enable রাখবে।
- TLS 1.0 এবং TLS 1.1 Disable করবে।
- SSL Protocol Disable করবে।

### 19.2 Secure Certificate Management

- Trusted CA ব্যবহার করবে।
- Certificate Expiry Monitor করবে।
- Automatic Renewal Configure করবে।
- Private Key কখনো GitHub-এ Upload করবে না।

### 19.3 Strong Cryptography

- Modern AEAD Cipher ব্যবহার করবে।
- TLS 1.3 Prefer করবে।
- Weak Cipher Suite Disable করবে।
- Secure Key Exchange ব্যবহার করবে।

### 19.4 Secure Cookies

```http
Set-Cookie: sessionId=abc123;
Secure;
HttpOnly;
SameSite=Lax
```

- `Secure`: HTTPS ছাড়া Cookie পাঠাবে না।
- `HttpOnly`: JavaScript দিয়ে Cookie Access সীমিত করে।
- `SameSite`: Cross-Site Request Risk কমাতে সাহায্য করে।

### 19.5 HSTS Enable

Production-এ HSTS Configure করবে।

### 19.6 API Security

- JWT Token HTTPS-এর মাধ্যমে পাঠাবে।
- Access Token এবং Refresh Token Securely Handle করবে।
- Token URL Query String-এ পাঠাবে না।
- Rate Limiting ব্যবহার করবে।
- Authentication ও Authorization আলাদাভাবে Implement করবে।

### 19.7 Database Security

HTTPS থাকলেই Application থেকে Database পর্যন্ত Connection Secure হয় না।

SQL Server Connection-এর জন্য আলাদাভাবে TLS Configure করবে।

Example Connection String:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SERVER;Database=MadrasaDb;User Id=app_user;Password=YOUR_SECRET;Encrypt=True;TrustServerCertificate=False;"
  }
}
```

Production-এ SQL Server-এর Valid Certificate এবং সঠিক Hostname Validation নিশ্চিত করবে।

### 19.8 Security Headers

প্রয়োজন অনুযায়ী Configure করবে:

```http
Strict-Transport-Security
X-Content-Type-Options
Content-Security-Policy
Referrer-Policy
```

Security Header Application-এর প্রয়োজন অনুযায়ী Configure করবে। Blindly সব Header Enable করবে না।

---

# 20. Common HTTPS Errors

| Error | কারণ | Solution |
|---|---|---|
| NET::ERR_CERT_AUTHORITY_INVALID | Untrusted Certificate | Trusted Certificate ব্যবহার |
| NET::ERR_CERT_DATE_INVALID | Certificate Expired | Renew Certificate |
| NET::ERR_CERT_COMMON_NAME_INVALID | Domain Mismatch | Correct Domain Certificate |
| SSL_ERROR_RX_RECORD_TOO_LONG | HTTPS Port-এ ভুল Protocol/Configuration | Server Binding Check |
| ERR_SSL_VERSION_OR_CIPHER_MISMATCH | TLS/Cipher Incompatibility | TLS Configuration Check |
| Mixed Content | HTTPS Page থেকে HTTP Resource | Resource HTTPS করা |
| Too Many Redirects | Redirect Loop | Proxy এবং HTTPS Middleware Check |
| CORS Error | Cross-Origin Policy সমস্যা | Correct CORS Origin Configure |

### Mixed Content Example

Wrong:

```typescript
const imageUrl = 'http://example.com/image.png';
```

Correct:

```typescript
const imageUrl = 'https://example.com/image.png';
```

HTTPS Website-এর মধ্যে HTTP Resource ব্যবহার করলে Browser সেটি Block করতে পারে।

---

# 21. Useful Commands

### .NET HTTPS Commands

```bash
dotnet dev-certs https --check
```

```bash
dotnet dev-certs https --trust
```

```bash
dotnet dev-certs https --clean
```

```bash
dotnet dev-certs https --export-path certificate.pfx -p YourStrongPassword
```

> PFX Export করলে সেটি Secret হিসেবে রাখবে। Repository-তে Upload করবে না।

### OpenSSL Commands

Certificate Information:

```bash
openssl x509 -in certificate.crt -text -noout
```

Certificate Expiry:

```bash
openssl x509 -in certificate.crt -noout -dates
```

Test TLS Connection:

```bash
openssl s_client -connect example.com:443 -servername example.com
```

Check TLS 1.3:

```bash
openssl s_client -connect example.com:443 -tls1_3
```

Check TLS 1.2:

```bash
openssl s_client -connect example.com:443 -tls1_2
```

### Nginx Commands

```bash
sudo nginx -t
```

```bash
sudo systemctl status nginx
```

```bash
sudo systemctl reload nginx
```

### Check HTTPS Headers

```bash
curl -I https://example.com
```

---

# 22. Interview Questions & Answers

### Q1. What is HTTPS?

**Answer:** HTTPS হলো HTTP-এর Secure Version, যা TLS ব্যবহার করে Client এবং Server-এর মধ্যে Encrypted Communication নিশ্চিত করে।

### Q2. What is TLS?

**Answer:** TLS হলো একটি Cryptographic Protocol, যা Network Communication-এর Confidentiality, Integrity এবং Authentication নিশ্চিত করে।

### Q3. What is the difference between SSL and TLS?

**Answer:** SSL হলো পুরোনো এবং Deprecated Protocol। TLS হলো SSL-এর Successor এবং বর্তমানে Secure Communication-এর জন্য ব্যবহৃত হয়।

### Q4. What is the default HTTPS port?

**Answer:** 443.

### Q5. What is a Digital Certificate?

**Answer:** Digital Certificate হলো CA-Signed Electronic Document, যা Server-এর Identity এবং Public Key Verify করতে সাহায্য করে।

### Q6. What is Certificate Authority?

**Answer:** CA হলো Trusted Organization, যারা Digital Certificate Issue এবং Sign করে।

### Q7. What is TLS Handshake?

**Answer:** TLS Handshake হলো Client ও Server-এর মধ্যে Secure Connection Establish করার Process।

### Q8. What is the difference between Symmetric and Asymmetric Encryption?

**Answer:** Symmetric Encryption-এ একই Secret Key ব্যবহার করা হয়। Asymmetric Cryptography-তে Public এবং Private Key Pair ব্যবহার করা হয়।

### Q9. What is HSTS?

**Answer:** HSTS হলো একটি Security Policy, যা Browser-কে নির্দিষ্ট সময়ের জন্য HTTPS ব্যবহার করতে নির্দেশ দেয়।

### Q10. What is Let's Encrypt?

**Answer:** Let's Encrypt হলো Free, Automated এবং Open Certificate Authority, যা TLS Certificate প্রদান করে।

### Q11. Does HTTPS prevent SQL Injection?

**Answer:** না। HTTPS Data Transmission Secure করে। SQL Injection প্রতিরোধ করতে Parameterized Queries, Stored Procedures এবং Input Validation প্রয়োজন।

### Q12. Does HTTPS encrypt Database Connections?

**Answer:** HTTPS Browser-to-Server Communication Secure করে। Database Connection Secure করতে আলাদাভাবে Database TLS Configuration প্রয়োজন।

### Q13. What is TLS Termination?

**Answer:** TLS Termination হলো Reverse Proxy, Load Balancer বা Web Server-এ TLS Connection Decrypt করার Process।

### Q14. What is a Self-Signed Certificate?

**Answer:** Self-Signed Certificate হলো এমন Certificate, যা কোনো Publicly Trusted CA দ্বারা Signed নয়; এটি নিজের Key দিয়েই Sign করা হয়। Development ও Internal Testing-এ ব্যবহার করা যায়।

### Q15. Why should we never upload a Private Key to GitHub?

**Answer:** Private Key Public হয়ে গেলে Attacker সেটি ব্যবহার করে Server Identity Impersonate করতে পারে। তাই Private Key Secret Manager বা Protected Server Storage-এ রাখতে হবে।

---

# 🎯 Final Checklist – Professional Developer

Production Website Deploy করার আগে:

- [ ] HTTPS Enabled
- [ ] Valid TLS Certificate Installed
- [ ] TLS 1.2 / TLS 1.3 Supported
- [ ] SSL / TLS 1.0 / TLS 1.1 Disabled
- [ ] HTTP to HTTPS Redirect Configured
- [ ] HSTS Configured
- [ ] Certificate Auto Renewal Enabled
- [ ] Private Key Securely Stored
- [ ] Angular API URL Uses HTTPS
- [ ] CORS Properly Configured
- [ ] Secure Cookies Configured
- [ ] SQL Server Connection Encryption Enabled
- [ ] No Mixed Content
- [ ] Security Headers Configured
- [ ] Certificate Expiry Monitoring Enabled
- [ ] Production HTTPS Tested

---

**Author:** Full Stack .NET Developer  
**Topics:** HTTPS | TLS | SSL | Cryptography | ASP.NET Core | Angular | Nginx | IIS | DevOps | Web Security

⬅️ [CI/CD](13-ci-cd.md) | ➡️ [JWT ও OAuth 2.0](15-jwt-and-oauth2.md)
