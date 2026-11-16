# Vector Database & Embeddings — Complete Guide (বাংলায়)

> Beginner থেকে Professional/Production-grade ধারণা — বিশেষভাবে C#, ASP.NET Core, SQL Server এবং AI Application Developers-এর জন্য।

## সূচিপত্র

- [1. Embedding কী?](#1-embedding-কী)
- [2. Embedding কেন প্রয়োজন?](#2-embedding-কেন-প্রয়োজন)
- [3. Vector কী?](#3-vector-কী)
- [4. Vector Database কী?](#4-vector-database-কী)
- [5. Embedding বনাম Vector Database](#5-embedding-বনাম-vector-database)
- [6. Similarity Search কীভাবে কাজ করে?](#6-similarity-search-কীভাবে-কাজ-করে)
- [7. ব্যবহারক্ষেত্র](#7-ব্যবহারক্ষেত্র)
- [8. RAG Architecture](#8-rag-architecture)
- [9. Data Ingestion ও Query Pipeline](#9-data-ingestion-ও-query-pipeline)
- [10. Chunking কী?](#10-chunking-কী)
- [11. .NET Developer-এর Practical Roadmap](#11-net-developer-এর-practical-roadmap)
- [12. Production-grade Design](#12-production-grade-design)
- [13. SQL Server বনাম Vector Database](#13-sql-server-বনাম-vector-database)
- [14. Common Mistakes](#14-common-mistakes)
- [15. Learning Roadmap](#15-learning-roadmap)
- [16. Interview Questions](#16-interview-questions)
- [17. Official Resources](#17-official-resources)

---

## 1. Embedding কী?

**Embedding হলো এমন একটি পদ্ধতি, যার মাধ্যমে Text, Image, Audio বা অন্য Data-কে সংখ্যার একটি Array (Vector)-এ রূপান্তর করা হয়।**

AI Embedding Model ডেটার অর্থ ও Context অনুযায়ী Numerical Representation তৈরি করে।

উদাহরণ:

```text
Text 1: I love programming.
Text 2: I enjoy coding.
Text 3: I love eating rice.
```

একটি Embedding Model এগুলোকে এমন Vector-এ রূপান্তর করতে পারে:

```text
Text 1 -> [0.21, 0.45, 0.78, 0.12]
Text 2 -> [0.23, 0.43, 0.80, 0.15]
Text 3 -> [0.89, 0.12, 0.34, 0.67]
```

> উপরের Vector-গুলো সম্পূর্ণ কাল্পনিক; শুধু ধারণা বোঝানোর জন্য। বাস্তব Embedding-এ Dimension অনেক বেশি হতে পারে।

Text 1 এবং Text 2-এর অর্থ কাছাকাছি। তাই উপযুক্ত Embedding Model সাধারণত তাদের Vector-কে কাছাকাছি অবস্থানে রাখবে। Text 3-এর অর্থ আলাদা হওয়ায় তার Vector তুলনামূলক দূরে থাকতে পারে।

**মূল কথা:** Embedding ডেটার Semantic Meaning-কে Machine-readable Numerical Representation-এ প্রকাশ করে।

### Embedding Dimension

Vector-এর মধ্যে যতগুলো সংখ্যা থাকে, সেটিই তার Dimension।

```csharp
float[] embedding =
{
    0.12f,
    0.45f,
    0.78f,
    0.91f
};

// এই উদাহরণে Dimension = 4
```

উদাহরণ হিসেবে OpenAI-এর `text-embedding-3-small` Model-এর Default Dimension 1536 এবং `text-embedding-3-large`-এর Default Dimension 3072। Model বা API-তে Dimension পরিবর্তনের সুবিধা থাকলে সেটিও Configuration অনুযায়ী নির্ধারিত হয়।

মনে রাখবে:

- Dimension বেশি হলেই Search সবসময় ভালো হবে না।
- Embedding Model নির্ধারণ করে Vector-এর Dimension।
- Query এবং Indexed Document-এর Embedding একই Compatible Model/Vector Space থেকে তৈরি হওয়া উচিত।
- Model পরিবর্তন করলে Re-embedding প্রয়োজন হতে পারে।

## 2. Embedding কেন প্রয়োজন?

ধরো, Database-এ আছে:

```text
Document:
"How to reset my password?"
```

User Search করল:

```text
"I forgot my login credentials."
```

Traditional Keyword Search হয়তো সঠিক Document খুঁজে পাবে না, কারণ `reset`, `forgot`, `password` এবং `credentials` আলাদা শব্দ।

Semantic Search শব্দের পাশাপাশি অর্থ ও Context-এর মিল খুঁজতে পারে।

```text
Document Text
     |
     v
Embedding Model
     |
     v
Document Vector

User Query
     |
     v
Same Embedding Model
     |
     v
Query Vector
     |
     v
Similarity Search
     |
     v
Relevant Document
```

**Embedding-এর প্রধান উদ্দেশ্য:** Exact Keyword না মিললেও অর্থের ভিত্তিতে সম্পর্কিত Data খুঁজে পেতে সাহায্য করা।

## 3. Vector কী?

Vector হলো সংখ্যার একটি Ordered Collection বা Array।

```csharp
float[] vector =
{
    0.12f,
    0.45f,
    0.78f,
    0.91f
};
```

Embedding Model এই ধরনের Vector তৈরি করে। প্রতিটি সংখ্যা আলাদাভাবে মানুষের কাছে সরাসরি অর্থপূর্ণ নাও হতে পারে; পুরো Vector-টি Model-এর শেখা Semantic Space-এ Data-এর Representation।

## 4. Vector Database কী?

**Vector Database হলো এমন Database/System, যা Numerical Vector সংরক্ষণ, Indexing এবং Similarity Search করার জন্য তৈরি।**

সাধারণ SQL Query:

```sql
SELECT *
FROM Employees
WHERE FullName = 'Abdullah';
```

এটি নির্দিষ্ট Value দিয়ে Exact Match খোঁজে।

Vector Search-এর ধারণা:

```text
User Query
    |
    v
Embedding Model
    |
    v
Query Vector
    |
    v
Vector Database
    |
    v
Similarity Search
    |
    v
Top-K Similar Results
```

Vector Database সাধারণত Vector-এর পাশাপাশি Metadata/Payload-ও রাখতে পারে। যেমন Document ID, Tenant ID, Source, Page Number এবং Category।

### Popular Vector Database / Search Systems

| Technology | পরিচিত ব্যবহার |
|---|---|
| [Qdrant](https://qdrant.tech/documentation/) | Vector Search, Filtering, Hybrid Retrieval |
| [Pinecone](https://docs.pinecone.io/) | Managed Vector Database |
| [Weaviate](https://docs.weaviate.io/) | Vector Search ও Hybrid Search |
| [Milvus](https://milvus.io/docs) | Large-scale Vector Search |
| [PostgreSQL + pgvector](https://github.com/pgvector/pgvector) | PostgreSQL-এর মধ্যে Vector Search |
| [Azure AI Search](https://learn.microsoft.com/azure/search/) | Enterprise Search ও Vector Retrieval |
| [SQL Server Vector Search](https://learn.microsoft.com/sql/) | Supported SQL Server/Azure SQL versions-এ Vector capability |

সব Project-এর জন্য Dedicated Vector Database লাগবেই এমন নয়। বিদ্যমান Database-এ প্রয়োজনীয় Vector Search Feature থাকলে সেটিও উপযুক্ত হতে পারে।

## 5. Embedding বনাম Vector Database

| বিষয় | Embedding | Vector Database |
|---|---|---|
| কাজ | Data-কে Vector-এ রূপান্তর | Vector Store ও Search |
| কী তৈরি/পরিচালনা করে? | Numerical Representation | Searchable Records |
| প্রধান প্রযুক্তি | Embedding Model | Database / Search Engine |
| উদাহরণ | OpenAI Embedding Model | Qdrant |
| উদ্দেশ্য | Semantic Representation | Similarity Retrieval |

সহজভাবে:

- **Embedding Model:** Text-এর অর্থকে Vector বানায়।
- **Vector Database:** Vector সংরক্ষণ করে এবং কাছাকাছি Vector খুঁজে দেয়।

## 6. Similarity Search কীভাবে কাজ করে?

ধরো, আমাদের কাছে তিনটি Document আছে:

```text
A: ASP.NET Core is used to build Web APIs.
B: C# is a programming language.
C: Angular is a frontend framework.
```

User Query:

```text
"I want to create a backend API using .NET."
```

প্রতিটি Document এবং User Query Embedding-এ রূপান্তরিত হবে। এরপর Similarity Metric দিয়ে Query Vector-এর কাছাকাছি Document Vector খোঁজা হবে।

### Common Similarity Metrics

**1. Cosine Similarity**

দুটি Vector-এর মধ্যকার Angle-এর Cosine দিয়ে Similarity পরিমাপ করে।

\[
\text{Cosine Similarity}(A,B)=
\frac{A\cdot B}{\|A\|\|B\|}
\]

সাধারণত Higher Score মানে বেশি Similarity। তবে Score-এর ব্যাখ্যা Model ও ব্যবহৃত Metric অনুযায়ী বুঝতে হবে।

**2. Euclidean Distance (L2)**

দুটি Vector-এর সরলরৈখিক দূরত্ব পরিমাপ করে।

\[
d(A,B)=\sqrt{\sum_{i=1}^{n}(A_i-B_i)^2}
\]

Lower Distance মানে Vector দুটি কাছাকাছি।

**3. Dot Product**

দুটি Vector-এর Corresponding Values গুণ করে যোগ করা হয়।

\[
A\cdot B=\sum_{i=1}^{n}A_iB_i
\]

কোন Metric ব্যবহার করবে তা Embedding Model, Vector Normalization এবং Database Configuration-এর ওপর নির্ভর করে।

### ANN কী?

ANN = Approximate Nearest Neighbor। বড় Dataset-এ দ্রুত কাছাকাছি Vector খুঁজতে Indexing Technique ব্যবহার করা হয়। এটি Search দ্রুত করতে পারে, তবে সবসময় mathematically exact nearest neighbor নিশ্চিত করে না।

## 7. ব্যবহারক্ষেত্র

1. **AI Customer Support:** Company Documentation থেকে Customer-এর প্রশ্নের উত্তর খোঁজা।
2. **Semantic Document Search:** PDF, Word, Policy ও Technical Documentation থেকে অর্থ অনুযায়ী তথ্য খোঁজা।
3. **Product Recommendation:** Similar Product Suggest করা।
4. **Image Search:** একটি Image-এর মতো Image খুঁজে পাওয়া।
5. **Personalized Learning:** Student-এর Learning History অনুযায়ী Course Suggest করা।
6. **Duplicate Detection:** একই বা কাছাকাছি অর্থের Content শনাক্ত করা।
7. **Fraud/Anomaly Detection:** স্বাভাবিক Pattern থেকে আলাদা Data শনাক্ত করতে সহায়তা করা।

## 8. RAG Architecture

**RAG = Retrieval-Augmented Generation**

RAG এমন একটি Architecture যেখানে LLM-কে উত্তর দেওয়ার আগে External Knowledge Base থেকে Relevant Information Retrieve করে Context হিসেবে দেওয়া হয়।

উদাহরণ: Madrasa Management System-এ Admission Policy, Student Rules, Notice এবং অন্যান্য Document থেকে AI Assistant উত্তর দেবে।

```text
                 USER QUESTION
                       |
                       v
              ASP.NET Core Web API
                       |
                       v
                Query Embedding
                       |
                       v
                Vector Database
                       |
                       v
              Relevant Documents
                       |
                       v
          Context + Original Question
                       |
                       v
                      LLM
                       |
                       v
                 AI Response
                       |
                       v
                    USER
```

RAG-এর তিনটি প্রধান ধাপ:

1. **Retrieval:** Relevant Information খুঁজে বের করা।
2. **Augmentation:** Retrieved Information-কে User Question-এর সঙ্গে Context হিসেবে যুক্ত করা।
3. **Generation:** LLM দিয়ে উত্তর তৈরি করা।

RAG LLM-এর Hallucination কমাতে সাহায্য করতে পারে, কিন্তু ভুল Retrieval, অসম্পূর্ণ Source বা ভুল Generation-এর কারণে ভুল উত্তর এখনও সম্ভব। তাই Source Citation, Evaluation এবং Permission Control জরুরি।

## 9. Data Ingestion ও Query Pipeline

একটি Professional RAG System-এ সাধারণত দুটি Pipeline থাকে।

### A. Data Ingestion Pipeline

```text
PDF / DOCX / TXT / Database
             |
             v
        Text Extraction
             |
             v
         Text Cleaning
             |
             v
          Chunking
             |
             v
     Embedding Generation
             |
             v
       Vector Database
```

ধাপগুলো:

1. Document সংগ্রহ।
2. Document থেকে Text Extract করা।
3. অপ্রয়োজনীয় Formatting পরিষ্কার করা।
4. Document-কে Chunk-এ ভাগ করা।
5. প্রতিটি Chunk-এর Embedding তৈরি করা।
6. Vector, Text এবং Metadata Store করা।

### B. User Query Pipeline

```text
User Question
      |
      v
Query Embedding
      |
      v
Vector / Hybrid Search
      |
      v
Metadata & Permission Filter
      |
      v
Top-K Relevant Chunks
      |
      v
Optional Reranking
      |
      v
Context Construction
      |
      v
LLM Generation
      |
      v
Answer + Sources
```

## 10. Chunking কী?

বড় Document-কে ছোট ছোট Text Segment-এ ভাগ করাকে Chunking বলে।

ধরো, একটি PDF-এ 100 পৃষ্ঠা আছে। পুরো PDF একসঙ্গে Embedding না করে ছোট অংশে ভাগ করলে Relevant Passage খুঁজে পাওয়া সহজ হয়।

```text
Original Document
       |
       v
     Chunking
       |
   +---+---+---+
   |   |   |   |
   v   v   v   v
  C1  C2  C3  C4
   |   |   |   |
   v   v   v   v
 Embeddings
       |
       v
   Vector DB
```

উদাহরণ:

```text
Chunk 1: Admission requirements and eligibility.
Chunk 2: Student registration process.
Chunk 3: Monthly tuition fee policy.
Chunk 4: Scholarship application procedure.
```

### Chunking Strategies

| Strategy | কাজ |
|---|---|
| Fixed-size | নির্দিষ্ট Token/Character অনুযায়ী ভাগ |
| Overlapping | আগের Chunk-এর কিছু অংশ পরের Chunk-এ রাখা |
| Semantic | অর্থ বা Topic অনুযায়ী ভাগ |
| Recursive | Paragraph, Sentence ইত্যাদির ভিত্তিতে ভাগ |
| Structure-aware | Heading, Page ও Section অনুযায়ী ভাগ |

শুরুতে 300–800 Tokens এবং 10–20% Overlap নিয়ে পরীক্ষা করা যেতে পারে। এগুলো Starting Point, Universal Rule নয়। Document Type, Retrieval Quality এবং Context Window অনুযায়ী Evaluate করে Chunk Size ঠিক করতে হবে।

প্রতিটি Chunk-এর সঙ্গে Source File, Page Number, Heading, Document ID, Tenant ID এবং Version-এর মতো Metadata রাখা উপকারী।

## 11. .NET Developer-এর Practical Roadmap

### Recommended Technology Stack

| Technology | Purpose |
|---|---|
| C# | Programming Language |
| ASP.NET Core Web API | Backend API |
| .NET 8+ | Application Platform |
| OpenAI Embeddings / Azure OpenAI | Embedding Generation |
| Qdrant | Vector Database |
| Microsoft.Extensions.AI | AI Abstractions |
| Microsoft.Extensions.VectorData | Vector Store Abstractions |
| SQL Server | Business Data ও Metadata |
| Angular 20 | Frontend |
| Redis | Optional Cache |
| RabbitMQ | Optional Background Processing |

### Step 1: Qdrant Local Development-এ চালানো

Docker Desktop চালু করে Terminal-এ:

```bash
docker run -d \
  --name qdrant \
  -p 6333:6333 \
  -p 6334:6334 \
  -v qdrant_storage:/qdrant/storage \
  qdrant/qdrant
```

Ports:

```text
6333 = HTTP API
6334 = gRPC API
```

Local Dashboard:

```text
http://localhost:6333/dashboard
```

> Local development-এর এই Basic Command-এ Authentication/TLS Configure করা নেই। Production-এ Publicly Expose করবে না। Authentication, TLS, Persistent Storage, Backup এবং Network Security Configure করবে।

### Step 2: Visual Studio-তে Project তৈরি

1. Visual Studio 2022 Open করো।
2. Create a new project নির্বাচন করো।
3. ASP.NET Core Web API নির্বাচন করো।
4. Project Name: `VectorSearch.Api`
5. Target Framework: `.NET 8` বা Compatible Newer Version।
6. Controllers Enable করো।

### Step 3: Suggested Folder Structure

```text
VectorSearch.Api/
│
├── Controllers/
│   └── SearchController.cs
│
├── Services/
│   ├── EmbeddingService.cs
│   ├── VectorSearchService.cs
│   └── DocumentIngestionService.cs
│
├── Models/
│   ├── Document.cs
│   ├── DocumentChunk.cs
│   └── SearchRequest.cs
│
├── Repositories/
│   └── IVectorRepository.cs
│
├── Configuration/
│   └── AIOptions.cs
│
├── Program.cs
└── appsettings.json
```

### Step 4: Packages

Microsoft-এর AI Abstractions এবং OpenAI Integration-এর জন্য:

```powershell
Install-Package Microsoft.Extensions.AI
Install-Package Microsoft.Extensions.AI.OpenAI
Install-Package Microsoft.Extensions.VectorData.Abstractions
```

Qdrant Provider ব্যবহার করলে তার Compatible Provider Package যোগ করতে হবে। Package Version এবং Provider API পরিবর্তিত হতে পারে, তাই Microsoft Learn ও Provider-এর Official Documentation অনুসরণ করবে।

### Step 5: Document Chunk Model

```csharp
public class DocumentChunk
{
    public Guid Id { get; set; }

    public string DocumentId { get; set; } = string.Empty;

    public string Content { get; set; } = string.Empty;

    public string Source { get; set; } = string.Empty;

    public int PageNumber { get; set; }

    public string TenantId { get; set; } = string.Empty;

    public float[] Embedding { get; set; }
        = Array.Empty<float>();
}
```

এখানে:

- `Id` = Unique Chunk ID
- `DocumentId` = Original Document ID
- `Content` = Original Text
- `Source` = File Name/Source Reference
- `PageNumber` = Document Page
- `TenantId` = Tenant Isolation
- `Embedding` = Numerical Vector

### Step 6: Embedding Generate করা

Microsoft.Extensions.AI-এর `IEmbeddingGenerator` Abstraction-এর ধারণাগত উদাহরণ:

```csharp
using Microsoft.Extensions.AI;

public class EmbeddingService
{
    private readonly IEmbeddingGenerator<
        string,
        Embedding<float>> _generator;

    public EmbeddingService(
        IEmbeddingGenerator<
            string,
            Embedding<float>> generator)
    {
        _generator = generator;
    }

    public async Task<ReadOnlyMemory<float>>
        GenerateAsync(string text)
    {
        var result =
            await _generator.GenerateAsync(text);

        return result.Vector;
    }
}
```

এটি দেখায় কীভাবে Abstraction ব্যবহার করা যায়। বাস্তবে DI Container-এ OpenAI/Azure OpenAI-এর Concrete Generator Register করতে হবে এবং API Key নিরাপদভাবে Configure করতে হবে।

### Step 7: Vector Record-এর ধারণা

Qdrant Point-এর একটি Simplified JSON উদাহরণ:

```json
{
  "id": "chunk-001",
  "vector": [0.12, 0.45, 0.78],
  "payload": {
    "documentId": "DOC-100",
    "content": "ASP.NET Core is used to build REST APIs.",
    "source": "dotnet-guide.pdf",
    "pageNumber": 10,
    "tenantId": "TENANT-01"
  }
}
```

> তিন Dimension-এর Vector এখানে শুধু বোঝানোর জন্য। বাস্তবে Collection-এর Vector Size অবশ্যই ব্যবহৃত Embedding Model-এর Dimension-এর সঙ্গে মিলতে হবে।

### Step 8: Vector Repository Abstraction

```csharp
public interface IVectorRepository
{
    Task<List<DocumentChunk>> SearchAsync(
        ReadOnlyMemory<float> queryVector,
        string tenantId,
        int topK,
        CancellationToken cancellationToken = default);
}
```

Repository-এর Implementation Qdrant SDK অথবা Microsoft Vector Data Provider দিয়ে তৈরি করতে হবে। Tenant Filter Database Query-এর মধ্যেই প্রয়োগ করতে হবে।

### Step 9: Search Service

```csharp
public class VectorSearchService
{
    private readonly EmbeddingService _embedding;
    private readonly IVectorRepository _repository;

    public VectorSearchService(
        EmbeddingService embedding,
        IVectorRepository repository)
    {
        _embedding = embedding;
        _repository = repository;
    }

    public async Task<List<DocumentChunk>> SearchAsync(
        string query,
        string tenantId,
        CancellationToken cancellationToken = default)
    {
        var vector = await _embedding.GenerateAsync(query);

        return await _repository.SearchAsync(
            vector,
            tenantId,
            topK: 5,
            cancellationToken);
    }
}
```

### Step 10: API Request Model

```csharp
public class SearchRequest
{
    public string Query { get; set; } = string.Empty;
}
```

### Step 11: API Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class SearchController : ControllerBase
{
    private readonly VectorSearchService _searchService;

    public SearchController(
        VectorSearchService searchService)
    {
        _searchService = searchService;
    }

    [HttpPost]
    public async Task<IActionResult> Search(
        SearchRequest request,
        CancellationToken cancellationToken)
    {
        if (string.IsNullOrWhiteSpace(request.Query))
            return BadRequest("Query is required.");

        // Tenant ID authenticated server-side claims
        // থেকে নিতে হবে; Client input বিশ্বাস করা যাবে না।
        var tenantId = User.FindFirst("tenant_id")?.Value;

        if (string.IsNullOrWhiteSpace(tenantId))
            return Forbid();

        var results = await _searchService.SearchAsync(
            request.Query,
            tenantId,
            cancellationToken);

        return Ok(results.Select(x => new
        {
            x.Content,
            x.Source,
            x.PageNumber
        }));
    }
}
```

### Step 12: Test API

Request:

```http
POST /api/search
Content-Type: application/json
Authorization: Bearer YOUR_ACCESS_TOKEN
```

```json
{
  "query": "How to develop REST API in .NET?"
}
```

সম্ভাব্য Response:

```json
{
  "results": [
    {
      "content": "ASP.NET Core is used to build REST APIs.",
      "source": "dotnet-guide.pdf",
      "pageNumber": 10
    }
  ]
}
```

এটি Expected Response-এর একটি ধারণাগত উদাহরণ; বাস্তব Response Repository Implementation ও Search Result-এর ওপর নির্ভর করবে।

> এই Section-এ Interface এবং Service Design দেখানো হয়েছে; এটি সম্পূর্ণ Copy-Paste Runnable Project নয়। Provider Registration, Qdrant Collection Creation, Upsert, Search এবং OpenAI Configuration-ও Implement করতে হবে।

## 12. Production-grade Design

Production Application-এ শুধু Embedding এবং Vector Database থাকলেই যথেষ্ট নয়।

### Suggested Architecture

```text
                Angular 20
                    |
                    v
             ASP.NET Core API
                    |
          +---------+---------+
          |                   |
          v                   v
   Authentication       Rate Limiting
          |
          v
      Application Layer
          |
    +-----+------------------+
    |                        |
    v                        v
SQL Server              RAG Services
Business Data                 |
                              v
                       Embedding Service
                              |
                              v
                         Qdrant
                              |
                              v
                      Relevant Chunks
                              |
                              v
                        Reranking
                              |
                              v
                             LLM
                              |
                              v
                       Answer + Sources
```

### Production Components

| Component | Responsibility |
|---|---|
| API Gateway | Routing, Rate Limiting |
| Authentication | JWT / OAuth 2.0 |
| Authorization | User ও Tenant Permission |
| Document Processor | PDF/DOCX/TXT Extraction |
| Message Queue | Background Processing |
| Embedding Service | Vector Generation |
| Vector Database | Vector Storage ও Retrieval |
| Metadata Database | Document ও Business Metadata |
| Reranker | Search Result পুনরায় Ranking |
| LLM | Final Answer Generation |
| Redis | Optional Cache |
| Logging | Structured Logs |
| Monitoring | Latency, Throughput, Token Usage |

### 12.1 Clean Architecture

Business Logic এবং External AI Provider আলাদা রাখবে।

```text
Domain
   |
Application
   |
Infrastructure
   |
API
```

Controller-এর মধ্যে সরাসরি OpenAI বা Qdrant SDK-এর Business Logic লিখবে না। Interface-এর মাধ্যমে Dependency Inversion অনুসরণ করবে।

### 12.2 Background Processing

বড় PDF Upload হলে API Request-এর মধ্যে সব Document Process না করে Background Worker ব্যবহার করো।

```text
Upload Document
       |
       v
Store Original File
       |
       v
Message Queue
       |
       v
Background Worker
       |
       v
Extract + Chunk
       |
       v
Generate Embeddings
       |
       v
Store in Vector DB
```

### 12.3 Multi-Tenant Security

একটি Company যেন অন্য Company-এর Document Search করতে না পারে।

প্রতিটি Vector Record-এ Tenant ID রাখবে এবং Authenticated User-এর Server-verified Tenant ID দিয়ে Search Filter করবে।

```json
{
  "tenantId": "TENANT-001",
  "documentId": "DOC-001",
  "content": "Company policy..."
}
```

শুধু Client-এর পাঠানো Tenant ID বিশ্বাস করবে না। Authorization এবং Filter Database Search-এর মধ্যেই প্রয়োগ করতে হবে।

### 12.4 Hybrid Search

শুধু Vector Search-এর ওপর নির্ভর না করে Keyword Search এবং Vector Search একসঙ্গে ব্যবহার করা যায়।

```text
             User Query
                 |
        +--------+--------+
        |                 |
        v                 v
   Keyword Search    Vector Search
        |                 |
        +--------+--------+
                 |
                 v
            Result Fusion
                 |
                 v
              Reranking
                 |
                 v
         Relevant Documents
```

Employee ID, Invoice Number, Product Code বা নির্দিষ্ট Technical Identifier-এর ক্ষেত্রে Exact Keyword Search গুরুত্বপূর্ণ।

### 12.5 Versioning

Embedding Model পরিবর্তন করলে Vector Dimension বা Semantic Space পরিবর্তিত হতে পারে।

তাই সংরক্ষণ করবে:

```text
EmbeddingModel
EmbeddingVersion
VectorDimension
ChunkingVersion
DocumentVersion
CreatedAt
```

Model পরিবর্তনের সময় Re-embedding এবং Re-indexing Strategy থাকতে হবে।

## 13. SQL Server বনাম Vector Database

| Feature | SQL Server | Dedicated Vector Database |
|---|---|---|
| Relational Data | Excellent | প্রধান উদ্দেশ্য নয় |
| JOIN | Yes | সীমিত/Provider-dependent |
| Transactions | Strong relational transactions | Provider-dependent |
| Exact Search | Yes | Metadata Filter |
| Semantic Search | Supported versions/configurations-এ Vector Search | Core Feature |
| Similarity Search | Vector capability | Core Feature |
| Business Data | Excellent | সাধারণত Search Index |

একটি বাস্তব Application-এ দুই ধরনের Storage একসঙ্গে ব্যবহার করা যায়:

```text
SQL Server
    |
    +-- Users
    +-- Employees
    +-- Payments
    +-- Documents Metadata
    +-- Permissions

Qdrant
    |
    +-- Document Embeddings
    +-- Chunk Content
    +-- Search Metadata
```

SQL Server হবে Business Data-এর Source of Truth। Vector Database হবে Semantic Retrieval-এর Search Index।

**গুরুত্বপূর্ণ:** Salary, Payment, Donation বা Account Balance-এর মতো Financial Data-এর জন্য শুধু Vector Search ব্যবহার করবে না। এগুলো Authorized SQL Query এবং Transaction-এর মাধ্যমে যাচাই করতে হবে।

## 14. Common Mistakes

| ভুল | সমাধান |
|---|---|
| পুরো PDF এক Chunk করা | Proper Chunking |
| প্রতিটি Request-এ সব Document Embed করা | Ingestion Pipeline |
| ভুল Dimension ব্যবহার | Model Dimension Validate |
| সব Tenant-এর Data একসঙ্গে Search | Mandatory Tenant Filter |
| শুধু Vector Search ব্যবহার | Hybrid Search বিবেচনা |
| API Key Code-এ রাখা | Secret Manager / Environment Secret |
| Retry নেই | Bounded Retry + Exponential Backoff |
| Duplicate Document Index করা | Idempotent Ingestion |
| Model Version Track না করা | Embedding Versioning |
| Search Quality Test না করা | Retrieval Evaluation |
| Monitoring নেই | Structured Logging + Metrics |

### Performance Optimization

- Batch Embedding Generation ব্যবহার করো।
- Frequently Used Query-এর জন্য Cache বিবেচনা করো।
- Appropriate ANN Index ব্যবহার করো।
- Top-K Result সীমিত রাখো।
- Metadata Filtering Optimize করো।
- API Timeout এবং CancellationToken ব্যবহার করো।
- Embedding API Rate Limit Handle করো।
- Document Update/Delete হলে পুরোনো Chunk Index পরিষ্কার করো।

### Evaluation Metrics

| Metric | কী পরিমাপ করে |
|---|---|
| Recall@K | Relevant Document Top-K-এর মধ্যে কতবার পাওয়া যায় |
| Precision@K | Top-K Result-এর কতগুলো Relevant |
| MRR | প্রথম Relevant Result কত উপরে এসেছে |
| NDCG | Ranking Quality |
| Faithfulness | Answer Retrieved Context-এর সঙ্গে কতটা সামঞ্জস্যপূর্ণ |
| Latency | Response দিতে কত সময় লাগে |
| Cost per Query | প্রতি Query-তে কত খরচ হচ্ছে |

## 15. Learning Roadmap

### Level 1 — Fundamentals

- AI ও Machine Learning Basics
- Embedding
- Vector এবং Dimension
- Cosine Similarity
- Semantic Search

### Level 2 — Vector Database

- Qdrant Installation
- Collection তৈরি
- Vector Insert / Update / Delete
- Similarity Search
- Metadata Filtering

### Level 3 — .NET Integration

- ASP.NET Core Web API
- OpenAI Embeddings
- Microsoft.Extensions.AI
- Microsoft.Extensions.VectorData
- Qdrant Integration

### Level 4 — RAG Application

- PDF Processing
- Text Chunking
- Embedding Pipeline
- Retrieval
- LLM Integration
- Source Citation

### Level 5 — Production Engineering

- Clean Architecture
- Background Worker
- RabbitMQ
- Redis
- Multi-Tenancy
- Hybrid Search
- Reranking
- Monitoring
- Security
- Evaluation

## 16. Interview Questions

**Q1. What is an Embedding?**

Embedding হলো Text, Image বা অন্য Data-এর Semantic Representation হিসেবে Numerical Vector তৈরি করার পদ্ধতি।

**Q2. What is a Vector Database?**

Vector Database হলো Embedding Vector Store, Index এবং Similarity Search করার জন্য ব্যবহৃত Database।

**Q3. What is Semantic Search?**

Semantic Search হলো Exact Keyword Matching-এর পরিবর্তে Meaning এবং Context অনুযায়ী Information খোঁজার পদ্ধতি।

**Q4. What is Cosine Similarity?**

দুটি Vector-এর মধ্যকার Angular Similarity পরিমাপ করার Mathematical Technique।

**Q5. What is RAG?**

Retrieval-Augmented Generation হলো এমন Architecture যেখানে External Knowledge Base থেকে Relevant Information Retrieve করে LLM-কে Context হিসেবে দেওয়া হয়।

**Q6. What is Chunking?**

বড় Document-কে ছোট ছোট Text Segment-এ ভাগ করার প্রক্রিয়া।

**Q7. What is Hybrid Search?**

Keyword Search এবং Vector Search একসঙ্গে ব্যবহার করে Relevant Information Retrieve করার পদ্ধতি।

**Q8. Does a Vector Database replace SQL Server?**

না। SQL Server Relational Business Data এবং Transaction-এর জন্য ভালো। Vector Database Semantic Search-এর জন্য বিশেষভাবে তৈরি।

**Q9. Why should indexing and querying use compatible embedding models?**

দুই Vector একই Compatible Vector Space-এ থাকা প্রয়োজন। Model পরিবর্তন করলে নতুন Model দিয়ে Re-embedding করতে হতে পারে।

**Q10. What is Reranking?**

প্রাথমিক Search Result-গুলোকে আরও উন্নত Relevance Model দিয়ে পুনরায় Score এবং Rank করা।

## 17. Official Resources

1. [Microsoft Learn — Vector Databases for .NET AI Apps](https://learn.microsoft.com/en-us/dotnet/ai/conceptual/vector-databases)
2. [Microsoft Learn — Build a .NET Vector Search App](https://learn.microsoft.com/en-us/dotnet/ai/vector-stores/how-to/build-vector-search-app)
3. [Microsoft Learn — Embeddings in .NET](https://learn.microsoft.com/en-us/dotnet/ai/conceptual/embeddings)
4. [OpenAI — Retrieval Guide](https://platform.openai.com/docs/guides/retrieval)
5. [Qdrant Documentation](https://qdrant.tech/documentation/)
6. [Qdrant .NET Client](https://github.com/qdrant/qdrant-dotnet)
7. [Pinecone Documentation](https://docs.pinecone.io/)
8. [Weaviate Documentation](https://docs.weaviate.io/)
9. [Milvus Documentation](https://milvus.io/docs)
10. [pgvector GitHub](https://github.com/pgvector/pgvector)

---

## Final Summary

| Concept | সহজ অর্থ |
|---|---|
| Embedding | Text/Data → Vector |
| Vector | Numerical Array |
| Dimension | Vector-এর Number of Values |
| Vector Database | Vector Storage + Search |
| Similarity Search | কাছাকাছি Meaning খোঁজা |
| Cosine Similarity | Vector-এর Angular Similarity |
| Chunking | Document ছোট অংশে ভাগ করা |
| Semantic Search | অর্থ অনুযায়ী Search |
| RAG | Search + Context + AI Answer |
| LLM | Human-readable Answer তৈরি |

```text
Embedding Model
      |
      v
Text -> Vector
      |
      v
Vector Database
      |
      v
Semantic Search
      |
      v
Relevant Context
      |
      v
LLM
      |
      v
AI Generated Answer
```

### Author

**Abdullah Hossain**

**Topics:** C# | ASP.NET Core | AI | Embeddings | Vector Database | Qdrant | Semantic Search | RAG | SQL Server | Clean Architecture

**Learning Goal:** Understanding how modern AI-powered applications retrieve, store and process information using Vector Embeddings and Vector Databases.
