# Chapter ১৩: CI/CD

# 🚀 CI/CD (Continuous Integration & Continuous Deployment/Delivery)
## A Complete Professional Guide – Basic to Advanced

**Technology:** Git, GitHub, GitHub Actions, .NET 8, Angular 20, Docker, SQL Server, Linux, AWS

---

# 📌 Table of Contents

1. What is CI/CD?
2. Why do we need CI/CD?
3. What is Continuous Integration (CI)?
4. What is Continuous Delivery (CD)?
5. What is Continuous Deployment?
6. CI/CD vs Traditional Deployment
7. How CI/CD Works
8. CI/CD Architecture
9. Important CI/CD Terminologies
10. GitHub Actions
11. CI/CD with ASP.NET Core
12. CI/CD with Angular 20
13. Professional CI/CD Pipeline
14. Docker Integration
15. AWS Deployment
16. Database Deployment Strategy
17. Environment Management
18. Security Best Practices
19. Rollback Strategy
20. Monitoring and Logging
21. Common CI/CD Mistakes
22. Real-World Project Example
23. Professional CI/CD Checklist
24. Interview Questions and Answers

---

# 1. What is CI/CD?

CI/CD is a software development practice that automates the process of building, testing, integrating, and deploying applications.

CI/CD = Continuous Integration + Continuous Delivery/Deployment.

এর মূল উদ্দেশ্য হচ্ছে Developer-এর Code Change থেকে শুরু করে Production Server-এ Application Deploy করা পর্যন্ত কাজগুলো Automation-এর মাধ্যমে সম্পন্ন করা।

### Example

ধরো, তুমি একটি Full Stack Application তৈরি করেছ।

**Technology:**
- Frontend: Angular 20
- Backend: ASP.NET Core 8 Web API
- Database: SQL Server
- Source Control: GitHub
- Deployment: AWS EC2

তুমি Backend-এ নতুন একটি API তৈরি করলে।

সাধারণত তোমাকে:

1. Code লিখতে হবে।
2. Build করতে হবে।
3. Unit Test চালাতে হবে।
4. Application Publish করতে হবে।
5. Server-এ Upload করতে হবে।
6. Application Restart করতে হবে।

CI/CD থাকলে এই কাজগুলোর অধিকাংশ Automatically সম্পন্ন হবে।

---

# 2. Why Do We Need CI/CD?

Professional Software Development-এ CI/CD অত্যন্ত গুরুত্বপূর্ণ।

### Without CI/CD

- Manual Build করতে হয়।
- Manual Testing করতে হয়।
- Manual Deployment করতে হয়।
- Deployment-এর সময় Human Error হতে পারে।
- Code Integration-এ Conflict হতে পারে।
- Bug Production-এ চলে যেতে পারে।
- Deployment করতে অনেক সময় লাগে।

### With CI/CD

- Automatic Build
- Automatic Testing
- Automatic Code Validation
- Automatic Deployment
- Faster Release
- Reduced Human Error
- Consistent Deployment
- Better Software Quality

### Real-World Comparison

| Feature | Without CI/CD | With CI/CD |
|---|---|---|
| Build | Manual | Automatic |
| Testing | Manual | Automatic |
| Deployment | Manual | Automatic |
| Code Validation | Limited | Automated |
| Release Speed | Slow | Fast |
| Human Error | High Risk | Reduced |
| Rollback | Often Manual | Can be Automated |

---

# 3. What is Continuous Integration (CI)?

Continuous Integration means developers frequently merge their code changes into a shared repository.

প্রতিবার Developer GitHub-এ Code Push বা Pull Request করলে Automated Pipeline চালু হয়।

### CI Workflow

1. Developer writes code.
2. Developer commits changes.
3. Developer pushes code to GitHub.
4. CI Pipeline starts.
5. Application builds.
6. Automated tests execute.
7. Code quality checks run.
8. Pipeline reports success or failure.

### Example

Developer একটি নতুন API তৈরি করল:

`GET /api/products`

তারপর GitHub-এ Push করল।

GitHub Actions automatically:

- .NET Restore করবে।
- .NET Build করবে।
- Unit Test চালাবে।
- Build Result দেখাবে।

যদি Test Fail করে, তাহলে Pipeline Failed হবে।

**Important:** Failed CI build সাধারণত Merge বা Release আটকে দেওয়ার জন্য ব্যবহার করা হয়।

---

# 4. What is Continuous Delivery (CD)?

Continuous Delivery means the application is automatically built, tested, and prepared for deployment.

কিন্তু Production-এ Deploy করার আগে সাধারণত Manual Approval প্রয়োজন হয়।

