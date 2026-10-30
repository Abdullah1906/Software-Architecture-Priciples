# 🔥 Advanced .NET Developer Interview Questions & Answers
### Part 2 — Real-world, Practical & Scenario-Based Interview Preparation

**Technology:** C# | ASP.NET Core 8/9 | Web API | ADO.NET | SQL Server | Angular 20 | JWT | Clean Architecture

**Level:** Intermediate → Advanced → Professional

---

# 📚 Table of Contents

1. Advanced C# — Q1–Q10
2. ASP.NET Core & Web API — Q11–Q25
3. ADO.NET & Database Architecture — Q26–Q35
4. SQL Server Advanced — Q36–Q50
5. Angular 20 Advanced — Q51–Q65
6. Authentication & Security — Q66–Q75
7. Architecture & Design Patterns — Q76–Q85
8. Performance, Testing & DevOps — Q86–Q95
9. Real-world Project Scenarios — Q96–Q100

---

# 1. Advanced C# Interview Questions

### Q1. What is the difference between Task.WhenAll() and Task.WhenAny()?

**Answer:**

দুটিই একাধিক asynchronous task একসঙ্গে manage করার জন্য ব্যবহৃত হয়।

**Task.WhenAll():**
সবগুলো task complete হওয়া পর্যন্ত অপেক্ষা করে।

**Task.WhenAny():**
যেকোনো একটি task complete হলেই return করে।

```csharp
Task<string> task1 = GetUserAsync();
Task<string> task2 = GetOrderAsync();

var results = await Task.WhenAll(task1, task2);

var firstCompleted = await Task.WhenAny(task1, task2);
```

**Practical Use:**

একসঙ্গে User এবং Order data load করতে `WhenAll()` ব্যবহার করতে পারি।

### Q2. What is CancellationToken?

**Answer:**

CancellationToken asynchronous operation cancel করার request communicate করে।

ASP.NET Core-এ client request cancel করলে database operation-ও cancel করার চেষ্টা করা যায়।

```csharp
[HttpGet]
public async Task<IActionResult> GetEmployees(
    CancellationToken cancellationToken)
{
    var result = await _service.GetEmployeesAsync(
        cancellationToken);

    return Ok(result);
}
```

Service:

```csharp
public async Task<List<Employee>> GetEmployeesAsync(
    CancellationToken cancellationToken)
{
    return await _repository.GetAllAsync(cancellationToken);
}
```

### Q3. What is ConfigureAwait(false)?

**Answer:**

`ConfigureAwait(false)` await-এর পরে আগের Synchronization Context-এ ফিরে আসার চেষ্টা না করার নির্দেশ দেয়।

Library code-এ এটি ব্যবহার করা হতে পারে।

ASP.NET Core সাধারণত classic ASP.NET-এর মতো Synchronization Context ব্যবহার করে না। তাই সাধারণ application code-এ এটি বাধ্যতামূলক নয়।

```csharp
await SomeOperationAsync().ConfigureAwait(false);
```

### Q4. What is Span<T> and Memory<T>?

**Answer:**

দুটিই memory-efficient data processing-এর জন্য ব্যবহৃত হয়।

**Span<T>:**
- Contiguous memory region represent করে।
- Stack-only ref struct।
- Heap-এ রাখা যায় না।
- সাধারণভাবে async method-এর মধ্যে await boundary অতিক্রম করতে পারে না।

**Memory<T>:**
- Managed memory region represent করে।
- Heap-এ রাখা যায়।
- Asynchronous workflow-এ ব্যবহার করা যায়।

### Q5. What is Record in C#?

**Answer:**

Record হলো data-centric type, যা value-based equality support করে।

DTO এবং immutable data model তৈরিতে এটি useful।

```csharp
public record EmployeeDto(
    int Id,
    string Name,
    decimal Salary
);

var employee1 = new EmployeeDto(1, "Rahim", 30000);
var employee2 = new EmployeeDto(1, "Rahim", 30000);

Console.WriteLine(employee1 == employee2); // True
```

### Q6. What is the difference between Class and Struct?

**Answer:**

| Class | Struct |
|---|---|
| Reference type | Value type |
| Heap allocation commonly | Inline storage possible |
| Supports inheritance | Cannot inherit another struct/class |
| Can be null | Non-nullable by default |
| Suitable for complex objects | Suitable for small value-like objects |

### Q7. What is Expression Tree?

**Answer:**

Expression Tree হলো code-এর structure-কে data structure হিসেবে represent করার পদ্ধতি।

ORM এবং LINQ providers এটি ব্যবহার করে query translate করতে পারে।

```csharp
Expression<Func<Employee, bool>> expression =
    e => e.Salary > 50000;
```

### Q8. What is yield return?

**Answer:**

`yield return` iterator তৈরি করে এবং একবারে সব data memory-তে load না করে lazily values return করতে পারে।

```csharp
public IEnumerable<int> GetNumbers()
{
    for (int i = 1; i <= 5; i++)
    {
        yield return i;
    }
}
```

### Q9. What is Thread Safety?

**Answer:**

একাধিক thread একই resource access করলেও data corruption বা unexpected behavior না হওয়াকে Thread Safety বলে।

সমাধান:

- lock
- SemaphoreSlim
- ConcurrentDictionary
- Immutable Collections
- Interlocked

```csharp
private readonly object _lock = new();

private int _count;

public void Increment()
{
    lock (_lock)
    {
        _count++;
    }
}
```

### Q10. What is the difference between lock and SemaphoreSlim?

**Answer:**

`lock` একই সময়ে একটি thread-কে critical section-এ প্রবেশ করতে দেয়।

`SemaphoreSlim` নির্দিষ্ট সংখ্যক concurrent operations allow করে এবং asynchronous wait support করে।

