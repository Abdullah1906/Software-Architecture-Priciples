# Chapter ১৪: HTTPS ও TLS

## ১. সমস্যা: HTTP কেন নিরাপদ নয়?

সাধারণ **HTTP**-তে data **plain text** হিসেবে যায়। পথে (Wi-Fi, ISP, router) যে কেউ দেখতে বা বদলাতে পারে।

```
আপনি ──[ password=1234 ]──▶ 👀 হ্যাকার ──▶ Server
                          (সব পড়তে পারে!)
```

তিনটি ঝুঁকি:
1. **আড়ি পাতা** (Eavesdropping) — password, token চুরি
2. **বদলে দেওয়া** (Tampering) — মাঝপথে data পরিবর্তন
3. **ভুয়া server** (Impersonation) — আপনি ভাবছেন ব্যাংকের সাথে কথা বলছেন, আসলে হ্যাকারের সাথে

---

## ২. HTTPS ও TLS কী?

- **HTTPS** = HTTP + **TLS** (HTTP over TLS)
- **TLS** (Transport Layer Security) = যোগাযোগকে নিরাপদ করার protocol। (পুরনো নাম **SSL**; এখন সবাই TLS ব্যবহার করে, যদিও "SSL certificate" নামটি রয়ে গেছে)

### TLS তিনটি গ্যারান্টি দেয়

| গ্যারান্টি | মানে | কীভাবে |
|---|---|---|
| **Confidentiality** (গোপনীয়তা) | অন্যরা পড়তে পারবে না | Encryption |
| **Integrity** (অখণ্ডতা) | মাঝপথে বদলালে ধরা পড়ে | MAC / AEAD |
| **Authentication** (পরিচয় যাচাই) | সত্যিই সঠিক server-এর সাথে কথা হচ্ছে | Certificate |

---

## ৩. মূল ধারণা: দুই ধরনের Encryption

| | Symmetric | Asymmetric |
|---|---|---|
| Key | **একটিই** key (এনক্রিপ্ট ও ডিক্রিপ্ট) | **Key জোড়া:** Public + Private |
| গতি | **দ্রুত** | ধীর |
| সমস্যা | Key নিরাপদে পাঠানো কঠিন | Key বিনিময়ে সুবিধা |
| ব্যবহার | **আসল data** encrypt | Key বিনিময় ও পরিচয় যাচাই |

> TLS **দুটোই ব্যবহার করে:** Asymmetric দিয়ে নিরাপদে একটি গোপন key ঠিক করে, তারপর দ্রুত Symmetric দিয়ে data আদানপ্রদান।

**Public/Private Key:**
- Public key → সবাইকে দেওয়া যায়; দিয়ে **encrypt** করা যায়
- Private key → **শুধু মালিকের কাছে**; দিয়ে **decrypt** বা **স্বাক্ষর** করা যায়

---

## ৪. Certificate ও CA

### Certificate (প্রমাণপত্র)
Server-এর **Public Key + পরিচয় (domain) + মেয়াদ + একটি স্বাক্ষর** — একটি ডিজিটাল পরিচয়পত্র।

