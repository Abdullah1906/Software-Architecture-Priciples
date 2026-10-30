# Chapter ১৩: CI/CD

## ১. CI/CD কী?

**CI** = Continuous Integration | **CD** = Continuous Delivery / Continuous Deployment

কোড লেখা থেকে production-এ পৌঁছানো পর্যন্ত **build → test → deploy** ধাপগুলো **স্বয়ংক্রিয়** করা।

### আগের পদ্ধতি (ম্যানুয়াল) ❌
```
কোড লেখা → নিজের পিসিতে build → ZIP → FTP/RDP করে server-এ কপি → আশা করা কাজ করবে 🙏
```
সমস্যা: ভুল হয়, ধীর, "আমার মেশিনে তো চলছিল", বিশৃঙ্খল।

### CI/CD-র পদ্ধতি ✅
```
git push → [স্বয়ংক্রিয়] build → test → scan → package → deploy
```

---

## ২. তিনটি ধারণার পার্থক্য

| | কী করে |
|---|---|
| **Continuous Integration (CI)** | প্রতিটি push-এ স্বয়ংক্রিয় build ও test; দলের কোড ঘন ঘন merge |
| **Continuous Delivery** | CI-র পর কোড **সবসময় deploy-যোগ্য**; production-এ যাওয়ার আগে **ম্যানুয়াল অনুমোদন** |
| **Continuous Deployment** | সব test পাস করলে **স্বয়ংক্রিয়ভাবে production-এ** |

---

## ৩. কেন দরকার?

- দ্রুত ও ঘন ঘন release
- Bug **আগেই** ধরা পড়ে (ছোট পরিবর্তনে খুঁজে পাওয়া সহজ)
- মানুষের ভুল কমে
- সবসময় একই প্রক্রিয়া (repeatable)
- দলের আস্থা বাড়ে; "Friday deploy"-এর ভয় কমে

---

## ৪. সাধারণ Pipeline-এর ধাপ

```
Code Push
   │
   ▼
① Checkout & Restore  ─ কোড আনা, package install
② Build               ─ compile
③ Unit Test           ─ দ্রুত test
④ Code Quality/Lint   ─ SonarQube, analyzers
⑤ Security Scan       ─ dependency vulnerability, secret scan
⑥ Package             ─ Docker image / artifact
⑦ Integration Test    ─ DB সহ test
⑧ Deploy → Staging    ─ পরীক্ষামূলক পরিবেশ
⑨ Smoke/E2E Test
⑩ Approval (ঐচ্ছিক)
⑪ Deploy → Production
⑫ Monitor & Rollback
```
**মূল নীতি: Fail Fast** — সস্তা ও দ্রুত ধাপ (build, unit test) আগে, ধীর ধাপ পরে।

---

## ৫. GitHub Actions উদাহরণ (.NET + Docker)

`.github/workflows/ci-cd.yml`

```yaml
name: CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Restore
        run: dotnet restore

      - name: Build
        run: dotnet build --no-restore -c Release

      - name: Test
        run: dotnet test --no-build -c Release --verbosity normal

  docker-publish:
    needs: build-test                      # test পাস না করলে চলবে না
    if: github.ref == 'refs/heads/main'    # শুধু main-এ
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Login to registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build & push image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}

  deploy:
    needs: docker-publish
    runs-on: ubuntu-latest
    environment: production                # এখানে manual approval সেট করা যায়
    steps:
      - name: Deploy
        run: echo "এখানে server/Kubernetes/Cloud-এ deploy command"
```

### পরিভাষা
- **Workflow** = পুরো pipeline | **Job** = ধাপের গ্রুপ | **Step** = একটি কাজ
- **Runner** = যে মেশিনে চলে | **Secrets** = গোপন মান (password, key)

---

## ৬. Deployment Strategy

| কৌশল | কীভাবে | সুবিধা | ঝুঁকি |
|---|---|---|---|
| **Recreate** | পুরনো বন্ধ → নতুন চালু | সরল | Downtime |
| **Rolling** | একটি একটি করে instance বদলানো | Downtime নেই | দুই version একসাথে চলে |
| **Blue-Green** ⭐ | নতুন (Green) পরিবেশ তৈরি, traffic সুইচ | তাৎক্ষণিক rollback | দ্বিগুণ অবকাঠামো |
| **Canary** ⭐ | প্রথমে ৫% user-কে নতুন version | ঝুঁকি কম, বাস্তব পরীক্ষা | জটিল monitoring |
| **Feature Flag** | কোড deploy, feature চালু/বন্ধ আলাদা | Deploy ও release আলাদা | Flag জঞ্জাল |

---

## ৭. Branching Strategy

| | বর্ণনা |
|---|---|
| **Trunk-Based** ⭐ | সবাই ছোট branch-এ কাজ করে দ্রুত `main`-এ merge; CI/CD-র জন্য আদর্শ |
| **GitFlow** | `develop`, `release`, `hotfix` — বেশি structure, ধীর |
| **GitHub Flow** | `main` + feature branch + Pull Request (সরল) |

**Pull Request-এ নিয়ম:** build পাস + test পাস + কমপক্ষে একজনের review ছাড়া merge নয়।

---

## ৮. Professional Practices

1. **Pipeline as Code** (YAML repo-তেই)
2. **Secrets কখনো কোডে নয়** — GitHub Secrets / Azure Key Vault / AWS Secrets Manager
3. **একবার build, সব পরিবেশে একই artifact** (Docker image) deploy
4. **Environment আলাদা:** Dev → Staging → Production
5. **Database migration স্বয়ংক্রিয়** ও backward-compatible (EF Migrations / DbUp)
6. **Rollback পরিকল্পনা** আগে থেকেই
7. **Pipeline দ্রুত রাখুন** — caching, parallel job (১০ মিনিটের নিচে ভালো)
8. **Test pyramid:** অনেক unit test, কিছু integration, অল্প E2E
9. **Security scan** pipeline-এ (Dependabot, Trivy, CodeQL)
10. **Infrastructure as Code** (Terraform, Bicep, Ansible)
11. **Monitoring ও alert** deploy-এর পর
12. **Health check** পাস না হলে স্বয়ংক্রিয় rollback

---

## ৯. সাধারণ ভুল

| ভুল | ফল |
|---|---|
| Test ছাড়া CI | শুধু build করা, নিরাপত্তা নেই |
| Production-এ সরাসরি manual deploy | অনির্ভরযোগ্য |
| Secret কোডে/log-এ | নিরাপত্তা ফাঁস |
| বিশাল, ধীর pipeline | সবাই এড়িয়ে চলে |
| ভাঙা build ঠিক না করা | CI অর্থহীন হয়ে যায় |
| Rollback ভাবা হয়নি | সমস্যায় আতঙ্ক |

---

## ✅ সারসংক্ষেপ

- **CI** = প্রতি push-এ build + test | **CD** = স্বয়ংক্রিয় release
- Pipeline: build → test → scan → package → deploy
- Blue-Green/Canary দিয়ে নিরাপদ deploy, rollback প্রস্তুত রাখুন
- Secret নিরাপদ রাখুন, pipeline দ্রুত রাখুন

⬅️ [gRPC ও GraphQL](12-grpc-and-graphql.md) | ➡️ [HTTPS ও TLS](14-https-and-tls.md)