```csharp
private readonly SemaphoreSlim _semaphore = new(3, 3);

public async Task ProcessAsync()
{
    await _semaphore.WaitAsync();

    try
    {
        await DoWorkAsync();
    }
    finally
    {
        _semaphore.Release();
    }
}
```

---

# 2. ASP.NET Core & Web API Advanced

### Q11. Explain ASP.NET Core Request Pipeline.

**Answer:**

Client থেকে request আসার পর middleware pipeline-এর মধ্য দিয়ে request controller বা endpoint-এ যায়।

Example:

```text
Client
  |
  v
Exception Middleware
  |
  v
HTTPS Redirection
  |
  v
CORS
  |
  v
Authentication
  |
  v
Authorization
  |
  v
Routing / Endpoint
  |
  v
Controller
  |
  v
Service
  |
  v
Repository
  |
  v
SQL Server
```

Middleware registration-এর order গুরুত্বপূর্ণ।

### Q12. Why should Authentication come before Authorization?

**Answer:**

Authentication প্রথমে user identity verify করে এবং `HttpContext.User` তৈরি করে।

Authorization এরপর সেই identity ও claims ব্যবহার করে access permission নির্ধারণ করে।

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

### Q13. What is Global Exception Handling?

**Answer:**

পুরো application-এর exception centrally handle করাকে Global Exception Handling বলে।

এতে প্রতিটি controller-এ আলাদা try-catch লেখার প্রয়োজন কমে।

```csharp
builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler();
```

Production application-এ structured error response এবং server-side logging রাখতে হবে।

### Q14. What is Action Filter?

**Answer:**

Action Filter controller action execute হওয়ার আগে বা পরে logic execute করতে পারে।

Use cases:

- Logging
- Validation
- Auditing
- Custom Action Processing

```csharp
public class LogActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        Console.WriteLine("Before Action");
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        Console.WriteLine("After Action");
    }
}
```

### Q15. What is Minimal API?

**Answer:**

Minimal API হলো কম boilerplate code দিয়ে HTTP endpoint তৈরি করার approach।

```csharp
var builder = WebApplication.CreateBuilder(args);

var app = builder.Build();

app.MapGet("/employees", () =>
{
    return Results.Ok("Employee List");
});

app.Run();
```

Small services এবং lightweight APIs-এর জন্য useful।

### Q16. What is IHostedService?

**Answer:**

Background service application startup এবং shutdown lifecycle-এর সঙ্গে কাজ করে।

Use cases:

- Scheduled jobs
- Queue processing
- Background tasks
- Periodic cleanup

```csharp
public class WorkerService : BackgroundService
{
    protected override async Task ExecuteAsync(
        CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            Console.WriteLine("Background task running");

            await Task.Delay(
                TimeSpan.FromMinutes(1),
                stoppingToken);
        }
    }
}
```

### Q17. What is the difference between IHostedService and BackgroundService?

**Answer:**

`IHostedService` একটি interface, যেখানে StartAsync এবং StopAsync implement করতে হয়।

`BackgroundService` হলো IHostedService-এর abstract base class, যা long-running background work সহজ করে।

### Q18. What is Health Check in ASP.NET Core?

**Answer:**

Application এবং dependent services healthy আছে কি না তা monitor করার জন্য Health Check ব্যবহার হয়।

```csharp
builder.Services.AddHealthChecks();

var app = builder.Build();

app.MapHealthChecks("/health");
```

Database health check-এর জন্য SQL Server connectivity-ও পরীক্ষা করা যায়।

### Q19. What is IHttpClientFactory?

**Answer:**

External API call করার জন্য HttpClient manage করার recommended approach।

এটি handler pooling এবং connection management সহজ করে।

```csharp
builder.Services.AddHttpClient<PaymentService>();
```

Usage:

```csharp
public class PaymentService
{
    private readonly HttpClient _httpClient;

    public PaymentService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<string> GetPaymentAsync()
    {
        return await _httpClient.GetStringAsync("/payments");
    }
}
```

### Q20. What is Content Negotiation?

**Answer:**

Client কোন response format চায়, যেমন JSON বা XML, তা request header-এর মাধ্যমে communicate করতে পারে।

```http
Accept: application/json
```

ASP.NET Core configured output formatters অনুযায়ী response format select করতে পারে।

### Q21. What is Idempotency in REST API?

**Answer:**

একই request একাধিকবার execute হলেও server-এর intended final state একই থাকলে সেটিকে Idempotent operation বলে।

HTTP examples:

| Method | Idempotent |
|---|---|
| GET | Yes |
| PUT | Yes |
| DELETE | Yes |
| POST | Not generally |
| PATCH | Not guaranteed |

Payment API-তে Idempotency Key ব্যবহার করে duplicate payment prevent করা যায়।

### Q22. What is API Rate Limiting in ASP.NET Core?

**Answer:**

একটি নির্দিষ্ট সময়ের মধ্যে request সংখ্যা limit করা।

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("api", limiter =>
    {
        limiter.PermitLimit = 10;
        limiter.Window = TimeSpan.FromMinutes(1);
        limiter.QueueLimit = 0;
    });
});
```

```csharp
app.UseRateLimiter();

app.MapControllers().RequireRateLimiting("api");
```

### Q23. What is Response Caching vs Output Caching?

**Answer:**

**Response Caching:**
HTTP caching semantics এবং client/proxy caching header-এর উপর নির্ভর করে।

**Output Caching:**
Server-side response caching policy অনুযায়ী response cache করে।

ASP.NET Core:

```csharp
builder.Services.AddOutputCache();

var app = builder.Build();