### Workflow

1. Code pushed to GitHub.
2. CI Build executes.
3. Automated tests pass.
4. Deployment artifact is generated.
5. Artifact is stored.
6. Deployment approval is requested.
7. Approved artifact is deployed to Production.

### Example

তুমি GitHub-এ Code Push করলে:

- Application Build হবে।
- Test হবে।
- Docker Image তৈরি হবে।
- Image Registry-তে Push হবে।
- Production Deployment-এর জন্য Ready থাকবে।

কিন্তু Production Deployment শুরু হবে Approval পাওয়ার পরে।

---

# 5. What is Continuous Deployment?

Continuous Deployment means every code change that successfully passes the required automated checks is automatically deployed to Production.

এখানে আলাদা Manual Approval প্রয়োজন হয় না।

### Workflow

Code Push → Build → Test → Security Scan → Deploy to Production

### Difference Between Continuous Delivery and Deployment

| Continuous Delivery | Continuous Deployment |
|---|---|
| Deployment Ready | Automatically Deployed |
| Manual Approval Usually Required | No Routine Manual Approval |
| Production Release Controlled | Production Release Automated |
| Suitable for Controlled Releases | Suitable for Mature Automation |

**Note:** CI = Integration, Continuous Delivery = Deployment Ready, Continuous Deployment = Automatically Released.

---

# 6. CI/CD vs Traditional Deployment

### Traditional Deployment

```text
Developer
    |
    v
Write Code
    |
    v
Manual Build
    |
    v
Manual Testing
    |
    v
Publish Application
    |
    v
Upload to Server
    |
    v
Restart Application
    |
    v
Production
```

### Automated CI/CD

```text
Developer
    |
    v
Git Push
    |
    v
GitHub Repository
    |
    v
CI Pipeline
    |
    +---- Restore
    |
    +---- Build
    |
    +---- Unit Testing
    |
    +---- Code Quality
    |
    v
Build Artifact
    |
    v
CD Pipeline
    |
    +---- Approval (Optional)
    |
    +---- Deploy
    |
    v
Production Server
    |
    v
Monitoring
```

---

# 7. How CI/CD Works

একটি Professional CI/CD Pipeline সাধারণত নিচের Stages অনুসরণ করে।

### Stage 1: Source Code Management

Tools:
- Git
- GitHub
- GitLab
- Bitbucket

Developer Code Push করে Repository-তে।

### Stage 2: Build

Tools:
- .NET SDK
- Node.js
- Angular CLI
- Maven
- Gradle

Source Code থেকে Executable বা Deployable Artifact তৈরি হয়।

### Stage 3: Testing

Tools:
- xUnit
- NUnit
- Jest
- Jasmine
- Playwright

Automated Testing করা হয়।

### Stage 4: Code Quality

Tools:
- SonarQube
- SonarCloud
- ESLint
- CodeQL

Code Quality, Security এবং Maintainability যাচাই করা হয়।

### Stage 5: Package

Tools:
- Docker
- Docker Hub
- GitHub Container Registry
- Amazon ECR

Application Package বা Container Image তৈরি করা হয়।

### Stage 6: Deployment

Tools:
- AWS EC2
- AWS ECS
- Kubernetes
- Azure App Service
- Linux Server
- Nginx

Application Server-এ Deploy হয়।

### Stage 7: Monitoring

Tools:
- Prometheus
- Grafana
- Loki
- AWS CloudWatch

Production Application Monitor করা হয়।

---

# 8. CI/CD Architecture

```text
                  DEVELOPER
                      |
                      v
                 Git Repository
                    GitHub
                      |
                      v
                GitHub Actions
                      |
              +-------+-------+
              |               |
              v               v
          Build CI        Pull Request
              |           Validation
              v
         Automated Test
              |
              v
        Code Quality Scan
              |
              v
         Build Artifact
              |
              v
         Docker Image
              |
              v
       Container Registry
              |
              v
       Deployment Pipeline
              |
              v
        AWS / Linux Server
              |
              v
         Production App
              |
              v
       Monitoring & Logging
```

---

# 9. Important CI/CD Terminologies

| Term | Explanation |
|---|---|
| Pipeline | Automated workflow |
| Workflow | Collection of automation jobs |
| Job | A group of related steps |
| Step | Individual task |
| Runner | Machine that executes jobs |
| Artifact | Build output |
| Build | Compile source code |
| Test | Validate application |
| Deployment | Release application |
| Environment | Dev, Staging, Production |
| Secret | Sensitive configuration |
| Trigger | Event that starts workflow |
| Rollback | Return to previous version |
| Registry | Stores Docker images |
| Release | A published application version |