### Certificate Authority (CA)
বিশ্বস্ত তৃতীয় পক্ষ (Let's Encrypt, DigiCert, Sectigo) যে যাচাই করে certificate-এ স্বাক্ষর করে।

```
Root CA (browser/OS-এ আগে থেকেই বিশ্বস্ত)
   └── Intermediate CA
          └── আপনার server-এর Certificate (example.com)
```
Browser এই **Chain of Trust** যাচাই করে। কোথাও ভাঙলে "Your connection is not private" সতর্কতা আসে।

---

## ৫. TLS Handshake (সরল ব্যাখ্যা)

```
Client                                          Server
  │ ① Client Hello (সমর্থিত cipher, TLS version) ─▶│
  │◀─ ② Server Hello + Certificate ────────────────│
  │ ③ Certificate যাচাই (CA বিশ্বস্ত? domain মিলছে? মেয়াদ আছে?)
  │ ④ গোপন key নিয়ে সম্মত (Key Exchange) ◀────────▶│
  │ ⑤ দুজনেই একই Session Key বানায়                │
  │◀═══ এখন থেকে সব data Symmetric encrypted ═════▶│
```

- **TLS 1.2** — এখনো ব্যাপক ব্যবহৃত
- **TLS 1.3** ⭐ — দ্রুততর (কম round-trip), বেশি নিরাপদ; পুরনো দুর্বল algorithm বাদ
- **Forward Secrecy** — ভবিষ্যতে server-এর private key চুরি হলেও পুরনো যোগাযোগ ডিক্রিপ্ট হয় না

---

## ৬. ASP.NET Core-এ HTTPS

```csharp
var app = builder.Build();

app.UseHttpsRedirection();   // HTTP → HTTPS-এ পাঠায়
app.UseHsts();               // Browser-কে বলে: এই site সবসময় HTTPS-এ খুলবে (production-এ)
```

Development-এ:
```bash
dotnet dev-certs https --trust
```

### Production-এ Certificate
- **Let's Encrypt** — বিনামূল্যে, স্বয়ংক্রিয় নবায়ন (certbot / Caddy / cloud provider)
- **Reverse Proxy-তে TLS termination:** Nginx/Cloud Load Balancer HTTPS সামলায়, ভেতরে Kestrel-এ HTTP

```nginx
server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate     /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;

    location / {
        proxy_pass http://localhost:5000;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
server { listen 80; return 301 https://$host$request_uri; }
```
> Proxy-র পেছনে থাকলে `UseForwardedHeaders` কনফিগার করুন, নইলে app ভাবে সবই HTTP।

---

## ৭. mTLS (Mutual TLS)

সাধারণ TLS-এ শুধু **client server-কে** যাচাই করে। **mTLS**-এ **দুজনেই দুজনকে** certificate দিয়ে যাচাই করে।

**ব্যবহার:** Microservices-এর ভেতরের যোগাযোগ, Zero Trust নেটওয়ার্ক, ব্যাংকিং API, Service Mesh (Istio)।

---

## ৮. Professional Practices

1. **সব কিছুতে HTTPS** — শুধু login পেজ নয়, পুরো site ও API
2. **TLS 1.2+ (সম্ভব হলে 1.3)**; SSL 3.0, TLS 1.0/1.1 বন্ধ
3. **HSTS** চালু করুন
4. Certificate **স্বয়ংক্রিয় নবায়ন** (মেয়াদ শেষে site বন্ধ হওয়ার ঘটনা খুব সাধারণ!)
5. **Secure cookie flag** — `Secure; HttpOnly; SameSite`
6. Private key **কখনো repo-তে নয়**, কঠোর access control
7. ভেতরের নেটওয়ার্কেও এনক্রিপশন বিবেচনা (mTLS)
8. **SSL Labs** (ssllabs.com/ssltest) দিয়ে নিজের site পরীক্ষা করুন
9. Mixed content এড়ান (HTTPS পেজে HTTP resource)
10. Database connection-এও TLS (`Encrypt=True` — SQL Server)

```
Server=...;Database=...;Encrypt=True;TrustServerCertificate=False;
```
> ⚠️ `TrustServerCertificate=True` শুধু development-এ। Production-এ নয়।

---

## ৯. সাধারণ ভুল

| ভুল | ঝুঁকি |
|---|---|
| Certificate মেয়াদোত্তীর্ণ | Site-এ ভয়ংকর সতর্কতা, API call ব্যর্থ |
| Self-signed cert production-এ | Browser বিশ্বাস করে না |
| Certificate যাচাই বন্ধ (`ServerCertificateCustomValidationCallback = true`) | Man-in-the-middle আক্রমণ |
| HTTP থেকে HTTPS redirect নেই | ব্যবহারকারী অসুরক্ষিতভাবে ঢোকে |
| পুরনো TLS সংস্করণ চালু | জানা দুর্বলতা |
| Token/password URL-এ | Log ও history-তে ফাঁস (HTTPS-এও) |

---

## ✅ সারসংক্ষেপ

- **HTTPS = HTTP + TLS** → গোপনীয়তা + অখণ্ডতা + পরিচয় যাচাই
- Asymmetric দিয়ে key ঠিক, Symmetric দিয়ে data encrypt
- Certificate ও CA = "এই server আসল" এর প্রমাণ
- Let's Encrypt, HSTS, TLS 1.2/1.3, স্বয়ংক্রিয় নবায়ন

⬅️ [CI/CD](13-ci-cd.md) | ➡️ [JWT ও OAuth 2.0](15-jwt-and-oauth2.md)