app.UseOutputCache();
```

Sensitive user-specific data cache করার সময় user isolation নিশ্চিত করতে হবে।

### Q24. What is ProblemDetails?

**Answer:**

ProblemDetails হলো HTTP API error response-এর standardized format।

Example:

```json
{
  "type": "https://example.com/errors/validation",
  "title": "Validation Failed",
  "status": 400,
  "detail": "Employee name is required."
}
```

### Q25. What is OpenAPI Specification?

**Answer:**

OpenAPI হলো REST API document করার standard specification।

এতে endpoint, request, response, parameters এবং authentication schemes describe করা যায়।

Swagger UI এই specification ব্যবহার করে API documentation দেখাতে পারে।

---

# 3. ADO.NET & Database Architecture

### Q26. Why use ADO.NET instead of Entity Framework Core?

**Answer:**

ADO.NET সরাসরি SQL Server-এর সঙ্গে কাজ করার সুযোগ দেয়।

Advantages:

- Full SQL control
- Stored Procedure-based architecture
- Less ORM abstraction
- Fine-grained query optimization
- Lightweight data access

Disadvantages:

- More boilerplate code
- Manual mapping
- Manual transaction management
- More responsibility for query maintenance

### Q27. What is the difference between ExecuteReader and DataAdapter?

**Answer:**

ExecuteReader connected, forward-only reading-এর জন্য useful।

DataAdapter disconnected model-এ DataTable বা DataSet populate করে।

Large sequential data reading-এর ক্ষেত্রে DataReader সাধারণত বেশি memory-efficient।

### Q28. How do you implement Repository Pattern using ADO.NET?

**Answer:**

প্রথমে Interface:

```csharp
public interface IEmployeeRepository
{
    Task<Employee?> GetByIdAsync(int id);
}
```

Implementation:

```csharp
public class EmployeeRepository : IEmployeeRepository
{
    private readonly string _connectionString;

    public EmployeeRepository(IConfiguration configuration)
    {
        _connectionString =
            configuration.GetConnectionString("DefaultConnection")!;
    }

    public async Task<Employee?> GetByIdAsync(int id)
    {
        using var connection =
            new SqlConnection(_connectionString);

        using var command =
            new SqlCommand("sp_GetEmployeeById", connection);

        command.CommandType = CommandType.StoredProcedure;

        command.Parameters.Add("@EmployeeId", SqlDbType.Int).Value = id;

        await connection.OpenAsync();

        using var reader = await command.ExecuteReaderAsync();

        if (!await reader.ReadAsync())
            return null;

        return new Employee
        {
            EmployeeId = reader.GetInt32(
                reader.GetOrdinal("EmployeeId")),

            Name = reader.GetString(
                reader.GetOrdinal("Name"))
        };
    }
}
```

### Q29. What is CommandType.StoredProcedure?

**Answer:**

ADO.NET-কে জানায় যে command text একটি Stored Procedure-এর নাম।

```csharp
command.CommandType = CommandType.StoredProcedure;
command.CommandText = "sp_GetEmployees";
```

### Q30. How do you handle NULL values from SQL Server?

**Answer:**

SQL NULL value handle করতে `DBNull.Value` এবং `IsDBNull()` ব্যবহার করা হয়।

```csharp
int salaryOrdinal = reader.GetOrdinal("Salary");

decimal? salary = reader.IsDBNull(salaryOrdinal)
    ? null
    : reader.GetDecimal(salaryOrdinal);
```

### Q31. What is CommandTimeout?

**Answer:**

SQL command কত সময় পর্যন্ত execute হওয়ার জন্য অপেক্ষা করবে তা নির্ধারণ করে।

```csharp
command.CommandTimeout = 60;
```

Default timeout সাধারণত 30 seconds।

### Q32. What is Table-Valued Parameter (TVP)?

**Answer:**

একাধিক row একসঙ্গে Stored Procedure-এ পাঠানোর জন্য TVP ব্যবহার করা হয়।

SQL:

```sql
CREATE TYPE dbo.EmployeeIdList AS TABLE
(
    EmployeeId INT
);
```

Procedure:

```sql
CREATE PROCEDURE sp_GetEmployeesByIds
    @Ids dbo.EmployeeIdList READONLY
AS
BEGIN
    SELECT e.*
    FROM Employees e
    INNER JOIN @Ids i
        ON e.EmployeeId = i.EmployeeId;
END;
```

C#:

```csharp
var table = new DataTable();

table.Columns.Add("EmployeeId", typeof(int));

table.Rows.Add(1);
table.Rows.Add(2);
table.Rows.Add(3);

using var command = new SqlCommand(
    "sp_GetEmployeesByIds", connection);

command.CommandType = CommandType.StoredProcedure;

var parameter = command.Parameters.AddWithValue("@Ids", table);

parameter.SqlDbType = SqlDbType.Structured;
parameter.TypeName = "dbo.EmployeeIdList";
```

### Q33. How do you handle multiple result sets?

**Answer:**

`NextResultAsync()` ব্যবহার করে একাধিক result set read করা যায়।

```csharp
using var reader = await command.ExecuteReaderAsync();

while (await reader.ReadAsync())
{
    // First result set
}

await reader.NextResultAsync();