### What is a Runner?

Runner হলো সেই Machine যেখানে GitHub Actions-এর Workflow Execute হয়।

Types:

1. GitHub-hosted Runner
2. Self-hosted Runner

**GitHub-hosted Runner:** GitHub-এর Managed Machine।

**Self-hosted Runner:** নিজের Server, VM অথবা EC2 Instance-এ Setup করা Runner।

---

# 10. What is GitHub Actions?

GitHub Actions is a CI/CD automation platform provided by GitHub.

এর মাধ্যমে Repository-এর ভিতর থেকেই Automated Workflow তৈরি করা যায়।

### Why GitHub Actions?

- GitHub-এর সঙ্গে Direct Integration।
- Build Automation।
- Testing Automation।
- Deployment Automation।
- Docker Integration।
- Cloud Deployment।
- Secrets Management।

### GitHub Actions Folder Structure

Project-এর Root Directory-তে:

```text
MyProject/
│
├── .github/
│   └── workflows/
│       ├── backend-ci.yml
│       ├── frontend-ci.yml
│       └── deploy.yml
│
├── backend/
│   └── MyApi/
│
├── frontend/
│   └── my-angular-app/
│
└── README.md
```

GitHub Actions-এর Workflow File সাধারণত `.github/workflows/` Directory-তে থাকে।

---

# 11. Practical Example: CI/CD with ASP.NET Core 8

এখন একটি Real-World ASP.NET Core Web API Project-এর CI Pipeline তৈরি করব।

### Step 1: Create .NET Project

Visual Studio 2022 দিয়ে:

1. Create New Project.
2. Select ASP.NET Core Web API.
3. Framework: .NET 8.
4. Enable HTTPS.
5. Create Project.

Example:

```text
MyCompany.API/
│
├── Controllers/
├── Services/
├── Repositories/
├── Models/
├── Program.cs
└── MyCompany.API.csproj
```

### Step 2: Add Unit Testing Project

Solution-এ একটি xUnit Test Project Add করো।

```text
MyCompany.sln
│
├── MyCompany.API/
│
└── MyCompany.Tests/
```

### Step 3: Create GitHub Repository

Terminal অথবা Visual Studio Git Integration ব্যবহার করে:

```bash
git init

git add .

git commit -m "Initial commit"

git branch -M main

git remote add origin https://github.com/USERNAME/REPOSITORY.git

git push -u origin main
```

### Step 4: Create GitHub Actions Workflow

Create this file:

`.github/workflows/backend-ci.yml`

```yaml
name: ASP.NET Core CI

on:
  push:
    branches:
      - main
      - develop

  pull_request:
    branches:
      - main

jobs:
  build-and-test:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Restore Dependencies
        run: dotnet restore MyCompany.sln

      - name: Build Application
        run: dotnet build MyCompany.sln --configuration Release --no-restore

      - name: Run Unit Tests
        run: dotnet test MyCompany.sln --configuration Release --no-build

      - name: Publish Application
        run: dotnet publish MyCompany.API/MyCompany.API.csproj -c Release -o ./publish

      - name: Upload Build Artifact
        uses: actions/upload-artifact@v4
        with:
          name: api-publish
          path: ./publish
```

**Note:** Solution এবং Project Path নিজের Repository অনুযায়ী পরিবর্তন করতে হবে।

### Step 5: Push Workflow

```bash
git add .

git commit -m "Add backend CI pipeline"

git push origin main
```

### Step 6: Check Pipeline

GitHub Repository:

1. Open GitHub.
2. Select Actions.
3. Select ASP.NET Core CI.
4. Open Latest Workflow Run.
5. Check Build Status.
6. Check Test Results.

<Badge>Expected Result</Badge>

```text
Checkout Source Code       ✓
Setup .NET                 ✓
Restore Dependencies      ✓
Build Application         ✓
Run Unit Tests            ✓
Publish Application       ✓
Upload Build Artifact     ✓

Workflow Status: SUCCESS
```

এখন প্রতিবার `main` অথবা `develop` Branch-এ Push করলে CI Pipeline Automatically Execute হবে।

---

# 12. Practical Example: CI with Angular 20

এখন Angular 20 Frontend-এর জন্য CI Pipeline তৈরি করব।

### Step 1: Angular Project

```bash
ng new my-angular-app
```

### Step 2: Create Workflow

File:

`.github/workflows/frontend-ci.yml`

