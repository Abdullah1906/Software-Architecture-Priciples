# Chapter ১৬: 🔐 SQL Injection & XSS Protection

> **Web Application Security | ASP.NET Core | C# | ADO.NET | SQL Server | Angular**
>
> এই গাইডে SQL Injection ও Cross-Site Scripting (XSS) কী, কেন হয়, কীভাবে Attack করা হয় এবং Professional Grade Application-এ কীভাবে প্রতিরোধ করতে হয় তা সহজ উদাহরণসহ আলোচনা করা হয়েছে।

---

# 📚 Table of Contents

1. [Introduction to Web Security](#1-introduction-to-web-security)
2. [SQL Injection কী?](#2-sql-injection-কি)
3. [SQL Injection কেন হয়?](#3-sql-injection-কেন-হয়)
4. [SQL Injection Attack Example](#4-sql-injection-attack-example)
5. [SQL Injection Protection](#5-sql-injection-protection)
6. [ADO.NET Parameterized Query](#6-adonet-parameterized-query)
7. [Stored Procedure Security](#7-stored-procedure-security)
8. [XSS কী?](#8-xss-cross-site-scripting-কি)
9. [XSS Attack Example](#9-xss-attack-example)
10. [XSS Protection](#10-xss-protection)
11. [Angular XSS Protection](#11-angular-xss-protection)
12. [ASP.NET Core Security Headers](#12-aspnet-core-security-headers)
13. [Professional Security Best Practices](#13-professional-security-best-practices)
14. [Final Security Checklist](#14-final-security-checklist)

---

# 1. Introduction to Web Security

Web Application তৈরি করার সময় শুধু functionality implement করলেই হয় না। Application-এর data, user information এবং server নিরাপদ রাখাও গুরুত্বপূর্ণ।

দুটি Common Web Security Vulnerability:

| Vulnerability | Full Form | Main Target |
|---|---|---|
| SQL Injection | SQL Injection | Database |
| XSS | Cross-Site Scripting | User's Browser |

**সহজভাবে:**

- SQL Injection: Attacker Database-এর SQL Query manipulate করার চেষ্টা করে।
- XSS: Attacker অন্য ব্যবহারকারীর Browser-এ malicious JavaScript execute করানোর চেষ্টা করে।

---

# 2. SQL Injection কী?

SQL Injection হলো একটি Database Security Vulnerability।

যখন Application User Input সরাসরি SQL Query-এর মধ্যে যুক্ত করে, তখন Attacker বিশেষভাবে তৈরি Input পাঠিয়ে SQL Query-এর original structure পরিবর্তন করতে পারে।

এর মাধ্যমে Attacker চেষ্টা করতে পারে:

- Unauthorized Login
- Sensitive Data Access
- Database Records Read
- Database Records Modify
- Database Records Delete

### Example

ধরো, তোমার Application-এ Login System আছে।

```sql
SELECT * FROM Users
WHERE Email = 'admin@gmail.com'
AND Password = '123456';
```

এখানে User-এর Email ও Password দিয়ে Database Query তৈরি হচ্ছে।

যদি User Input সরাসরি SQL Query-এর সঙ্গে concatenate করা হয়, তাহলে SQL Injection Vulnerability তৈরি হতে পারে।

---

# 3. SQL Injection কেন হয়?

সবচেয়ে Common কারণ:

**Unsafe SQL Query Construction**

### ❌ Vulnerable C# Code

```csharp
string email = txtEmail.Text;
string password = txtPassword.Text;

string query = "SELECT * FROM Users WHERE Email = '"
               + email
               + "' AND Password = '"
               + password + "'";

SqlCommand cmd = new SqlCommand(query, connection);
```

### সমস্যা কোথায়?

এখানে User Input সরাসরি SQL Query-এর সঙ্গে যুক্ত করা হয়েছে।

Attacker এমন Input পাঠিয়ে Query-এর Logic পরিবর্তন করার চেষ্টা করতে পারে।

**মূল সমস্যা:** Application User Input-কে Data হিসেবে না দেখে SQL Command-এর অংশ হিসেবে গ্রহণ করছে।

---

# 4. SQL Injection Attack Example

### Example: Authentication Bypass

ধরো, Application-এর Login Query:

```sql
SELECT * FROM Users
WHERE Email = 'USER_INPUT'
AND Password = 'PASSWORD_INPUT';
```

Attacker Email Field-এ নিচের Input দিতে পারে:

```sql
' OR '1'='1' --
```

ফলে Vulnerable Query-এর Structure এমন হতে পারে:

```sql
SELECT * FROM Users
WHERE Email = '' OR '1'='1' --'
AND Password = 'anything';
```

এখানে:

- `'1'='1'` সবসময় TRUE।
- `--` SQL Server-এ পরবর্তী অংশকে Comment হিসেবে গণ্য করে।

ফলে Vulnerable Authentication Logic bypass হওয়ার ঝুঁকি তৈরি হয়।

> এটি শুধু একটি শিক্ষামূলক উদাহরণ। বাস্তব Authentication System-এ Password কখনো Plain Text হিসেবে Database-এ সংরক্ষণ করা উচিত নয়।

---

# 5. SQL Injection Protection

SQL Injection প্রতিরোধের সবচেয়ে গুরুত্বপূর্ণ উপায়:

1. Parameterized Query ব্যবহার করা।
2. Stored Procedure ব্যবহার করা।
3. Dynamic SQL এড়িয়ে চলা।
4. Database Least Privilege অনুসরণ করা।
5. Input Validation করা।
6. ORM ব্যবহার করলে Parameterized Operations নিশ্চিত করা।

**মনে রাখবে:** শুধু Input Validation SQL Injection Protection-এর বিকল্প নয়।

---

# 6. ADO.NET Parameterized Query

Parameterized Query হলো SQL Injection Protection-এর সবচেয়ে গুরুত্বপূর্ণ Technique।

এখানে SQL Query এবং User Input আলাদা রাখা হয়।

### ❌ Unsafe Query

```csharp
string query =
    $"SELECT * FROM Users WHERE Email = '{email}'";
```

### ✅ Secure Query

```csharp
using Microsoft.Data.SqlClient;
using System.Data;

public async Task<User?> GetUserByEmailAsync(string email)
{
    using var connection = new SqlConnection(connectionString);

    using var command = new SqlCommand(
        "SELECT Id, Email FROM Users WHERE Email = @Email",
        connection);

    command.Parameters.Add(
        "@Email",
        SqlDbType.NVarChar,
        256
    ).Value = email;

    await connection.OpenAsync();

    using var reader = await command.ExecuteReaderAsync();

    if (await reader.ReadAsync())
    {
        return new User
        {
            Id = reader.GetInt32(0),
            Email = reader.GetString(1)
        };
    }

    return null;
}
```

### এখানে কী হচ্ছে?

```sql
SELECT Id, Email
FROM Users
WHERE Email = @Email;
```

`@Email` একটি SQL Parameter।

```csharp
command.Parameters.Add(
    "@Email",
    SqlDbType.NVarChar,
    256
).Value = email;
```

এখানে User Input SQL Command-এর অংশ না হয়ে Parameter হিসেবে পাঠানো হচ্ছে।

ফলে User Input দিয়ে SQL Query-এর Structure পরিবর্তন করা যায় না।

### Professional Recommendation

- Explicit SQL Data Type ব্যবহার করো।
- String Length নির্দিষ্ট করো।
- `AddWithValue()`-এর পরিবর্তে উপযুক্ত Explicit Parameter Type ব্যবহার করো।
- `using` দিয়ে Connection ও Command Dispose করো।
- Async Database Operation ব্যবহার করো।

---

# 7. Stored Procedure Security

Professional .NET Application-এ Stored Procedure ব্যবহার করা খুবই প্রচলিত।

তবে একটি গুরুত্বপূর্ণ বিষয়:

**Stored Procedure ব্যবহার করলেই SQL Injection থেকে সম্পূর্ণ নিরাপদ হওয়া যায় না।**

Stored Procedure-এর ভিতরেও Unsafe Dynamic SQL থাকলে Vulnerability তৈরি হতে পারে।

### Step 1: Secure Stored Procedure

```sql
CREATE PROCEDURE SP_GetUserByEmail
    @Email NVARCHAR(256)
AS
BEGIN
    SET NOCOUNT ON;

    SELECT Id, Email
    FROM Users
    WHERE Email = @Email;
END;
```

এখানে `@Email` একটি Parameter।

### Step 2: C# ADO.NET Implementation

```csharp
public async Task<User?> GetUserByEmailAsync(string email)
{
    using var connection = new SqlConnection(connectionString);

    using var command = new SqlCommand(
        "SP_GetUserByEmail",
        connection);

    command.CommandType = CommandType.StoredProcedure;

    command.Parameters.Add(
        "@Email",
        SqlDbType.NVarChar,
        256
    ).Value = email;

    await connection.OpenAsync();

    using var reader = await command.ExecuteReaderAsync();

    if (await reader.ReadAsync())
    {
        return new User
        {
            Id = reader.GetInt32(0),
            Email = reader.GetString(1)
        };
    }

    return null;
}
```

### Step 3: Unsafe Dynamic SQL এড়িয়ে চলো

❌ Vulnerable Stored Procedure:

```sql
CREATE PROCEDURE SP_SearchUser
    @Email NVARCHAR(256)
AS
BEGIN
    DECLARE @SQL NVARCHAR(MAX);

    SET @SQL =
        'SELECT * FROM Users WHERE Email = '''
        + @Email + '''';

    EXEC(@SQL);
END;
```

এখানে User Input সরাসরি SQL String-এর সঙ্গে যুক্ত হয়েছে।

### Secure Dynamic SQL

যদি Dynamic SQL প্রয়োজন হয়, তাহলে `sp_executesql` এবং Parameter ব্যবহার করো।

```sql
CREATE PROCEDURE SP_SearchUser
    @Email NVARCHAR(256)
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @SQL NVARCHAR(MAX);

    SET @SQL = N'
        SELECT Id, Email
        FROM Users
        WHERE Email = @Email';

    EXEC sys.sp_executesql
        @SQL,
        N'@Email NVARCHAR(256)',
        @Email = @Email;
END;
```

**Important:** Dynamic Column Name বা Sort Direction Parameter হিসেবে পাঠানো যায় না। সেক্ষেত্রে Allowlist ব্যবহার করতে হবে।

### SQL Injection Protection Flow

```text
Angular Application
        |
        v
ASP.NET Core Web API
        |
        v
Input Validation
        |
        v
Application Service
        |
        v
ADO.NET Repository
        |
        v
Parameterized Stored Procedure
        |
        v
SQL Server Database
```

---

# 8. XSS (Cross-Site Scripting) কী?

XSS হলো একটি Client-Side Web Security Vulnerability।

যখন Application User-এর দেওয়া Untrusted Content নিরাপদভাবে Encode বা Sanitize না করে Browser-এ Render করে, তখন Attacker-এর JavaScript Code Execute হওয়ার সুযোগ তৈরি হতে পারে।

এর মাধ্যমে Attacker চেষ্টা করতে পারে:

- User-এর Browser-এ Malicious JavaScript চালানো।
- Fake Login Form দেখানো।
- User-এর পক্ষ থেকে Unauthorized Action করানো।
- Sensitive Page Content Access করা।
- Session-related Attack করা।

### XSS-এর প্রধান ৩টি ধরন

| Type | Description |
|---|---|
| Stored XSS | Malicious Content Database-এ Save থাকে |
| Reflected XSS | Request-এর Input Response-এ Unsafeভাবে ফিরে আসে |
| DOM-Based XSS | Client-Side JavaScript DOM Manipulation-এর মাধ্যমে Vulnerability তৈরি করে |

---

# 9. XSS Attack Example

ধরো, তোমার Application-এ User Profile-এর Name দেখানো হয়।

User Profile Name হিসেবে এমন Content Save করল:

```html
<script>
    alert('XSS Attack');
</script>
```

যদি Application এটি সরাসরি HTML হিসেবে Render করে, তাহলে Browser JavaScript Execute করতে পারে।

### ❌ Vulnerable JavaScript

```javascript
const userInput = "<img src=x onerror=alert('XSS')>";

document.getElementById("profile").innerHTML = userInput;
```

এখানে `innerHTML` User Input-কে HTML হিসেবে Interpret করছে।

ফলে XSS Vulnerability তৈরি হতে পারে।

---

# 10. XSS Protection

XSS প্রতিরোধের গুরুত্বপূর্ণ Techniques:

1. Output Encoding।
2. HTML Sanitization।
3. Safe DOM API ব্যবহার।
4. Content Security Policy (CSP)।
5. Secure Cookie Configuration।
6. Untrusted HTML সরাসরি Render না করা।

### 10.1 Output Encoding

Untrusted Input-কে HTML-এর Special Character হিসেবে Encode করতে হবে।

উদাহরণ:

| Original | Encoded |
|---|---|
| `<` | `&lt;` |
| `>` | `&gt;` |
| `&` | `&amp;` |
| `"` | `&quot;` |
| `'` | `&#x27;` |

### 10.2 Safe JavaScript DOM Manipulation

❌ Unsafe:

```javascript
element.innerHTML = userInput;
```

✅ Safe:

```javascript
element.textContent = userInput;
```

`textContent` User Input-কে Text হিসেবে Render করে, HTML হিসেবে Execute করে না।

### 10.3 HTML Sanitization

যদি Application-এ Rich Text Editor প্রয়োজন হয়, যেমন:

- Blog Content
- Article Description
- User Comments
- Rich Text Messages

তাহলে Trusted HTML Sanitization Library ব্যবহার করতে হবে।

Example: DOMPurify।

```javascript
const cleanHTML = DOMPurify.sanitize(userInput);

document.getElementById("content").innerHTML = cleanHTML;
```

**মনে রাখবে:** HTML Sanitization এবং Output Encoding এক জিনিস নয়। Context অনুযায়ী সঠিক Protection ব্যবহার করতে হবে।

---

# 11. Angular XSS Protection

Angular-এর Template System সাধারণত Interpolation ও Property Binding-এর মাধ্যমে Automatically Context-Aware Escaping করে।

### 11.1 Angular Interpolation

✅ Safe:

```typescript
export class ProfileComponent {
    username = "<script>alert('XSS')</script>";
}
```

```html
<h2>{{ username }}</h2>
```

Angular এখানে User Input-কে Text হিসেবে Render করবে।

JavaScript Execute করবে না।

### 11.2 Property Binding

```html
<div [title]="username">
    User Profile
</div>
```

Angular এখানে নিরাপদ Property Binding ব্যবহার করে।

### 11.3 Avoid Direct innerHTML

❌ Avoid:

```typescript
element.nativeElement.innerHTML = userInput;
```

### 11.4 Angular innerHTML Binding

Angular-এর Built-in Sanitizer HTML Context-এ কিছু Unsafe Content Remove বা Sanitize করে।

```html
<div [innerHTML]="userContent"></div>
```

তবুও Untrusted HTML-এর জন্য প্রয়োজন অনুযায়ী Trusted Sanitization Policy অনুসরণ করতে হবে।

### 11.5 Never Misuse bypassSecurityTrustHtml()

❌ Dangerous:

```typescript
this.sanitizer.bypassSecurityTrustHtml(userInput);
```

এটি Angular-এর Built-in Sanitization Bypass করে।

শুধু তখনই ব্যবহার করা উচিত যখন HTML-এর Trustworthiness স্বাধীনভাবে নিশ্চিত করা হয়েছে এবং Security Review সম্পন্ন হয়েছে।

---

# 12. ASP.NET Core Security Headers

Professional Grade Web Application-এ Security Headers ব্যবহার করা গুরুত্বপূর্ণ।

### 12.1 Content Security Policy (CSP)

CSP Browser-কে নির্দেশ দেয় কোন Source থেকে Script, Style ও অন্যান্য Resource Load করা যাবে।

`Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.Use(async (context, next) =>
{
    context.Response.Headers.Append(
        "Content-Security-Policy",
        "default-src 'self'; " +
        "script-src 'self'; " +
        "object-src 'none'; " +
        "base-uri 'self'; " +
        "frame-ancestors 'none';"
    );

    context.Response.Headers.Append(
        "X-Content-Type-Options",
        "nosniff"
    );

    context.Response.Headers.Append(
        "Referrer-Policy",
        "strict-origin-when-cross-origin"
    );

    await next();
});

app.MapControllers();

app.Run();
```

> CSP হলো Defense-in-Depth Security Layer। এটি Secure Coding-এর বিকল্প নয়। Production-এ Angular Build, CDN এবং প্রয়োজনীয় Resource অনুযায়ী Policy Test করে Configure করতে হবে। প্রথমে `Content-Security-Policy-Report-Only` দিয়ে পরীক্ষা করা ভালো।

### 12.2 Secure Cookie

যদি Authentication Cookie ব্যবহার করো:

```csharp
builder.Services.AddAuthentication()
    .AddCookie(options =>
    {
        options.Cookie.HttpOnly = true;
        options.Cookie.SecurePolicy =
            CookieSecurePolicy.Always;

        options.Cookie.SameSite =
            SameSiteMode.Lax;
    });
```

- `HttpOnly`: JavaScript থেকে Cookie Access বন্ধ করে।
- `Secure`: HTTPS ছাড়া Cookie পাঠানো বন্ধ করে।
- `SameSite`: Cross-Site Cookie Sending সীমিত করে।

> Cookie-based Authentication ব্যবহার করলে CSRF Protection-ও implement করতে হবে। JWT localStorage-এ রাখলেই XSS থেকে নিরাপদ হওয়া যায় না।

---

# 13. Professional Security Best Practices

## SQL Injection Prevention

| Best Practice | Purpose |
|---|---|
| Parameterized Query | SQL Injection প্রতিরোধ |
| Stored Procedure | Database Logic Centralize করা |
| sp_executesql | Secure Dynamic SQL |
| Input Validation | Invalid Data Reject করা |
| Least Privilege | Database Access সীমিত রাখা |
| Error Handling | Sensitive SQL Details Hide করা |
| Audit Logging | Suspicious Activity Track করা |

## XSS Prevention

| Best Practice | Purpose |
|---|---|
| Output Encoding | HTML/JavaScript Injection প্রতিরোধ |
| Angular Interpolation | Safe Text Rendering |
| textContent | Safe DOM Manipulation |
| DOMPurify | প্রয়োজনীয় HTML Sanitization |
| CSP | Script Execution সীমিত করা |
| HttpOnly Cookie | JavaScript Cookie Access বন্ধ করা |
| Avoid innerHTML | Unsafe HTML Injection এড়ানো |

## Additional Security

Professional Application-এ আরও কিছু বিষয় অনুসরণ করতে হবে:

- HTTPS Everywhere
- JWT Signature Validation
- Authentication & Authorization
- Role-Based Access Control (RBAC)
- CSRF Protection
- Rate Limiting
- Secure Password Hashing
- Secrets Management
- Dependency Vulnerability Scanning
- Security Audit Logging
- Database Backup & Recovery

---

# 14. Final Security Checklist

### SQL Injection Checklist

- [ ] Parameterized Query ব্যবহার করেছি।
- [ ] ADO.NET Parameter Type নির্দিষ্ট করেছি।
- [ ] Stored Procedure-এ Unsafe Dynamic SQL ব্যবহার করিনি।
- [ ] Dynamic SQL-এ `sp_executesql` ব্যবহার করেছি।
- [ ] Dynamic Sort Column-এর জন্য Allowlist ব্যবহার করেছি।
- [ ] Database User-কে প্রয়োজনীয় Permission দিয়েছি।
- [ ] Sensitive SQL Exception User-কে দেখাই না।

### XSS Checklist

- [ ] Angular Interpolation ব্যবহার করেছি।
- [ ] Untrusted Input সরাসরি `innerHTML`-এ দিইনি।
- [ ] প্রয়োজন হলে HTML Sanitization ব্যবহার করেছি।
- [ ] `bypassSecurityTrustHtml()` এড়িয়ে চলেছি।
- [ ] CSP Configure করেছি।
- [ ] Authentication Cookie-তে HttpOnly ও Secure ব্যবহার করেছি।
- [ ] User-generated HTML Content Security Test করেছি।

---

# 🎯 Final Conclusion

SQL Injection এবং XSS দুটি গুরুত্বপূর্ণ Web Application Security Vulnerability।

**SQL Injection Protection:**

```text
User Input
    |
    v
Parameterized Query
    |
    v
Secure Stored Procedure
    |
    v
SQL Server
```

**XSS Protection:**

```text
User Input
    |
    v
Angular Sanitization
    |
    v
Safe DOM Rendering
    |
    v
Browser
```

### Remember

> Never trust user input.
>
> Always validate input, parameterize SQL queries, encode output and follow secure coding practices.

**Professional Developer-এর দায়িত্ব শুধু Application তৈরি করা নয়, Application-কে Secure রাখাও।**

---


**Topics:** `SQL Injection` `XSS` `Web Security` `ASP.NET Core` `ADO.NET` `SQL Server` `Stored Procedure` `Angular` `Secure Coding` `OWASP`


⬅️ [JWT ও OAuth 2.0](15-jwt-and-oauth2.md) | 🏠 [README](README.md)