while (await reader.ReadAsync())
{
    // Second result set
}
```

### Q34. What is Transaction Isolation Level?

**Answer:**

একটি transaction অন্য transaction-এর uncommitted বা concurrent changes কীভাবে দেখতে পারবে তা নিয়ন্ত্রণ করে।

SQL Server-এর গুরুত্বপূর্ণ Isolation Levels:

- Read Uncommitted
- Read Committed
- Repeatable Read
- Snapshot
- Serializable

### Q35. What is Optimistic Concurrency Control?

**Answer:**

একাধিক user একই data update করার সময় conflict detect করার পদ্ধতি হলো Optimistic Concurrency Control।

SQL Server-এর `rowversion` ব্যবহার করা যায়।

```sql
CREATE TABLE Employees
(
    EmployeeId INT PRIMARY KEY,
    Name NVARCHAR(100),
    RowVer ROWVERSION
);
```

Update:

```sql
UPDATE Employees
SET Name = @Name
WHERE EmployeeId = @EmployeeId
AND RowVer = @OldRowVer;
```

যদি affected rows 0 হয়, তাহলে অন্য user data পরিবর্তন করে থাকতে পারে।

---

# 4. SQL Server Advanced

### Q36. What is SARGable Query?

**Answer:**

SARGable মানে Search ARGument ABLE।

যে query condition index ব্যবহার করে efficient data retrieval করতে সাহায্য করে তাকে SARGable query বলে।

Bad:

```sql
SELECT *
FROM Employees
WHERE YEAR(JoiningDate) = 2026;
```

Better:

```sql
SELECT *
FROM Employees
WHERE JoiningDate >= '20260101'
AND JoiningDate < '20270101';
```

### Q37. What is Index Seek vs Index Scan?

**Index Seek:**

Index structure ব্যবহার করে নির্দিষ্ট matching rows খুঁজে বের করে।

**Index Scan:**

Index-এর অনেক বা সব অংশ scan করে।

Seek সবসময় Scan-এর চেয়ে ভালো নয়। Data distribution এবং query অনুযায়ী সিদ্ধান্ত নিতে হয়।

### Q38. What is Covering Index?

**Answer:**

যে index একটি query-এর প্রয়োজনীয় search এবং output columns সরবরাহ করতে পারে তাকে Covering Index বলে।

```sql
CREATE NONCLUSTERED INDEX IX_Employees_Department
ON Employees(DepartmentId)
INCLUDE(Name, Salary);
```

### Q39. What is Composite Index?

**Answer:**

একাধিক column নিয়ে তৈরি index হলো Composite Index।

```sql
CREATE INDEX IX_Employee_Department_Salary
ON Employees(DepartmentId, Salary);
```

Column order গুরুত্বপূর্ণ।

### Q40. What is Parameter Sniffing?

**Answer:**

SQL Server Stored Procedure compile করার সময় parameter value দেখে execution plan তৈরি করতে পারে।

কোনো নির্দিষ্ট parameter-এর জন্য তৈরি plan অন্য parameter-এর ক্ষেত্রে inefficient হতে পারে। এটিকে Parameter Sniffing সমস্যা বলা হয়।

সম্ভাব্য সমাধান:

- Query rewrite
- Statistics maintenance
- Query Store analysis
- OPTION (RECOMPILE), যখন উপযুক্ত
- OPTIMIZE FOR, যখন প্রয়োজন

### Q41. What is Query Store?

**Answer:**

Query Store SQL Server-এর query execution history, plans এবং runtime statistics সংরক্ষণ করে।

এটি performance regression identify করতে সাহায্য করে।

### Q42. What is Window Function?

**Answer:**

একাধিক row-এর উপর calculation করতে পারে, কিন্তু প্রতিটি row-এর আলাদা result বজায় রাখে।

```sql
SELECT
    EmployeeId,
    Name,
    Salary,
    ROW_NUMBER() OVER (
        ORDER BY Salary DESC
    ) AS RowNumber,
    RANK() OVER (
        ORDER BY Salary DESC
    ) AS SalaryRank,
    DENSE_RANK() OVER (
        ORDER BY Salary DESC
    ) AS DenseSalaryRank
FROM Employees;
```

### Q43. Difference between ROW_NUMBER, RANK and DENSE_RANK?

**ROW_NUMBER:**
প্রতিটি row-কে unique sequential number দেয়।

**RANK:**
Tie থাকলে একই rank দেয় এবং পরবর্তী rank skip করে।

**DENSE_RANK:**
Tie থাকলেও পরবর্তী rank skip করে না।

### Q44. What is PIVOT?

**Answer:**

Rows-কে columns-এ convert করতে PIVOT ব্যবহার করা হয়।

```sql
SELECT *
FROM
(
    SELECT DepartmentId, Salary
    FROM Employees
) AS SourceTable
PIVOT
(
    SUM(Salary)
    FOR DepartmentId IN ([1], [2], [3])
) AS PivotTable;
```

### Q45. What is MERGE Statement?

**Answer:**

একটি target table-এর সঙ্গে source data compare করে INSERT, UPDATE বা DELETE করার জন্য MERGE ব্যবহার করা যায়।

তবে concurrency এবং complex matching scenario-তে আলাদা INSERT/UPDATE statements অধিক স্পষ্ট ও নিরাপদ হতে পারে।

### Q46. What is TRY...CATCH in SQL Server?

**Answer:**

SQL Server-এ exception handling করার জন্য TRY...CATCH ব্যবহার হয়।

```sql
BEGIN TRY

    BEGIN TRANSACTION;

    UPDATE Accounts
    SET Balance = Balance - 1000
    WHERE AccountId = 1;

    COMMIT TRANSACTION;

END TRY
BEGIN CATCH

    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;

    THROW;

END CATCH;
```

### Q47. What is Isolation Level Snapshot?

**Answer:**

Snapshot Isolation row versioning ব্যবহার করে transaction শুরু হওয়ার সময়কার consistent snapshot দেখায়।

এটি অনেক ক্ষেত্রে reader-writer blocking কমাতে সাহায্য করে।

Database-এ ALLOW_SNAPSHOT_ISOLATION enable করতে হয়।

### Q48. What is Temp Table vs Table Variable?

| Temp Table | Table Variable |
|---|---|
| #Temp | @Temp |
| TempDB ব্যবহার করে | TempDB-backed storage |
| Statistics support | Statistics behavior more limited |
| Indexes তৈরি করা যায় | Index limitations রয়েছে |
| Larger/intermediate workloads-এ useful | Small datasets-এ useful হতে পারে |

### Q49. What is Database Normalization vs Denormalization?

**Normalization:**

Data redundancy কমায় এবং consistency উন্নত করে।

**Denormalization:**

Read performance বা reporting সহজ করার জন্য controlled redundancy introduce করে।

### Q50. How do you find missing indexes?

**Answer:**

সম্ভাব্য missing index information DMV এবং actual execution plan থেকে দেখা যায়।

```sql
SELECT
    mid.statement,
    migs.avg_user_impact,
    migs.user_seeks,
    mid.equality_columns,
    mid.inequality_columns,
    mid.included_columns