```yaml
name: Angular 20 CI

on:
  push:
    branches:
      - main
      - develop

  pull_request:
    branches:
      - main

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json

      - name: Install Dependencies
        working-directory: frontend
        run: npm ci

      - name: Run Tests
        working-directory: frontend
        run: npm test -- --watch=false

      - name: Build Angular
        working-directory: frontend
        run: npm run build

      - name: Upload Build Artifact
        uses: actions/upload-artifact@v4
        with:
          name: angular-build
          path: frontend/dist/
```

**Important:** Angular 20-এর জন্য Node.js version project-এর supported version অনুযায়ী নির্বাচন করবে। Angular CLI project অনুযায়ী Test Command পরিবর্তন করতে হতে পারে।

### Expected Workflow

```text
Git Push
   |
   v
Checkout Code
   |
   v
Setup Node.js
   |
   v
npm ci
   |
   v
Run Tests
   |
   v
Angular Build
   |
   v
Upload Artifact
   |
   v
CI Success
```

---

# 13. Professional CI/CD Pipeline

Professional Company-তে শুধু Build এবং Deploy করলেই CI/CD Complete হয় না।

একটি Mature Pipeline-এ নিচের বিষয়গুলো থাকা উচিত।

### Professional Pipeline Stages

1. Source Control
2. Branch Validation
3. Dependency Restore
4. Build
5. Unit Testing
6. Integration Testing
7. Code Quality Analysis
8. Security Scanning
9. Artifact Generation
10. Artifact Versioning
11. Container Image Build
12. Container Registry Push
13. Staging Deployment
14. Smoke Testing
15. Production Approval
16. Production Deployment
17. Health Check
18. Monitoring
19. Rollback

### Professional Pipeline Flow

```text
Developer
   |
   v
Feature Branch
   |
   v
Pull Request
   |
   v
CI Validation
   |
   +---- Build
   |
   +---- Unit Test
   |
   +---- Integration Test
   |
   +---- Security Scan
   |
   v
Code Review
   |
   v
Merge to Main
   |
   v
Build Release Artifact
   |
   v
Docker Image
   |
   v
Container Registry
   |
   v
Deploy to Staging
   |
   v
Smoke Testing
   |
   v
Manual Approval
   |
   v
Production Deployment
   |
   v
Health Check
   |
   v
Monitoring
```

---

# 14. Docker Integration with CI/CD

Docker হলো Containerization Platform।

এর মাধ্যমে Application এবং তার Dependencies-কে একটি Container Image-এর মধ্যে Package করা যায়।

### Why Docker with CI/CD?

- Consistent Environment
- Easy Deployment
- Application Isolation
- Portability
- Faster Release
- Easy Scaling

### Example: ASP.NET Core Dockerfile

Create `Dockerfile` inside API project:

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build

WORKDIR /src

COPY . .

RUN dotnet restore MyCompany.API/MyCompany.API.csproj

RUN dotnet publish MyCompany.API/MyCompany.API.csproj \
    -c Release \
    -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0

WORKDIR /app

COPY --from=build /app/publish .

ENV ASPNETCORE_HTTP_PORTS=8080

EXPOSE 8080

ENTRYPOINT ["dotnet", "MyCompany.API.dll"]
```

### Build Docker Image

```bash
docker build -t mycompany-api:1.0 .
```

### Run Container

```bash
docker run -d \
  -p 8080:8080 \
  --name mycompany-api \
  mycompany-api:1.0
```

### Verify

```bash
docker ps
```

Test API:

```text
http://localhost:8080
```

### Docker Image Versioning

Professional Practice:

```text
mycompany-api:1.0.0
mycompany-api:1.0.1
mycompany-api:1.1.0
mycompany-api:latest
```

Production Deployment-এ `latest`-এর পরিবর্তে Immutable Version Tag অথবা Git Commit SHA ব্যবহার করা ভালো।

---

# 15. Deploy Docker Image to AWS EC2

ধরো তোমার কাছে:

- AWS EC2 Ubuntu Server
- Docker Installed
- GitHub Repository
- Docker Hub Account

তুমি CI/CD ব্যবহার করে EC2-তে Application Deploy করতে চাও।

### Step 1: Create AWS EC2

1. Login to AWS Console.
2. Open EC2.
3. Launch Instance.
4. Select Ubuntu Server.
5. Choose Instance Type.
6. Configure Security Group.
7. Create Key Pair.
8. Launch Instance.

### Step 2: Install Docker on EC2

Ubuntu Server-এ Docker-এর Official Installation Guide অনুসরণ করে Docker Engine Install করো।

Verify:

```bash
docker --version
```

### Step 3: Configure GitHub Secrets

GitHub Repository:

Settings → Secrets and variables → Actions

Add:

```text
DOCKER_USERNAME
DOCKER_PASSWORD
EC2_HOST
EC2_USER
EC2_SSH_KEY
```

**Security:** Password এবং SSH Private Key কখনো GitHub Repository-তে Hardcode করবে না। Docker Hub-এর ক্ষেত্রে Access Token ব্যবহার করবে।

### Step 4: CI/CD Deployment Workflow

File:

`.github/workflows/deploy.yml`

```yaml
name: Docker Build and Deploy

on:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:

  deploy:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and Push Docker Image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.DOCKER_USERNAME }}/mycompany-api:${{ github.sha }}

      - name: Deploy to EC2
        uses: appleboy/ssh-action@v1.2.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            docker pull ${{ secrets.DOCKER_USERNAME }}/mycompany-api:${{ github.sha }}

            docker stop mycompany-api || true

            docker rm mycompany-api || true

            docker run -d \
              --name mycompany-api \
              --restart unless-stopped \
              -p 8080:8080 \
              ${{ secrets.DOCKER_USERNAME }}/mycompany-api:${{ github.sha }}
```

**Important Production Considerations:**

এই Example একটি Basic Deployment Demonstration। Production-এ ব্যবহার করার আগে:

- SSH Host Key Verification Configure করতে হবে।
- SSH Access সীমিত করতে হবে।
- Database Connection Secret Securely Configure করতে হবে।
- Zero-Downtime Deployment Strategy ব্যবহার করতে হবে।
- Health Check এবং Rollback Implement করতে হবে।
- Docker Hub-এর পরিবর্তে AWS ECR ব্যবহার করা যেতে পারে।

---

# 16. Database Deployment Strategy

Professional .NET Application-এ Database Deployment খুব গুরুত্বপূর্ণ।

ধরো তোমার Application:

- ASP.NET Core Web API
- Angular 20
- SQL Server
- ADO.NET
- Stored Procedures

তাহলে Database Schema এবং Stored Procedure Change-গুলোও Version Control করতে হবে।

### Database Deployment Flow

```text
Developer
   |
   v
SQL Script
   |
   v
Git Repository
   |
   v
CI Validation
   |
   v
Database Migration Package
   |
   v
Staging Database
   |
   v
Validation
   |
   v
Production Approval
   |
   v
Production Database
```

### Example SQL Migration

File:

`Database/Scripts/001_Create_Products.sql`

```sql
CREATE TABLE Products
(
    Id INT IDENTITY(1,1) PRIMARY KEY,

    Name NVARCHAR(200) NOT NULL,

    Price DECIMAL(18,2) NOT NULL,

    CreatedAt DATETIME2 NOT NULL
        DEFAULT SYSUTCDATETIME()
);
```

### Stored Procedure

File:

`Database/Scripts/002_Create_Product_Procedure.sql`

```sql
CREATE OR ALTER PROCEDURE dbo.SP_GetProducts
AS
BEGIN
    SET NOCOUNT ON;

    SELECT
        Id,
        Name,
        Price,
        CreatedAt
    FROM dbo.Products
    ORDER BY Id DESC;
END;
```

### Recommended Database Deployment Tools

- DbUp
- Flyway
- Liquibase
- SQL Server Database Projects / DACPAC

**Best Practice:**

1. SQL Script Version Control করতে হবে।
2. Production Database Backup রাখতে হবে।
3. Migration Idempotency বিবেচনা করতে হবে।
4. Schema Change আগে Staging-এ Test করতে হবে।
5. Destructive Migration এড়িয়ে চলতে হবে।
6. Database Rollback-এর জন্য Backup এবং Forward-Fix Plan রাখতে হবে।

**Important:** Database Migration এবং Application Deployment একই সময়ে Atomicভাবে Rollback করা সবসময় সম্ভব নয়। তাই Backward-Compatible Schema Change ব্যবহার করা উচিত।

---

# 17. Environment Management

Professional Application-এ সাধারণত একাধিক Environment থাকে।

| Environment | Purpose |
|---|---|
| Development | Developer Testing |
| Testing | QA Testing |
| Staging | Production-like Testing |
| Production | Live Application |

### ASP.NET Core Configuration

```text
appsettings.json

appsettings.Development.json

appsettings.Staging.json

appsettings.Production.json
```

### Example

```json
{
  "ConnectionStrings": {
    "DefaultConnection": ""
  }
}
```

Production Database Connection String কখনো Source Code-এ Store করবে না।

Use:

- GitHub Secrets
- AWS Secrets Manager
- Environment Variables
- Azure Key Vault

### Environment-Based Deployment

```text
Develop Branch
     |
     v
