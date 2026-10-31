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

## ভাগ ১: JWT (JSON Web Token)

### ১. JWT কী?

একটি **ছোট, স্বাক্ষরিত, self-contained token** যাতে user-এর তথ্য (claims) থাকে। Server session মনে রাখে না (**stateless**) — token দেখেই বুঝে নেয়।

### ২. গঠন: তিন অংশ (`.` দিয়ে আলাদা)

```
xxxxx.yyyyy.zzzzz
Header.Payload.Signature
```

```
① Header     { "alg": "HS256", "typ": "JWT" }            ← কোন algorithm
② Payload    { "sub": "42", "name": "Abdullah",           ← Claims (তথ্য)
               "role": "Admin", "exp": 1760000000 }
③ Signature  HMACSHA256( base64(header) + "." + base64(payload), secret )
```
Header ও Payload শুধু **Base64URL encoded** — **এনক্রিপ্ট করা নয়!** যে কেউ পড়তে পারে।
Signature নিশ্চিত করে data **বদলানো হয়নি**।

> 🚨 **JWT-র Payload-এ password, সংবেদনশীল তথ্য রাখবেন না।**

### ৩. Standard Claims

| Claim | অর্থ |
|---|---|
| `iss` | Issuer (কে বানিয়েছে) |
| `sub` | Subject (user Id) |
| `aud` | Audience (কার জন্য) |
| `exp` | মেয়াদ শেষের সময় |
| `iat` | তৈরির সময় |
| `jti` | অনন্য token Id |

### ৪. কীভাবে কাজ করে?

```
① Login (username+password) ─────────────▶ Server
② Server যাচাই করে JWT বানায়, স্বাক্ষর করে
③ ◀──────────────────────────────────────── JWT
④ পরের request: Authorization: Bearer <JWT> ─▶ Server
⑤ Server স্বাক্ষর ও exp যাচাই করে → অনুমতি
```

### ৫. ASP.NET Core উদাহরণ

**Token তৈরি:**
```csharp
public string GenerateToken(User user)
{
    var claims = new[]
    {
        new Claim(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
        new Claim(ClaimTypes.Name, user.Name),
        new Claim(ClaimTypes.Role, user.Role)
    };

    var key   = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_config["Jwt:Key"]!));
    var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

    var token = new JwtSecurityToken(
        issuer: _config["Jwt:Issuer"],
        audience: _config["Jwt:Audience"],
        claims: claims,
        expires: DateTime.UtcNow.AddMinutes(15),   // ছোট মেয়াদ
        signingCredentials: creds);

    return new JwtSecurityTokenHandler().WriteToken(token);
}
```

**যাচাই (Program.cs):**
```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(o =>
    {
        o.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!)),
            ClockSkew = TimeSpan.FromSeconds(30)
        };
    });

app.UseAuthentication();
app.UseAuthorization();

[Authorize(Roles = "Admin")]
[HttpDelete("{id}")]
public IActionResult Delete(int id) { ... }
```

### ৬. Access Token + Refresh Token ⭐

| | Access Token | Refresh Token |
|---|---|---|
| মেয়াদ | **ছোট** (৫–১৫ মিনিট) | **দীর্ঘ** (দিন/সপ্তাহ) |
| কাজ | API call-এ ব্যবহার | নতুন access token আনা |
| সংরক্ষণ | — | **DB-তে** (বাতিল করা যায়) |
| চুরি গেলে | ক্ষতি সীমিত (দ্রুত মেয়াদ শেষ) | বাতিল (revoke) করা যায় |

```
Access token মেয়াদোত্তীর্ণ (401)
   └─▶ POST /auth/refresh { refreshToken }
         └─▶ নতুন Access Token (+ নতুন Refresh Token — rotation)
```

### ৭. JWT-র সুবিধা ও অসুবিধা

| ✅ সুবিধা | ❌ অসুবিধা |
|---|---|
| Stateless, scale করা সহজ | **মেয়াদের আগে বাতিল করা কঠিন** |
| Microservices-এ সুবিধাজনক | Token আকারে বড় |
| Cross-domain কাজ করে | চুরি গেলে মেয়াদ পর্যন্ত ব্যবহারযোগ্য |
| Self-contained | Payload সবাই পড়তে পারে |

### ৮. Token কোথায় রাখবেন? (Browser)

| জায়গা | ঝুঁকি |
|---|---|
| `localStorage` | **XSS**-এ চুরি যেতে পারে |
| **`HttpOnly` + `Secure` + `SameSite` Cookie** ⭐ | JS পড়তে পারে না; তবে **CSRF** সামলাতে হয় |

---

## ভাগ ২: OAuth 2.0

### ১. OAuth 2.0 কী?

> **তৃতীয় পক্ষকে আপনার password না দিয়ে, আপনার data-তে সীমিত প্রবেশাধিকার দেওয়ার** একটি standard পদ্ধতি।

**উদাহরণ:** একটি অ্যাপ বলল "Google Drive থেকে ছবি আনতে চাই।" আপনি Google-কে পাসওয়ার্ড দেন (অ্যাপকে নয়) এবং শুধু "ছবি পড়ার" অনুমতি দেন। অ্যাপ একটি **token** পায়।

### ২. চারটি ভূমিকা