FROM sys.dm_db_missing_index_details mid
INNER JOIN sys.dm_db_missing_index_groups mig
    ON mid.index_handle = mig.index_handle
INNER JOIN sys.dm_db_missing_index_group_stats migs
    ON mig.index_group_handle = migs.group_handle;
```

এগুলো suggestion মাত্র। সব missing index blindly create করা উচিত নয়।

---

# 5. Angular 20 Advanced

### Q51. What is the difference between signal() and computed()?

**Answer:**

`signal()` writable reactive state তৈরি করে।

`computed()` অন্য signal-এর উপর নির্ভর করে derived state তৈরি করে।

```typescript
count = signal(10);

doubleCount = computed(() => this.count() * 2);
```

### Q52. What is effect() in Angular Signals?

**Answer:**

Signal-এর value পরিবর্তন হলে side effect execute করার জন্য `effect()` ব্যবহার করা হয়।

```typescript
constructor() {
  effect(() => {
    console.log('Count:', this.count());
  });
}
```

Computed state তৈরির জন্য effect নয়, computed ব্যবহার করাই ভালো।

### Q53. What is toSignal()?

**Answer:**

Observable-কে Signal-এ convert করার জন্য `toSignal()` ব্যবহার হয়।

```typescript
import { toSignal } from '@angular/core/rxjs-interop';

employees = toSignal(
  this.employeeService.getEmployees(),
  { initialValue: [] }
);
```

### Q54. What is toObservable()?

**Answer:**

Signal-কে Observable-এ convert করে RxJS ecosystem-এর সঙ্গে integrate করা যায়।

```typescript
import { toObservable } from '@angular/core/rxjs-interop';

count$ = toObservable(this.count);
```

### Q55. What is switchMap()?

**Answer:**

একটি নতুন source value আসলে আগের inner subscription unsubscribe করে নতুন Observable subscribe করে।

Search API-এর জন্য useful।

```typescript
this.searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(value =>
    this.employeeService.search(value)
  )
);
```

### Q56. Difference between switchMap, mergeMap, concatMap and exhaustMap?

| Operator | Behavior |
|---|---|
| switchMap | Previous inner subscription unsubscribe |
| mergeMap | Concurrent inner subscriptions |
| concatMap | Sequential execution |
| exhaustMap | Current inner completes না হওয়া পর্যন্ত নতুন values ignore |

**Practical:**

- Search: switchMap
- Independent parallel requests: mergeMap
- Ordered save operations: concatMap
- Prevent duplicate login submissions: exhaustMap

### Q57. What is debounceTime()?

**Answer:**

একটি নির্দিষ্ট সময় নতুন value না এলে সর্বশেষ value emit করে।

Search input-এর API calls কমাতে ব্যবহৃত হয়।

```typescript
searchControl.valueChanges.pipe(
  debounceTime(500)
);
```

### Q58. What is takeUntilDestroyed()?

**Answer:**

Component বা service destroy হলে Observable subscription automatically cleanup করতে সাহায্য করে।

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

this.employeeService.getEmployees()
  .pipe(takeUntilDestroyed())
  .subscribe(data => {
    console.log(data);
  });
```

### Q59. What is CanMatch vs CanActivate?

**CanActivate:**

Route activate হওয়ার আগে check করে।

**CanMatch:**

Route matching-এর সময় check করে, ফলে অন্য matching route try করা সম্ভব।

### Q60. What is Functional Interceptor?

**Answer:**

Angular-এ function-based interceptor হলো functional interceptor।

```typescript
import { HttpInterceptorFn } from '@angular/common/http';

export const authInterceptor: HttpInterceptorFn =
  (req, next) => {

    const token = localStorage.getItem('access_token');

    const authReq = token
      ? req.clone({
          setHeaders: {
            Authorization: `Bearer ${token}`
          }
        })
      : req;

    return next(authReq);
  };
```

Sensitive token storage-এর ক্ষেত্রে XSS threat বিবেচনা করতে হবে।

### Q61. What is Angular Control Flow?

**Answer:**

Angular built-in control flow template syntax ব্যবহার করে conditional এবং loop rendering করা যায়।

```html
@if (isLoggedIn()) {
  <h2>Welcome</h2>
} @else {
  <h2>Please Login</h2>
}
```

Loop:

```html
@for (employee of employees(); track employee.id) {
  <p>{{ employee.name }}</p>
} @empty {
  <p>No employees found</p>
}
```

### Q62. What is @defer?

**Answer:**

Angular-এর `@defer` block ব্যবহার করে নির্দিষ্ট template content প্রয়োজন অনুযায়ী lazy render করা যায়।

```html
@defer (on viewport) {
  <app-heavy-chart />
} @placeholder {
  <p>Chart will load here</p>
} @loading {
  <p>Loading...</p>
}
```

### Q63. What is Angular Hydration?

**Answer:**

Server-Side Rendered HTML-কে browser-এ Angular application-এর সঙ্গে attach করার process হলো Hydration।

এতে server-rendered content পুনরায় সম্পূর্ণ render না করেও interactive করা যায়।

### Q64. What is ChangeDetectionStrategy.OnPush?

**Answer:**

OnPush strategy component change detection-এর কাজ সীমিত করতে সাহায্য করে।

এটি signals, input changes, events এবং অন্যান্য supported triggers-এর সঙ্গে কাজ করে।

```typescript
@Component({
  selector: 'app-employee',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `...`
})
export class EmployeeComponent {}
```

### Q65. How do you manage state in Angular?

**Answer:**