Development

Main Branch
     |
     v
Staging
     |
     v
Approval
     |
     v
Production
```

---

# 18. CI/CD Security Best Practices

Professional DevOps Pipeline-এ Security অত্যন্ত গুরুত্বপূর্ণ।

### Important Security Practices

1. Never commit passwords or API keys.
2. Use GitHub Secrets.
3. Use Least Privilege Access.
4. Use HTTPS.
5. Scan dependencies for vulnerabilities.
6. Scan Docker Images.
7. Protect Main Branch.
8. Require Pull Request Reviews.
9. Use Environment Protection Rules.
10. Restrict Deployment Permissions.
11. Pin Third-Party Actions to Full Commit SHA.
12. Rotate Credentials Regularly.

### Recommended Security Tools

| Tool | Purpose |
|---|---|
| GitHub Dependabot | Dependency Updates |
| GitHub CodeQL | Code Security Analysis |
| SonarQube | Code Quality |
| Trivy | Container Vulnerability Scan |
| OWASP ZAP | Web Application Security Testing |
| GitHub Secret Scanning | Secret Detection |

---

# 19. Rollback Strategy

ধরো Production-এ নতুন Version Deploy করার পরে Application Crash করছে।

এখন আগের Stable Version-এ ফিরে যেতে হবে।

এটাকেই Rollback বলে।

### Example

```text
Version 1.0.0 → Stable
Version 1.0.1 → Failed
```

Rollback:

```text
Production
    |
    v
Version 1.0.1
    |
    v
Health Check Failed
    |
    v
Rollback Triggered
    |
    v
Version 1.0.0
    |
    v
Application Healthy
```

### Docker Rollback Example

```bash
docker pull mycompany-api:1.0.0

docker stop mycompany-api

docker rm mycompany-api

docker run -d \
  --name mycompany-api \
  --restart unless-stopped \
  -p 8080:8080 \
  mycompany-api:1.0.0
```

### Professional Rollback Strategies

- Previous Stable Artifact
- Blue-Green Deployment
- Canary Deployment
- Kubernetes Rolling Update
- Database Backup and Restore
- Feature Flags

---

# 20. Monitoring and Logging

Deployment Complete হলেই CI/CD-এর কাজ শেষ নয়।

Production Application Monitor করতে হবে।

### Recommended Tools

| Tool | Purpose |
|---|---|
| Prometheus | Metrics Collection |
| Grafana | Monitoring Dashboard |
| Loki | Log Aggregation |
| Grafana Alloy | Telemetry Collection |
| AWS CloudWatch | AWS Monitoring |
| OpenTelemetry | Distributed Observability |

### Monitor These Metrics

- CPU Usage
- Memory Usage
- HTTP Request Count
- API Response Time
- HTTP 5xx Errors
- Database Connection
- Application Availability
- Deployment Status

### Production Health Check

ASP.NET Core-এ Health Check Configure করা যায়।

`Program.cs`:

```csharp
builder.Services.AddHealthChecks();

var app = builder.Build();

app.MapHealthChecks("/health");