| ভূমিকা | অর্থ | উদাহরণ |
|---|---|---|
| **Resource Owner** | data-র মালিক | আপনি (user) |
| **Client** | যে অ্যাপ অনুমতি চায় | আপনার Angular/Mobile অ্যাপ |
| **Authorization Server** | অনুমতি দেয়, token বানায় | Google, Auth0, Keycloak, IdentityServer |
| **Resource Server** | সুরক্ষিত API | Google Drive API, আপনার Backend |

### ৩. মূল শব্দ

- **Access Token** — API call-এর চাবি
- **Refresh Token** — নতুন access token আনতে
- **Scope** — কতটুকু অনুমতি (`read:orders`, `profile`)
- **Client ID / Client Secret** — অ্যাপের পরিচয়

### ৪. Grant Type (প্রবাহ)

| Grant | কখন | মন্তব্য |
|---|---|---|
| **Authorization Code + PKCE** ⭐ | Web/SPA/Mobile (user আছে) | **বর্তমানে প্রস্তাবিত, সবচেয়ে নিরাপদ** |
| **Client Credentials** | Machine-to-machine (user নেই) | Service ↔ Service |
| Device Code | TV/IoT (কীবোর্ড নেই) | — |
| ~~Implicit~~ | — | ❌ পরিত্যক্ত |
| ~~Password (ROPC)~~ | — | ❌ এড়িয়ে চলুন |

### ৫. Authorization Code + PKCE প্রবাহ

```
① User "Login" চাপে → Client, Authorization Server-এ পাঠায়
      (client_id, scope, redirect_uri, code_challenge ← PKCE)
② Authorization Server: User login করে ও অনুমতি দেয়
③ Server → Client-কে redirect করে সাথে একটি ক্ষণস্থায়ী "code"
④ Client → Server: code + code_verifier দেয়
⑤ Server → Client-কে Access Token (+ Refresh Token) দেয়
⑥ Client → Resource Server: Bearer Access Token
```
**PKCE** = code চুরি গেলেও যেন ব্যবহার না করা যায় (SPA/Mobile-এর জন্য অপরিহার্য)।

### ৬. Client Credentials (Service-to-Service)

```http
POST /connect/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=billing-service&client_secret=***&scope=orders.read
```

### ৭. OpenID Connect (OIDC)

OAuth 2.0 শুধু **অনুমতি** দেয়; user কে তা বলে না। OIDC যোগ করে একটি **ID Token (JWT)** — যাতে user-এর পরিচয় থাকে। "Login with Google/Microsoft" আসলে OIDC।

---

## ভাগ ৩: একসাথে বাস্তবে

```
Angular App ──(Auth Code + PKCE)──▶ Auth Server (Keycloak/Auth0/Entra ID)
     ◀────────────── Access Token (JWT) ──────────────
Angular App ──(Bearer JWT)──▶ ASP.NET Core API (Resource Server)
                                   └─ JWT-র স্বাক্ষর, exp, aud, scope যাচাই
```
এখানে **OAuth 2.0 = প্রবাহ**, **JWT = token-এর ফরম্যাট**।

---

## ৪. Professional Practices

1. **নিজে Auth Server বানাবেন না** — Keycloak, Auth0, Azure Entra ID, Duende IdentityServer ব্যবহার করুন
2. SPA/Mobile-এ **Authorization Code + PKCE**
3. **Access token ছোট মেয়াদের**, Refresh token rotation + revoke ব্যবস্থা
4. JWT-র **`iss`, `aud`, `exp`, স্বাক্ষর** সবসময় যাচাই
5. **Secret key শক্তিশালী ও গোপন** (কোডে নয়; Key Vault)। সম্ভব হলে **asymmetric (RS256)** — শুধু Auth Server private key রাখে, API শুধু public key দিয়ে যাচাই করে
6. **শুধু HTTPS**
7. **Scope/Role** দিয়ে ন্যূনতম অধিকার (Least Privilege)
8. Payload-এ সংবেদনশীল data নয়
9. `alg: none` সংক্রান্ত আক্রমণ ঠেকাতে algorithm সুনির্দিষ্টভাবে বাধ্য করুন
10. Token কখনো log-এ বা URL-এ নয়
11. Logout/চুরির সময় refresh token বাতিল

---

## ৫. সাধারণ ভুল

| ভুল | ঝুঁকি |
|---|---|
| JWT-তে password/PII | সবাই পড়তে পারে |
| দীর্ঘ মেয়াদের access token (দিন/মাস) | চুরি গেলে বড় ক্ষতি |
| `exp`/`aud` যাচাই না করা | অন্য সেবার token চলে |
| `localStorage`-এ token | XSS-এ চুরি |
| Implicit/Password grant | পরিত্যক্ত ও অনিরাপদ |
| JWT = Authentication ভাবা | OIDC/login প্রবাহ আলাদা |
| দুর্বল/ছোট secret key | Brute force-এ ভাঙা |

---

## ✅ সারসংক্ষেপ

- **JWT** = স্বাক্ষরিত token ফরম্যাট (Header.Payload.Signature); এনক্রিপ্টেড নয়
- **OAuth 2.0** = সীমিত অনুমতি দেওয়ার framework; **OIDC** = তার উপর login
- Access (ছোট) + Refresh (দীর্ঘ, বাতিলযোগ্য) token
- **Auth Code + PKCE** (user), **Client Credentials** (service)

⬅️ [HTTPS ও TLS](14-https-and-tls.md) | ➡️ [SQL Injection ও XSS](16-sql-injection-and-xss.md)