Application-এর complexity অনুযায়ী state management approach নির্বাচন করি।

Small application:

- Component state
- Signals
- Services

Medium application:

- Shared Services
- Signals
- RxJS

Large application:

- NgRx Store
- NgRx Effects
- Entity State
- Signal Store

সব project-এ NgRx বাধ্যতামূলক নয়।

---

# 6. Authentication & Security Advanced

### Q66. Where should JWT be stored in Angular?

**Answer:**

এটি security trade-off-এর বিষয়।

**LocalStorage:**
- সহজে access করা যায়।
- JavaScript access করতে পারে।
- XSS হলে token theft-এর ঝুঁকি থাকে।

**HttpOnly Secure Cookie:**
- JavaScript থেকে সরাসরি access করা যায় না।
- Secure এবং SameSite configuration প্রয়োজন।
- Cookie-based authentication-এ CSRF protection বিবেচনা করতে হয়।

High-security application-এ token lifecycle, XSS, CSRF এবং deployment architecture বিবেচনা করে সিদ্ধান্ত নিতে হয়।

### Q67. What is JWT Signature?

**Answer:**

JWT-এর signature token integrity এবং authenticity verify করতে সাহায্য করে।

Payload Base64Url encoded, encrypted নয়। তাই JWT payload-এ password বা secret রাখা যাবে না।

### Q68. What is Token Revocation?

**Answer:**

কোনো token expire হওয়ার আগেই invalid করার process।

Possible solutions:

- Refresh token revocation
- Token denylist
- Token versioning
- Short-lived access token

### Q69. What is Multi-Tenancy?

**Answer:**

একই application infrastructure ব্যবহার করে একাধিক organization বা tenant-এর data আলাদাভাবে manage করাকে Multi-Tenancy বলে।

Example:

```text
Donation Platform
   |
   |-- Madrasa A
   |-- Madrasa B
   |-- Madrasa C
```

প্রতিটি tenant শুধুমাত্র নিজের authorized data access করতে পারবে।

### Q70. How do you implement Tenant Isolation?

**Answer:**

1. প্রতিটি tenant-এর TenantId থাকবে।
2. User-এর সঙ্গে TenantId associate থাকবে।
3. Server-side trusted claims থেকে tenant identify করতে হবে।
4. Repository query-তে tenant filter apply করতে হবে।
5. Authorization policy enforce করতে হবে।
6. Cross-tenant access tests লিখতে হবে।

```sql
SELECT *
FROM Donations
WHERE TenantId = @TenantId;
```

Client-এর পাঠানো TenantId blindly trust করা যাবে না।

### Q71. What is API Key Authentication?

**Answer:**

API Key হলো application বা client identify করার জন্য ব্যবহৃত credential।

Use cases:

- Third-party integrations
- Server-to-server communication
- Internal services

API key-তে hashing, rotation, expiry এবং permission scope ব্যবহার করা উচিত।

### Q72. What is Refresh Token Reuse Detection?

**Answer:**

একবার ব্যবহার করা refresh token আবার ব্যবহার করা হলে সেটি suspicious activity হিসেবে detect করা।

সম্ভাব্য response:

- Token family revoke
- Session terminate
- Security event log
- Re-authentication required

### Q73. What is CSRF Token?

**Answer:**

CSRF Token হলো request-এর সঙ্গে পাঠানো একটি unpredictable token, যা server verify করে।

Cookie-based authentication-এ এটি গুরুত্বপূর্ণ protection mechanism।

### Q74. What is CORS Preflight Request?

**Answer:**

Browser cross-origin request পাঠানোর আগে প্রয়োজন হলে OPTIONS request পাঠায়।

এটিকে Preflight Request বলে।

Example:

```http
OPTIONS /api/employees
Origin: http://localhost:4200
Access-Control-Request-Method: POST
```

Server permitted origin, method এবং headers জানায়।

### Q75. What is Defense in Depth?

**Answer:**

একটি security layer-এর উপর নির্ভর না করে একাধিক security layer ব্যবহার করাকে Defense in Depth বলে।

Example:

- HTTPS
- Authentication
- Authorization
- Input Validation
- SQL Parameterization
- Rate Limiting
- Audit Logging
- Secure Secrets

---

# 7. Architecture & Design Patterns

### Q76. What is the difference between Clean Architecture and N-Tier Architecture?

**Answer:**

N-Tier architecture application-কে logical বা physical layers-এ ভাগ করে।

Clean Architecture dependency direction এবং business rules-এর independence-এর উপর গুরুত্ব দেয়।

Clean Architecture-এ সাধারণত outer layer inner layer-এর উপর নির্ভর করে।

### Q77. What is Vertical Slice Architecture?

**Answer:**

Feature বা use case অনুযায়ী code organize করার architectural approach।

Example:

```text
Features/
  Employees/
    Create/
    GetById/
    Update/
    Delete/
  Donations/
    Create/
    GetById/
```

প্রতিটি feature-এর request, handler, validator এবং অন্যান্য প্রয়োজনীয় code এক জায়গায় রাখা যায়।

### Q78. What is Mediator Pattern?

**Answer:**

একাধিক component-এর direct communication কমিয়ে একটি mediator-এর মাধ্যমে communication coordinate করা।

MediatR-এর মতো library দিয়ে CQRS implementation করা যায়।

### Q79. What is Strategy Pattern?

**Answer:**

একই কাজের একাধিক algorithm encapsulate করে runtime-এ প্রয়োজন অনুযায়ী select করা।

Payment example:

```csharp
public interface IPaymentStrategy
{
    Task PayAsync(decimal amount);
}

public class BkashStrategy : IPaymentStrategy
{
    public Task PayAsync(decimal amount)
    {
        Console.WriteLine("Bkash Payment");
        return Task.CompletedTask;
    }
}

public class NagadStrategy : IPaymentStrategy
{
    public Task PayAsync(decimal amount)
    {
        Console.WriteLine("Nagad Payment");
        return Task.CompletedTask;
    }
}
```