app.Run();
```

Test:

```text
GET /health
```

Deployment-এর পরে Health Check Success হওয়া নিশ্চিত করতে হবে।

---

# 21. Common CI/CD Mistakes

| Mistake | Solution |
|---|---|
| Hardcoded Secrets | GitHub Secrets |
| No Automated Tests | Add Unit Tests |
| Direct Production Deployment | Staging First |
| No Rollback Plan | Versioned Artifacts |
| Manual Database Changes | Versioned SQL Scripts |
| No Monitoring | Prometheus/Grafana |
| Using latest Image Tag | Immutable Image Tags |
| No Branch Protection | Protect Main Branch |
| No Security Scan | CodeQL/Trivy |
| Deploy Without Health Check | Automated Health Check |

---

# 22. Real-World Professional Project Example

## Full Stack .NET + Angular CI/CD Architecture

ধরো তুমি একটি Madrasa Donation Management Platform তৈরি করছ।

### Technology Stack

**Frontend**
- Angular 20
- TypeScript
- RxJS
- Angular Signals

**Backend**
- ASP.NET Core 8 Web API
- Clean Architecture
- CQRS
- ADO.NET
- Stored Procedures
- SQL Server

**DevOps**
- Git
- GitHub
- GitHub Actions
- Docker
- AWS EC2
- AWS ECR
- Nginx
- Prometheus
- Grafana

### Project Structure

```text
DonationPlatform/
│
├── backend/
│   ├── Donation.Domain/
│   ├── Donation.Application/
│   ├── Donation.Infrastructure/
│   ├── Donation.API/
│   └── Donation.Tests/
│
├── frontend/
│   └── donation-web/
│
├── database/
│   └── Scripts/
│
├── docker/
│   ├── Dockerfile.api
│   └── Dockerfile.frontend
│
├── .github/
│   └── workflows/
│       ├── backend-ci.yml
│       ├── frontend-ci.yml
│       ├── security.yml
│       └── production-deploy.yml
│
└── README.md
```

### Complete CI/CD Workflow

```text
                 DEVELOPER
                     |
                     v
                GitHub Push
                     |
                     v
                GitHub Actions
                     |
          +----------+----------+
          |                     |
          v                     v
     Backend CI            Frontend CI
          |                     |
          v                     v
     .NET Restore           npm ci
          |                     |
          v                     v
     .NET Build             Angular Build
          |                     |
          v                     v
     Unit Testing           Angular Tests
          |                     |
          +----------+----------+
                     |
                     v
               Security Scan
                     |
                     v
              Build Artifacts
                     |
                     v
              Docker Images
                     |
                     v
                 AWS ECR
                     |
                     v
              Deploy Staging
                     |
                     v
              Smoke Testing
                     |
                     v
             Manual Approval
                     |
                     v
             Deploy Production
                     |
          +----------+----------+
          |                     |
          v                     v
       Angular UI           .NET API
          |                     |
          +----------+----------+
                     |
                     v
                SQL Server
                     |
                     v
              Monitoring