### Q80. What is Adapter Pattern?

**Answer:**

Incompatible interfaces-কে compatible করার জন্য Adapter Pattern ব্যবহার হয়।

Example:

External payment gateway-এর API-কে নিজের `IPaymentGateway` interface-এর সঙ্গে integrate করা।

### Q81. What is Decorator Pattern?

**Answer:**

Existing object-এর behavior runtime-এ extend করতে Decorator Pattern ব্যবহার করা হয়।

Example:

- Logging
- Caching
- Retry
- Validation

### Q82. What is Domain Event?

**Answer:**

Domain-এর মধ্যে কোনো গুরুত্বপূর্ণ business event ঘটলে সেটি Domain Event হিসেবে represent করা হয়।

Example:

- DonationReceived
- OrderConfirmed
- PaymentCompleted

### Q83. What is Integration Event?

**Answer:**

একটি service-এর গুরুত্বপূর্ণ event অন্য service-কে communicate করার জন্য Integration Event ব্যবহার হয়।

Example:

Payment service `PaymentCompleted` event publish করল।

Notification service সেটি consume করল।

### Q84. What is Outbox Pattern?

**Answer:**

Database transaction এবং event publishing-এর consistency নিশ্চিত করতে Outbox Pattern ব্যবহার করা হয়।

একই database transaction-এ business data এবং outbox event save করা হয়।

Background worker পরে event publish করে।

### Q85. What is Eventual Consistency?

**Answer:**

Distributed system-এ সব service একই সময়ে updated না হলেও কিছু সময় পরে consistent state-এ পৌঁছাতে পারাকে Eventual Consistency বলে।

---

# 8. Performance, Testing & DevOps

### Q86. What is Unit Testing?

**Answer:**

একটি ছোট unit বা method-কে isolated অবস্থায় test করাকে Unit Testing বলে।

Example:

```csharp
[Fact]
public void Add_ShouldReturnCorrectResult()
{
    int result = 10 + 20;

    Assert.Equal(30, result);
}
```

Popular tools:

- xUnit
- NUnit
- MSTest
- Moq

### Q87. What is Integration Testing?

**Answer:**

একাধিক component একসঙ্গে ঠিকমতো কাজ করছে কি না পরীক্ষা করাকে Integration Testing বলে।

Example:

- API + SQL Server
- Repository + Stored Procedure
- Authentication + Authorization

### Q88. What is Mocking?

**Answer:**

Unit Test-এ external dependency-এর পরিবর্তে controlled fake behavior provide করাকে Mocking বলে।

```csharp
var mock = new Mock<IEmployeeRepository>();

mock.Setup(x => x.GetByIdAsync(1))
    .ReturnsAsync(new Employee
    {
        EmployeeId = 1,
        Name = "Rahim"
    });
```

### Q89. What is Docker?

**Answer:**

Docker application এবং তার dependencies-কে container image-এ package করে consistent environment-এ run করতে সাহায্য করে।

.NET API:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 8080

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM base AS final
WORKDIR /app

COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

### Q90. What is CI/CD?

**Answer:**

CI = Continuous Integration।

CD = Continuous Delivery অথবা Continuous Deployment।

CI/CD pipeline-এর মাধ্যমে:

1. Code Build
2. Automated Testing
3. Code Analysis
4. Docker Build
5. Deployment

automate করা যায়।

### Q91. What is Redis Distributed Cache?

**Answer:**

একাধিক application instance-এর মধ্যে shared cache হিসেবে Redis ব্যবহার করা যায়।

এতে একই data প্রতিটি application server-এ আলাদাভাবে store করতে হয় না।

### Q92. What is Load Balancing?

**Answer:**

একাধিক server-এর মধ্যে incoming traffic distribute করাকে Load Balancing বলে।

এতে availability এবং scalability বাড়তে পারে।

### Q93. What is Horizontal vs Vertical Scaling?

**Horizontal Scaling:**

আরও server বা instance যোগ করা।

**Vertical Scaling:**

একটি server-এর CPU, RAM বা storage বৃদ্ধি করা।

### Q94. What is Observability?

**Answer:**

Application-এর internal behavior বুঝতে তিনটি গুরুত্বপূর্ণ pillar ব্যবহার হয়:

- Logs
- Metrics
- Traces

Tools:

- OpenTelemetry
- Prometheus
- Grafana
- Loki

### Q95. What is Distributed Tracing?

**Answer:**

একটি request বিভিন্ন service-এর মধ্য দিয়ে যাওয়ার সময় পুরো journey track করাকে Distributed Tracing বলে।

এতে slow service এবং bottleneck identify করা সহজ হয়।

---

# 9. Real-world Project Scenario Questions

### Q96. Your API is slow when returning 100,000 records. How will you optimize it?

**Answer:**

আমি নিচের approach নেব:

1. Pagination implement করব।
2. Only required columns select করব।
3. SQL Execution Plan check করব।
4. Proper indexes তৈরি করব।
5. Large response-এর জন্য streaming বিবেচনা করব।
6. ADO.NET DataReader ব্যবহার করতে পারি।
7. Compression এবং caching বিবেচনা করব।
8. Query এবং API performance measure করব।

Example:

```sql
SELECT EmployeeId, Name, Salary
FROM Employees
ORDER BY EmployeeId
OFFSET @Offset ROWS
FETCH NEXT @PageSize ROWS ONLY;
```

### Q97. Your payment API receives the same request twice. How do you prevent duplicate payment?

**Answer:**

Idempotency Key ব্যবহার করব।

Workflow:

1. Client unique Idempotency Key পাঠাবে।
2. Server key database-এ check করবে।
3. আগের successful result থাকলে সেটি return করবে।
4. না থাকলে payment processing করবে।
5. Key এবং result persist করবে।
6. Concurrent duplicate request prevent করতে unique constraint ব্যবহার করবে।