```

### Professional Deployment Recommendation

For this architecture:

- Angular: Docker + Nginx অথবা Static Hosting.
- ASP.NET Core: Docker + AWS EC2/ECS.
- SQL Server: Managed Database অথবা Secured SQL Server Instance.
- Container Registry: AWS ECR.
- Secrets: AWS Secrets Manager.
- CI/CD: GitHub Actions.
- Monitoring: CloudWatch + Prometheus + Grafana.

Database-কে Public Internet-এ Expose করবে না।

---

# 23. Professional CI/CD Checklist

### Source Control

- [ ] Git Repository Created
- [ ] Feature Branch Strategy
- [ ] Pull Request Review
- [ ] Main Branch Protection

### CI Pipeline

- [ ] Automatic Restore
- [ ] Automatic Build
- [ ] Unit Testing
- [ ] Integration Testing
- [ ] Code Quality Scan
- [ ] Dependency Security Scan

### CD Pipeline

- [ ] Build Artifact
- [ ] Docker Image
- [ ] Image Versioning
- [ ] Container Registry
- [ ] Staging Deployment
- [ ] Production Approval
- [ ] Health Check
- [ ] Rollback Strategy

### Security

- [ ] GitHub Secrets
- [ ] No Hardcoded Credentials
- [ ] Least Privilege
- [ ] HTTPS
- [ ] Container Scanning
- [ ] Database Backup

### Monitoring

- [ ] Application Logs
- [ ] API Health Check
- [ ] CPU and Memory Monitoring
- [ ] Error Monitoring
- [ ] Deployment Monitoring

---

# 24. CI/CD Interview Questions and Answers

### Basic Questions

**Q1. What is CI/CD?**

Answer: CI/CD is a software development practice that automates code integration, testing, building, and application deployment.

**Q2. What is Continuous Integration?**

Answer: Continuous Integration is the practice of frequently merging code changes into a shared repository and automatically validating those changes.

**Q3. What is Continuous Delivery?**

Answer: Continuous Delivery automatically builds, tests, and prepares application releases for deployment, usually with manual production approval.

**Q4. What is Continuous Deployment?**

Answer: Continuous Deployment automatically releases every successfully validated change to Production.

**Q5. What is GitHub Actions?**

Answer: GitHub Actions is an automation platform used to build CI/CD workflows directly inside GitHub repositories.

### Intermediate Questions

**Q6. What is a Pipeline?**

Answer: A Pipeline is an automated sequence of tasks that builds, tests, packages, or deploys an application.

**Q7. What is a Runner?**

Answer: A Runner is a machine that executes GitHub Actions workflow jobs.

**Q8. What is an Artifact?**

Answer: An Artifact is a generated build output that can be stored and used in later pipeline stages.

**Q9. What is Docker's role in CI/CD?**

Answer: Docker packages applications and dependencies into portable container images for consistent deployment.

**Q10. What is Rollback?**

Answer: Rollback means restoring a previously stable application version after a failed deployment.

### Advanced Questions

**Q11. What is Blue-Green Deployment?**

Answer: Blue-Green Deployment maintains two production environments. One serves live traffic while the other receives the new release. Traffic switches after validation.

**Q12. What is Canary Deployment?**

Answer: Canary Deployment gradually releases a new version to a small percentage of users before expanding to everyone.

**Q13. What is Infrastructure as Code (IaC)?**

Answer: IaC is the practice of managing infrastructure using version-controlled configuration files.

Tools: Terraform, AWS CloudFormation, Ansible.

**Q14. What is a Self-hosted Runner?**

Answer: A Self-hosted Runner is a machine managed by the organization to execute CI/CD workflows.

**Q15. How do you secure a CI/CD Pipeline?**

Answer:
- Use Secrets Management.
- Apply Least Privilege.
- Scan Dependencies.
- Protect Branches.
- Validate Pull Requests.
- Secure Deployment Credentials.
- Restrict Production Access.

**Q16. How do you deploy a .NET API using GitHub Actions?**

Answer:

1. Checkout Source Code.
2. Setup .NET SDK.
3. Restore Dependencies.
4. Build Application.
5. Run Tests.
6. Publish Application.
7. Generate Docker Image.
8. Push Image to Registry.
9. Deploy to Server.
10. Run Health Checks.

**Q17. How do you handle database changes in CI/CD?**

Answer: Store version-controlled SQL migration scripts and execute them through a controlled database deployment process. Validate in Staging and maintain backup and recovery plans.

**Q18. What happens if a CI Pipeline fails?**

Answer: The failed workflow reports errors. A protected Pull Request should not be merged until the required checks pass.

**Q19. What is the difference between GitHub-hosted and Self-hosted Runners?**

Answer: GitHub-hosted runners are managed by GitHub, while self-hosted runners are managed by the organization.

**Q20. What is Zero-Downtime Deployment?**

Answer: Zero-Downtime Deployment releases a new application version without interrupting service availability, typically using rolling, blue-green, or traffic-switching strategies.

---

# 🎯 Final Learning Roadmap

CI/CD শেখার জন্য নিচের Sequence Follow করো।

| Level | Topics | Practice |
|---|---|---|
| 1 | Git & GitHub | Push, Pull, Branch |
| 2 | GitHub Actions | First Workflow |
| 3 | .NET CI | Restore, Build, Test |
| 4 | Angular CI | npm ci, Build, Test |
| 5 | Docker | Dockerfile, Image, Container |
| 6 | Docker Registry | Docker Hub / AWS ECR |
| 7 | CD Pipeline | Automated Deployment |
| 8 | AWS EC2 | Linux Deployment |
| 9 | Security | Secrets, CodeQL, Trivy |
| 10 | Advanced CI/CD | Blue-Green, Canary |
| 11 | Monitoring | Prometheus, Grafana |
| 12 | Infrastructure | Terraform, Kubernetes |

### Recommended Tools to Learn

| Category | Tools |
|---|---|
| Version Control | Git, GitHub |
| CI/CD | GitHub Actions, Jenkins |
| Build | .NET SDK, Node.js |
| Testing | xUnit, NUnit, Playwright |
| Code Quality | SonarQube |
| Security | CodeQL, Trivy |
| Containerization | Docker |
| Container Registry | Docker Hub, AWS ECR |
| Cloud | AWS, Azure |
| Web Server | Nginx |
| Infrastructure | Terraform, Ansible |
| Orchestration | Kubernetes |
| Monitoring | Prometheus, Grafana |
| Logging | Loki, Grafana Alloy |

---

## Conclusion

CI/CD is one of the most important practices in modern software development.

A professional Full Stack Developer should understand how to:

1. Manage Source Code using Git.
2. Build Automated CI Pipelines.
3. Run Automated Tests.
4. Integrate Docker.
5. Automate Deployment.
6. Secure Application Secrets.
7. Manage Database Changes.
8. Monitor Production Applications.
9. Implement Rollback Strategies.

**Remember:**

`Write Code → Commit → Push → Build → Test → Package → Deploy → Monitor`

This is the foundation of modern CI/CD and DevOps Engineering.

---

**Author:** Full Stack .NET Developer

**Technologies:** ASP.NET Core | Angular 20 | SQL Server | ADO.NET | GitHub Actions | Docker | AWS

**Purpose:** Learning, Practical Implementation, Professional Development & Interview Preparation.

⬅️ [gRPC ও GraphQL](12-grpc-and-graphql.md) | ➡️ [HTTPS ও TLS](14-https-and-tls.md)