### Q98. Two users update the same employee record simultaneously. How do you handle it?

**Answer:**

Optimistic Concurrency Control ব্যবহার করব।

SQL Server-এ rowversion column রাখব।

```sql
UPDATE Employees
SET Name = @Name
WHERE EmployeeId = @Id
AND RowVer = @OldRowVer;
```

Affected rows 0 হলে conflict response return করব।

### Q99. A user from Tenant A accesses Tenant B's donation record. How do you prevent this?

**Answer:**

Tenant Isolation enforce করব।

1. JWT-এর trusted tenant claim verify করব।
2. Server-side tenant context তৈরি করব।
3. Repository-তে TenantId filter বাধ্যতামূলক করব।
4. Resource-level authorization check করব।
5. TenantId ছাড়া sensitive query execute করব না।
6. Cross-tenant integration tests লিখব।

### Q100. How would you design a Donation Management System using .NET and Angular?

**Answer:**

আমি নিচের architecture ব্যবহার করব:

**Frontend:**
- Angular 20
- Standalone Components
- Signals
- RxJS
- Reactive Forms
- HTTP Interceptors
- Route Guards

**Backend:**
- ASP.NET Core Web API
- Clean Architecture
- CQRS
- ADO.NET
- Repository Pattern
- JWT Authentication
- Role-Based Authorization

**Database:**
- SQL Server
- Stored Procedures
- Transactions
- Foreign Keys
- Indexes
- RowVersion

**Main Modules:**

```text
Donation Platform
│
├── Authentication
├── Organization Management
├── User & Role Management
├── Donor Management
├── Donation Management
├── Payment Management
├── Fund Management
├── Student Sponsorship
├── Expense Management
├── Ledger Management
├── Reports
└── Audit Logs
```

**Donation Transaction Workflow:**

```text
Donor
  |
  v
Create Donation
  |
  v
Payment Gateway
  |
  v
Verify Payment
  |
  v
Database Transaction
  |
  |-- Donation Record
  |-- Payment Record
  |-- Ledger Entry
  |-- Fund Balance Update
  |
  v
Commit Transaction
  |
  v
Generate Receipt
  |
  v
Send Notification
```

গুরুত্বপূর্ণ বিষয়:

- Payment gateway callback-এর signature verify করতে হবে।
- Duplicate callback prevent করতে হবে।
- Payment এবং ledger update transactionally manage করতে হবে।
- Failed payment-এর জন্য proper status রাখতে হবে।
- প্রতিটি organization-এর data isolate করতে হবে।
- Financial records-এর audit trail রাখতে হবে।

---

# 🎯 Final Advanced Interview Preparation Checklist

## C# Advanced

- [ ] Task.WhenAll / WhenAny
- [ ] CancellationToken
- [ ] Span / Memory
- [ ] Record
- [ ] Expression Tree
- [ ] Thread Safety
- [ ] SemaphoreSlim
- [ ] Garbage Collection

## ASP.NET Core

- [ ] Middleware Pipeline
- [ ] Global Exception Handling
- [ ] IHttpClientFactory
- [ ] BackgroundService
- [ ] Health Checks
- [ ] Rate Limiting
- [ ] Output Caching
- [ ] Idempotency
- [ ] ProblemDetails

## ADO.NET

- [ ] SqlConnection
- [ ] SqlCommand
- [ ] SqlDataReader
- [ ] SqlTransaction
- [ ] TVP
- [ ] Multiple Result Sets
- [ ] Connection Pooling
- [ ] Optimistic Concurrency
- [ ] Parameterized Queries

## SQL Server

- [ ] SARGable Queries
- [ ] Execution Plans
- [ ] Covering Index
- [ ] Composite Index
- [ ] Parameter Sniffing
- [ ] Query Store
- [ ] Window Functions
- [ ] Isolation Levels
- [ ] Deadlock

## Angular 20

- [ ] Signals
- [ ] Computed
- [ ] Effect
- [ ] RxJS Operators
- [ ] takeUntilDestroyed
- [ ] Functional Interceptor
- [ ] Functional Guards
- [ ] Lazy Loading
- [ ] @defer
- [ ] OnPush
- [ ] Hydration

## Architecture & DevOps

- [ ] Clean Architecture
- [ ] CQRS
- [ ] DDD
- [ ] Vertical Slice Architecture
- [ ] Outbox Pattern
- [ ] Multi-Tenancy
- [ ] Docker
- [ ] CI/CD
- [ ] Redis
- [ ] OpenTelemetry

---

# 💡 Final Advice

Professional .NET Developer হতে হলে শুধু syntax জানলেই হবে না।

নিচের প্রশ্নগুলোর উত্তর দিতে পারতে হবে:

1. কেন ADO.NET ব্যবহার করছ?
2. কেন Repository Pattern ব্যবহার করছ?
3. কীভাবে SQL Injection prevent করছ?
4. কীভাবে concurrent requests handle করছ?
5. কীভাবে JWT secure করছ?
6. কীভাবে Multi-Tenant data isolate করছ?
7. কীভাবে slow API optimize করছ?
8. কীভাবে database transaction maintain করছ?
9. কীভাবে Angular application-এর state manage করছ?
10. কীভাবে production application monitor করছ?

**Interview Golden Rule:**

> Definition + Practical Example + Real-world Use Case + Trade-off

এই চারটি বিষয় বুঝিয়ে বলতে পারলে technical interview-তে তোমার উত্তর অনেক বেশি শক্তিশালী হবে।

---

**Document:** Advanced .NET Developer Interview Cheat Sheet  
**Part:** 02  
**Language:** Bangla  
**Level:** Intermediate to Advanced  
**Stack:** ASP.NET Core | Angular 20 | ADO.NET | SQL Server
