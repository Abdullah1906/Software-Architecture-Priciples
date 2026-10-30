# 🚀 Full Stack .NET Developer Interview Questions & Answers
### Basic to Advanced | Bangla Interview Preparation

**Technology Stack:** C# | ASP.NET Core | Web API | Angular 20 | TypeScript | ADO.NET | SQL Server | SSMS | JWT | Clean Architecture

---

## 📌 Table of Contents

1. [C# Fundamentals](#1-c-fundamentals)
2. [OOP Concepts](#2-oop-concepts)
3. [Advanced C#](#3-advanced-c)
4. [ASP.NET Core](#4-aspnet-core)
5. [ASP.NET Core Web API](#5-aspnet-core-web-api)
6. [ADO.NET](#6-adonet)
7. [SQL Server & SSMS](#7-sql-server--ssms)
8. [Angular 20](#8-angular-20)
9. [Authentication & Authorization](#9-authentication--authorization)
10. [SOLID Principles & Design Patterns](#10-solid-principles--design-patterns)
11. [Clean Architecture](#11-clean-architecture)
12. [Performance & Security](#12-performance--security)
13. [Real-world Scenario Based Questions](#13-real-world-scenario-based-questions)

---

# 1. C# Fundamentals

### Q1. What is C#?

**Answer:**

C# হলো Microsoft-এর তৈরি একটি modern, object-oriented, type-safe programming language।

এটি ব্যবহার করে আমরা তৈরি করতে পারি:

- Web Applications
- Desktop Applications
- REST APIs
- Cloud Applications
- Enterprise Applications
- Game Development

C# সাধারণত .NET Runtime-এর মাধ্যমে execute হয়।

### Q2. What is .NET?

**Answer:**

.NET হলো Microsoft-এর একটি development platform, যার মাধ্যমে বিভিন্ন ধরনের application তৈরি করা যায়।

.NET-এর প্রধান components:

- CLR (Common Language Runtime)
- BCL (Base Class Library)
- SDK (Software Development Kit)
- ASP.NET Core
- ADO.NET

### Q3. What is CLR?

**Answer:**

CLR-এর পূর্ণরূপ Common Language Runtime।

এটি .NET application execute করার environment।

CLR-এর কাজ:

- Memory Management
- Garbage Collection
- Exception Handling
- Thread Management
- Type Safety
- JIT Compilation

### Q4. What is CTS?

**Answer:**

CTS-এর পূর্ণরূপ Common Type System।

.NET-এর বিভিন্ন programming language যেন একই type system ব্যবহার করতে পারে, সেটি নিশ্চিত করে।

উদাহরণ:

C#-এর `int` এবং VB.NET-এর `Integer` একই .NET type `System.Int32` নির্দেশ করে।

### Q5. What is CLS?

**Answer:**

CLS-এর পূর্ণরূপ Common Language Specification।

এটি .NET languages-এর মধ্যে interoperability নিশ্চিত করার জন্য কিছু common rules নির্ধারণ করে।

### Q6. What is the difference between .NET Framework and modern .NET?

| .NET Framework | Modern .NET |
|---|---|
| Primarily Windows-focused | Cross-platform |
| Older platform | Modern platform |
| Limited container support | Strong container support |
| Legacy applications | Modern applications |
| Windows ecosystem | Windows, Linux, macOS |

Modern .NET-কে .NET 5 থেকে .NET 10 পর্যন্ত version হিসেবে প্রকাশ করা হয়েছে।

### Q7. What is the difference between Value Type and Reference Type?

**Answer:**

Value Type সরাসরি তার value ধারণ করে।

Reference Type object-এর reference ধারণ করে।

**Value Type Examples:**
- int
- float
- double
- bool
- struct
- enum

**Reference Type Examples:**
- class
- interface
- array
- string
- delegate

```csharp
int a = 10;
int b = a;

b = 20;

Console.WriteLine(a); // 10
```

Reference Type:

```csharp
class Employee
{
    public string Name { get; set; }
}

Employee e1 = new Employee();
e1.Name = "Rahim";

Employee e2 = e1;
e2.Name = "Karim";

Console.WriteLine(e1.Name); // Karim
```

### Q8. What is the difference between var, dynamic and object?

| var | dynamic | object |
|---|---|---|
| Compile-time type inference | Runtime binding | Base type of all .NET types |
| Type fixed after compilation | Type checked at runtime | Requires casting for specific operations |
| Strongly typed | Less compile-time checking | Strongly typed |

```csharp
var name = "Abdullah";

dynamic value = 100;
value = "Hello";

object data = "Hello";

string result = (string)data;
```

### Q9. What is the difference between const and readonly?

**const:**
- Compile-time constant।
- Declaration-এর সময় value দিতে হয়।
- Implicitly static।

**readonly:**
- Runtime-এ value assign করা যায়।
- Declaration অথবা constructor-এ assign করা যায়।

```csharp
public class Employee
{
    public const double Tax = 0.05;

    public readonly int EmployeeId;

    public Employee(int id)
    {
        EmployeeId = id;
    }
}
```

### Q10. What is Boxing and Unboxing?

**Answer:**

Value Type-কে Reference Type হিসেবে convert করাকে Boxing বলে।

Reference Type থেকে Value Type-এ convert করাকে Unboxing বলে।

```csharp
int number = 100;

// Boxing
object obj = number;

// Unboxing
int result = (int)obj;
```

---

# 2. OOP Concepts

### Q11. What is OOP?

**Answer:**

OOP-এর পূর্ণরূপ Object-Oriented Programming।

এটি এমন একটি programming paradigm যেখানে Class এবং Object-এর মাধ্যমে application design করা হয়।

OOP-এর চারটি প্রধান Pillars:

1. Encapsulation
2. Inheritance
3. Polymorphism
4. Abstraction

### Q12. What is Encapsulation?

**Answer:**

Data এবং Methods-কে একটি class-এর মধ্যে একত্রিত রাখা এবং বাইরের সরাসরি access নিয়ন্ত্রণ করাকে Encapsulation বলে।

```csharp
public class BankAccount
{
    private decimal balance;

    public void Deposit(decimal amount)
    {
        if (amount > 0)
            balance += amount;
    }

    public decimal GetBalance()
    {
        return balance;
    }
}
```

এখানে `balance` private হওয়ায় সরাসরি পরিবর্তন করা যায় না।

### Q13. What is Inheritance?

**Answer:**

একটি class অন্য class-এর properties এবং methods গ্রহণ করতে পারলে তাকে Inheritance বলে।

```csharp
public class Animal
{
    public void Eat()
    {
        Console.WriteLine("Eating");
    }
}

public class Dog : Animal
{
    public void Bark()
    {
        Console.WriteLine("Barking");
    }
}
```

### Q14. What is Polymorphism?

**Answer:**

একই method বা interface বিভিন্নভাবে কাজ করতে পারাকে Polymorphism বলে।

দুই ধরনের:

- Compile-time Polymorphism (Method Overloading)
- Runtime Polymorphism (Method Overriding)

```csharp
public class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }

    public int Add(int a, int b, int c)
    {
        return a + b + c;
    }
}
```

Method Overriding:

```csharp
public class Animal
{
    public virtual void Sound()
    {
        Console.WriteLine("Animal Sound");
    }
}

public class Cat : Animal
{
    public override void Sound()
    {
        Console.WriteLine("Meow");
    }
}
```

### Q15. What is Abstraction?

**Answer:**

অপ্রয়োজনীয় implementation details লুকিয়ে শুধু প্রয়োজনীয় functionality প্রকাশ করাকে Abstraction বলে।

C#-এ Abstract Class এবং Interface ব্যবহার করা হয়।

```csharp
public interface IPayment
{
    void Pay(decimal amount);
}

public class BkashPayment : IPayment
{
    public void Pay(decimal amount)
    {
        Console.WriteLine("Payment through Bkash");
    }
}
```

### Q16. Difference between Abstract Class and Interface?

| Abstract Class | Interface |
|---|---|
| Abstract ও concrete methods থাকতে পারে | Contract define করে |
| Fields থাকতে পারে | Instance fields থাকতে পারে না |
| Constructor থাকতে পারে | Instance constructor থাকে না |
| একটি class একটি class inherit করে | একাধিক interface implement করতে পারে |
| Shared base implementation | Shared contract |

### Q17. What is the difference between Method Overloading and Overriding?

**Overloading:**
একই class-এ একই method name কিন্তু different parameters।

**Overriding:**
Parent class-এর virtual method child class-এ নতুনভাবে implement করা।

### Q18. What is a Constructor?

**Answer:**

Constructor হলো special method, যা object তৈরি হওয়ার সময় automatically execute হয়।

```csharp
public class Employee
{
    public string Name { get; set; }

    public Employee(string name)
    {
        Name = name;
    }
}
```

### Q19. What is Static Class?

**Answer:**

Static class-এর object তৈরি করা যায় না।

এর members static হতে হয়।

```csharp
public static class MathHelper
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}
```

### Q20. What is Sealed Class?

**Answer:**

যে class থেকে অন্য class inherit করতে পারে না তাকে Sealed Class বলে।

```csharp
public sealed class SecurityService
{
}
```

---

# 3. Advanced C#

### Q21. What is Exception Handling?

**Answer:**

Application-এ runtime error handle করার process হলো Exception Handling।

C#-এ ব্যবহার হয়:

- try
- catch
- finally
- throw

```csharp
try
{
    int result = 10 / 0;
}
catch (DivideByZeroException ex)
{
    Console.WriteLine(ex.Message);
}
finally
{
    Console.WriteLine("Execution Completed");
}
```

### Q22. What is async and await?

**Answer:**

`async` এবং `await` asynchronous programming-এর জন্য ব্যবহৃত হয়।

এটি I/O-bound কাজের সময় thread অপ্রয়োজনীয়ভাবে block হওয়া থেকে বিরত রাখতে সাহায্য করে।

```csharp
public async Task<string> GetDataAsync()
{
    string data = await File.ReadAllTextAsync("data.txt");

    return data;
}
```

### Q23. What is Task and Thread?

| Task | Thread |
|---|---|
| Represents an asynchronous operation | Represents an execution thread |
| High-level abstraction | Lower-level abstraction |
| Supports async/await | Manages execution |
| Usually uses ThreadPool for CPU work | Can represent a dedicated thread |

### Q24. What is LINQ?

**Answer:**

LINQ-এর পূর্ণরূপ Language Integrated Query।

C# collection, database এবং বিভিন্ন data source query করার জন্য ব্যবহৃত হয়।

```csharp
var employees = new List<Employee>
{
    new() { Name = "Rahim", Salary = 30000 },
    new() { Name = "Karim", Salary = 50000 },
    new() { Name = "Abdullah", Salary = 60000 }
};

var result = employees
    .Where(x => x.Salary > 40000)
    .OrderByDescending(x => x.Salary)
    .ToList();
```

### Q25. What is IEnumerable and IQueryable?

| IEnumerable | IQueryable |
|---|---|
| In-memory iteration abstraction | Queryable data source abstraction |
| LINQ operations generally execute in memory | Provider may translate expressions into SQL |
| Uses delegates | Uses expression trees |
| Suitable for in-memory collections | Suitable for remote query providers |

### Q26. What is Delegate?

**Answer:**

Delegate হলো type-safe function reference।

```csharp
public delegate int Operation(int a, int b);

public class Program
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}
```

### Q27. What is Event?

**Answer:**

Event হলো Delegate-এর উপর ভিত্তি করে তৈরি notification mechanism।

কোনো নির্দিষ্ট ঘটনা ঘটলে subscriber-দের notify করা যায়।

### Q28. What is Garbage Collection?

**Answer:**

Garbage Collector (GC) হলো .NET-এর automatic memory management system।

এটি আর ব্যবহার হচ্ছে না এমন managed objects-এর memory reclaim করে।

GC-এর Generations:

- Generation 0
- Generation 1
- Generation 2
- Large Object Heap (LOH)

### Q29. What is IDisposable?

**Answer:**

IDisposable interface unmanaged resources বা অন্যান্য disposable resources release করার জন্য ব্যবহৃত হয়।

```csharp
using SqlConnection connection =
    new SqlConnection(connectionString);
```

`using` block শেষ হলে Dispose automatically call হয়।

### Q30. What is Dependency Injection?

**Answer:**

একটি class-এর প্রয়োজনীয় dependency বাইরে থেকে provide করাকে Dependency Injection বলে।

এটি Loose Coupling এবং Testability বাড়ায়।

```csharp
public interface IEmailService
{
    void SendEmail(string message);
}

public class EmailService : IEmailService
{
    public void SendEmail(string message)
    {
        Console.WriteLine(message);
    }
}

public class OrderService
{
    private readonly IEmailService _emailService;

    public OrderService(IEmailService emailService)
    {
        _emailService = emailService;
    }
}
```

---

# 4. ASP.NET Core

### Q31. What is ASP.NET Core?

**Answer:**

ASP.NET Core হলো .NET-এর cross-platform web development framework।

এটি দিয়ে তৈরি করা যায়:

- REST API
- MVC Applications
- Web Applications
- Minimal APIs
- Real-time Applications

### Q32. What is Middleware?

**Answer:**

Middleware হলো HTTP request এবং response pipeline-এর একটি component।

একটি request একাধিক middleware অতিক্রম করে।

উদাহরণ:

- Exception Handling
- Authentication
- Authorization
- CORS
- Routing

```csharp
app.UseExceptionHandler();

app.UseHttpsRedirection();

app.UseCors();

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

### Q33. What is the difference between Use, Run and Map?

**Use:**
পরবর্তী middleware call করতে পারে।

**Run:**
Pipeline terminate করে।

**Map:**
নির্দিষ্ট path অনুযায়ী আলাদা pipeline তৈরি করে।

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Before");

    await next();

    Console.WriteLine("After");
});

app.Run(async context =>
{
    await context.Response.WriteAsync("Hello");
});
```

### Q34. What is appsettings.json?

**Answer:**

Application configuration রাখার জন্য ব্যবহৃত JSON file।

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=DemoDb;Trusted_Connection=True;TrustServerCertificate=True;"
  }
}
```

Production-এ sensitive credentials নিরাপদভাবে সংরক্ষণ করতে হবে।

### Q35. What are Dependency Injection Lifetimes?

**Answer:**

ASP.NET Core-এ তিন ধরনের প্রধান DI Lifetime রয়েছে।

| Lifetime | Description |
|---|---|
| Transient | প্রতিবার নতুন instance |
| Scoped | প্রতি HTTP request-এ একটি instance |
| Singleton | Application lifetime-এ একটি instance |

```csharp
builder.Services.AddTransient<IEmailService, EmailService>();

builder.Services.AddScoped<IUserService, UserService>();

builder.Services.AddSingleton<ICacheService, CacheService>();
```

### Q36. What is IConfiguration?

**Answer:**

Application configuration read করার interface।

```csharp
public class MyService
{
    private readonly IConfiguration _configuration;

    public MyService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string GetConnectionString()
    {
        return _configuration.GetConnectionString("DefaultConnection")!;
    }
}
```

### Q37. What is Kestrel?

**Answer:**

Kestrel হলো ASP.NET Core-এর built-in cross-platform web server।

এটি HTTP requests receive এবং process করে।

### Q38. What is the difference between MVC and Web API?

| MVC | Web API |
|---|---|
| Usually returns Views | Usually returns data |
| Razor Views | JSON/XML responses |
| Server-side rendering | API-based communication |
| Web applications | Frontend/backend integration |

### Q39. What is Model Binding?

**Answer:**

HTTP request থেকে data নিয়ে Action Method-এর parameter বা model-এ automatically map করাকে Model Binding বলে।

```csharp
[HttpPost]
public IActionResult Create(Employee employee)
{
    return Ok(employee);
}
```

### Q40. What is Model Validation?

**Answer:**

Incoming request data valid কিনা যাচাই করার process।

```csharp
public class EmployeeDto
{
    [Required]
    public string Name { get; set; } = string.Empty;

    [Range(10000, 100000)]
    public decimal Salary { get; set; }
}
```

---

# 5. ASP.NET Core Web API

### Q41. What is REST API?

**Answer:**

REST-এর পূর্ণরূপ Representational State Transfer।

REST API হলো এমন একটি architectural style, যেখানে HTTP protocol ব্যবহার করে client ও server-এর মধ্যে communication হয়।

### Q42. What are HTTP Methods?

| Method | Purpose |
|---|---|
| GET | Data retrieve |
| POST | New resource create |
| PUT | Resource replace/update |
| PATCH | Partial update |
| DELETE | Resource delete |

### Q43. What are HTTP Status Codes?

| Status | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Unprocessable Content |
| 500 | Internal Server Error |

### Q44. Difference between 401 and 403?

**401 Unauthorized:**

User authenticated নয় অথবা valid authentication credentials নেই।

**403 Forbidden:**

User authenticated, কিন্তু requested resource access করার permission নেই।

### Q45. What is DTO?

**Answer:**

DTO-এর পূর্ণরূপ Data Transfer Object।

Client এবং server-এর মধ্যে নির্দিষ্ট data transfer করার জন্য DTO ব্যবহার করা হয়।

```csharp
public class CreateEmployeeDto
{
    public string Name { get; set; } = string.Empty;

    public decimal Salary { get; set; }
}
```

DTO ব্যবহারের সুবিধা:

- Security
- Data validation
- Separation of concerns
- API contract control

### Q46. What is Repository Pattern?

**Answer:**

Repository Pattern database access logic-কে business logic থেকে আলাদা করে।

```csharp
public interface IEmployeeRepository
{
    Task<Employee?> GetByIdAsync(int id);
}
```

```csharp
public class EmployeeRepository : IEmployeeRepository
{
    public async Task<Employee?> GetByIdAsync(int id)
    {
        // Database operation
        return null;
    }
}
```

### Q47. What is ActionResult and IActionResult?

**Answer:**

`IActionResult` হলো বিভিন্ন ধরনের HTTP response return করার interface।

`ActionResult<T>` নির্দিষ্ট response data type এবং HTTP result দুটোই support করে।

```csharp
[HttpGet("{id}")]
public ActionResult<EmployeeDto> GetEmployee(int id)
{
    if (id <= 0)
        return BadRequest();

    return Ok(new EmployeeDto());
}
```

### Q48. What is API Versioning?

**Answer:**

API-এর একাধিক version maintain করাকে API Versioning বলে।

উদাহরণ:

```text
/api/v1/employees
/api/v2/employees
```

এতে পুরোনো client break না করেই নতুন API functionality তৈরি করা যায়।

### Q49. What is Swagger / OpenAPI?

**Answer:**

OpenAPI হলো API description specification।

Swagger ecosystem ব্যবহার করে API document, visualize এবং test করা যায়।

ASP.NET Core-এ OpenAPI document generate করে Swagger UI বা অন্য compatible UI দিয়ে API পরীক্ষা করা যায়।

### Q50. What is CORS?

**Answer:**

CORS-এর পূর্ণরূপ Cross-Origin Resource Sharing।

Browser-এ একটি origin থেকে অন্য origin-এর API request করার permission নিয়ন্ত্রণ করে।

```csharp
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
```

Middleware:

```csharp
app.UseCors("AngularPolicy");
```

---

# 6. ADO.NET

### Q51. What is ADO.NET?

**Answer:**

ADO.NET হলো .NET-এর data access technology।

এটি ব্যবহার করে SQL Server-এর মতো database-এর সঙ্গে connection তৈরি, query execute এবং data retrieve করা যায়।

### Q52. What are the main components of ADO.NET?

1. SqlConnection
2. SqlCommand
3. SqlDataReader
4. SqlDataAdapter
5. DataSet
6. DataTable
7. SqlTransaction
8. SqlParameter

### Q53. What is SqlConnection?

**Answer:**

SQL Server database-এর সঙ্গে connection তৈরি করার জন্য SqlConnection ব্যবহার করা হয়।

```csharp
using Microsoft.Data.SqlClient;

using SqlConnection connection =
    new SqlConnection(connectionString);

await connection.OpenAsync();
```

### Q54. What is SqlCommand?

**Answer:**

SQL statement অথবা Stored Procedure execute করার জন্য SqlCommand ব্যবহার করা হয়।

```csharp
using SqlCommand command =
    new SqlCommand("SELECT * FROM Employees", connection);
```

### Q55. What is SqlDataReader?

**Answer:**

SqlDataReader database থেকে data forward-only এবং read-only fashion-এ retrieve করে।

এটি সাধারণত দ্রুত data reading-এর জন্য ব্যবহার করা হয়।

```csharp
using SqlDataReader reader =
    await command.ExecuteReaderAsync();

while (await reader.ReadAsync())
{
    Console.WriteLine(reader["Name"]);
}
```

### Q56. Difference between ExecuteReader, ExecuteScalar and ExecuteNonQuery?

| Method | Purpose |
|---|---|
| ExecuteReader | Multiple rows retrieve |
| ExecuteScalar | First row-এর first column retrieve |
| ExecuteNonQuery | INSERT, UPDATE, DELETE-এর মতো operations |

```csharp
// Read multiple rows
using var reader = await command.ExecuteReaderAsync();

// Read single value
var count = await command.ExecuteScalarAsync();

// Insert / Update / Delete
int affectedRows = await command.ExecuteNonQueryAsync();
```

### Q57. What is SqlDataAdapter?

**Answer:**

SqlDataAdapter database থেকে data নিয়ে DataTable বা DataSet populate করতে পারে।

এটি disconnected data access model support করে।

### Q58. What is DataSet and DataTable?

**DataTable:**

Memory-তে একটি table-এর মতো data রাখে।

**DataSet:**

একাধিক DataTable এবং তাদের relationships ধারণ করতে পারে।

### Q59. What is Connected and Disconnected Architecture?

**Connected Architecture:**

Database connection open রেখে data access করা হয়।

উদাহরণ: SqlDataReader।

**Disconnected Architecture:**

Data retrieve করার পরে connection close করেও memory-তে data নিয়ে কাজ করা যায়।

উদাহরণ: DataSet, DataTable।

### Q60. What is Stored Procedure?

**Answer:**

Stored Procedure হলো SQL Server database-এ সংরক্ষিত একটি pre-defined SQL program।

```sql
CREATE PROCEDURE sp_GetEmployees
AS
BEGIN
    SELECT EmployeeId, Name, Salary
    FROM Employees;
END;
```

ADO.NET থেকে execute:

```csharp
using SqlCommand command =
    new SqlCommand("sp_GetEmployees", connection);

command.CommandType = CommandType.StoredProcedure;
```

### Q61. What is SqlParameter?

**Answer:**

SQL query বা Stored Procedure-তে parameter pass করার জন্য SqlParameter ব্যবহার হয়।

```csharp
using SqlCommand command =
    new SqlCommand("sp_GetEmployeeById", connection);

command.CommandType = CommandType.StoredProcedure;

command.Parameters.Add("@EmployeeId", SqlDbType.Int).Value = 10;
```

এটি SQL Injection-এর ঝুঁকি কমাতে সাহায্য করে।

### Q62. What is SQL Injection?

**Answer:**

যখন attacker malicious SQL input ব্যবহার করে database query manipulate করে, তখন তাকে SQL Injection বলে।

**Bad Practice:**

```csharp
string query =
    "SELECT * FROM Users WHERE Username = '" + username + "'";
```

**Good Practice:**

```csharp
command.CommandText =
    "SELECT * FROM Users WHERE Username = @Username";

command.Parameters.Add("@Username", SqlDbType.NVarChar, 100)
    .Value = username;
```

### Q63. What is SqlTransaction?

**Answer:**

একাধিক database operation-কে একটি atomic unit হিসেবে execute করার জন্য SqlTransaction ব্যবহার করা হয়।

সব operation successful হলে Commit এবং কোনো operation fail হলে Rollback করা হয়।

```csharp
using var transaction =
    await connection.BeginTransactionAsync();

try
{
    using var command = connection.CreateCommand();

    command.Transaction = (SqlTransaction)transaction;

    command.CommandText =
        "UPDATE Accounts SET Balance = Balance - 1000 WHERE Id = 1";

    await command.ExecuteNonQueryAsync();

    command.CommandText =
        "UPDATE Accounts SET Balance = Balance + 1000 WHERE Id = 2";

    await command.ExecuteNonQueryAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

### Q64. What is Connection Pooling?

**Answer:**

Connection Pooling হলো database connection reuse করার mechanism।

প্রতিবার নতুন physical connection তৈরি না করে existing connection pool থেকে connection নেওয়া হয়।

এতে performance উন্নত হয়।

### Q65. Why should we use async ADO.NET methods?

**Answer:**

Database operation-এর মতো I/O-bound কাজ asynchronous করলে request thread অপ্রয়োজনীয়ভাবে block হয় না।

উদাহরণ:

```csharp
await connection.OpenAsync();

await command.ExecuteNonQueryAsync();

await reader.ReadAsync();
```

---

# 7. SQL Server & SSMS

### Q66. What is SQL Server?

**Answer:**

SQL Server হলো Microsoft-এর Relational Database Management System (RDBMS)।

এটি structured data store, retrieve এবং manage করতে ব্যবহৃত হয়।

### Q67. What is SSMS?

**Answer:**

SSMS-এর পূর্ণরূপ SQL Server Management Studio।

এটি SQL Server database manage, query execute, backup, restore এবং performance monitoring করার graphical tool।

### Q68. What is Primary Key?

**Answer:**

Primary Key একটি table-এর প্রতিটি row-কে uniquely identify করে।

- Duplicate value allowed নয়।
- NULL allowed নয়।
- একটি table-এ একটি Primary Key constraint থাকে, যা একাধিক column নিয়ে composite হতে পারে।

### Q69. What is Foreign Key?

**Answer:**

Foreign Key দুটি table-এর মধ্যে relationship তৈরি করে।

```sql
CREATE TABLE Departments
(
    DepartmentId INT PRIMARY KEY,
    DepartmentName VARCHAR(100)
);

CREATE TABLE Employees
(
    EmployeeId INT PRIMARY KEY,
    Name VARCHAR(100),
    DepartmentId INT,

    FOREIGN KEY (DepartmentId)
    REFERENCES Departments(DepartmentId)
);
```

### Q70. What is Unique Key?

**Answer:**

Unique Constraint একটি column বা column combination-এর duplicate value prevent করে।

Primary Key এবং Unique Key-এর মধ্যে পার্থক্য:

- একটি table-এ একটি Primary Key constraint থাকে।
- একাধিক Unique Constraint থাকতে পারে।
- Primary Key NULL গ্রহণ করে না।
- SQL Server-এ সাধারণ Unique Constraint-এর ক্ষেত্রে NULL value-এর জন্যও uniqueness enforcement হয়।

### Q71. What is Index?

**Answer:**

Index হলো database object, যা data retrieval দ্রুত করতে সাহায্য করে।

Types:

- Clustered Index
- Nonclustered Index
- Unique Index
- Composite Index
- Filtered Index

```sql
CREATE NONCLUSTERED INDEX IX_Employees_Name
ON Employees(Name);
```

### Q72. Clustered vs Nonclustered Index?

| Clustered | Nonclustered |
|---|---|
| Table data rows-এর physical organization নির্ধারণ করে | Separate index structure |
| একটি table-এ সর্বোচ্চ একটি | একাধিক থাকতে পারে |
| Leaf level-এ data rows থাকে | Leaf level-এ key ও row locator থাকে |

### Q73. What is JOIN?

**Answer:**

একাধিক table থেকে related data retrieve করার জন্য JOIN ব্যবহার করা হয়।

Types:

1. INNER JOIN
2. LEFT JOIN
3. RIGHT JOIN
4. FULL OUTER JOIN
5. CROSS JOIN

```sql
SELECT
    e.Name,
    d.DepartmentName
FROM Employees e
INNER JOIN Departments d
    ON e.DepartmentId = d.DepartmentId;
```

### Q74. Difference between WHERE and HAVING?

**WHERE:**

Grouping-এর আগে rows filter করে।

**HAVING:**

GROUP BY-এর পরে grouped results filter করে।

```sql
SELECT DepartmentId, COUNT(*) AS TotalEmployees
FROM Employees
WHERE Salary > 20000
GROUP BY DepartmentId
HAVING COUNT(*) > 5;
```

### Q75. What is GROUP BY?

**Answer:**

একই ধরনের data group করে aggregate calculation করার জন্য GROUP BY ব্যবহার হয়।

```sql
SELECT DepartmentId, COUNT(*) AS Total
FROM Employees
GROUP BY DepartmentId;
```

### Q76. What is Stored Procedure vs Function?

| Stored Procedure | Function |
|---|---|
| EXEC দিয়ে execute করা যায় | Query-এর মধ্যে ব্যবহার করা যায় |
| Multiple result sets return করতে পারে | নির্দিষ্ট function return type থাকে |
| Transaction control করা যায় | UDF-এর ক্ষেত্রে transaction control করা যায় না |
| DML operations করতে পারে | UDF-এ database state পরিবর্তনের সীমাবদ্ধতা রয়েছে |

### Q77. What is View?

**Answer:**

View হলো একটি virtual table, যা SQL query-এর উপর ভিত্তি করে তৈরি হয়।

```sql
CREATE VIEW vw_EmployeeDetails
AS
SELECT
    EmployeeId,
    Name,
    Salary
FROM Employees;
```

### Q78. What is CTE?

**Answer:**

CTE-এর পূর্ণরূপ Common Table Expression।

একটি query-এর মধ্যে temporary named result set তৈরি করার জন্য CTE ব্যবহার হয়।

```sql
WITH HighSalaryEmployees AS
(
    SELECT *
    FROM Employees
    WHERE Salary > 50000
)
SELECT *
FROM HighSalaryEmployees;
```

### Q79. What is Subquery?

**Answer:**

একটি SQL query-এর ভিতরে আরেকটি query থাকলে তাকে Subquery বলে।

```sql
SELECT *
FROM Employees
WHERE Salary >
(
    SELECT AVG(Salary)
    FROM Employees
);
```

### Q80. What is Normalization?

**Answer:**

Database redundancy কমানো এবং data integrity বাড়ানোর জন্য database table organize করার process হলো Normalization।

প্রধান Normal Forms:

- 1NF
- 2NF
- 3NF
- BCNF

### Q81. What is ACID Property?

**Answer:**

Database transaction-এর চারটি প্রধান property:

- Atomicity: সম্পূর্ণ transaction সফল হবে অথবা কোনো অংশই কার্যকর থাকবে না।
- Consistency: Database valid state-এ থাকবে।
- Isolation: Concurrent transactions-এর মধ্যে isolation বজায় থাকবে।
- Durability: Commit হওয়ার পর data স্থায়ী থাকবে।

### Q82. What is Deadlock?

**Answer:**

দুই বা তার বেশি transaction একে অপরের locked resource-এর জন্য অপেক্ষা করলে Deadlock হতে পারে।

Prevention:

- Consistent resource access order
- Short transactions
- Proper indexing
- Avoid unnecessary locks
- Deadlock retry strategy

### Q83. What is Trigger?

**Answer:**

Trigger হলো database object, যা নির্দিষ্ট INSERT, UPDATE বা DELETE event-এর পর বা পরিবর্তনের সময় automatically execute হয়।

Types:

- AFTER Trigger
- INSTEAD OF Trigger

### Q84. Difference between DELETE, TRUNCATE and DROP?

| DELETE | TRUNCATE | DROP |
|---|---|---|
| Selected rows delete | All rows remove | Entire object remove |
| WHERE supported | WHERE not supported | WHERE not supported |
| Row-level logging | Minimally logged in many cases | Object removal |
| Identity usually not reset | Identity reset | Object removed |

### Q85. What is Execution Plan?

**Answer:**

Execution Plan হলো SQL Server কীভাবে একটি query execute করছে তার graphical বা textual representation।

এটি দিয়ে identify করা যায়:

- Table Scan
- Index Seek
- Index Scan
- Key Lookup
- Join Operations
- Query Cost

---

# 8. Angular 20

### Q86. What is Angular?

**Answer:**

Angular হলো Google-এর তৈরি TypeScript-based frontend framework।

এটি Single Page Application (SPA) তৈরি করতে ব্যবহৃত হয়।

### Q87. What is Angular Component?

**Answer:**

Component হলো Angular application-এর একটি UI building block।

প্রধান অংশ:

- TypeScript class
- HTML Template
- CSS/SCSS
- Component Metadata

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-home',
  standalone: true,
  template: `<h1>Hello Angular 20</h1>`
})
export class HomeComponent {}
```

### Q88. What is Standalone Component?

**Answer:**

Standalone Component সরাসরি অন্য component, directive, pipe এবং প্রয়োজনীয় dependencies import করতে পারে।

এটির জন্য Angular NgModule বাধ্যতামূলক নয়।

Angular 20-এ Standalone architecture recommended।

### Q89. What is Data Binding?

**Answer:**

Component এবং HTML Template-এর মধ্যে data communication-কে Data Binding বলে।

Types:

1. Interpolation
2. Property Binding
3. Event Binding
4. Two-way Binding

```html
<h1>{{ title }}</h1>

<input [value]="name">

<button (click)="save()">Save</button>

<input [(ngModel)]="name">
```

### Q90. What is Directive?

**Answer:**

Directive HTML element-এর behavior বা appearance পরিবর্তন করে।

Types:

- Component Directive
- Structural Directive
- Attribute Directive

Angular built-in examples:

- @if
- @for
- @switch
- ngClass
- ngStyle

### Q91. What is Angular Service?

**Answer:**

Reusable business logic এবং API communication manage করার জন্য Service ব্যবহার হয়।

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class EmployeeService {

  getEmployees() {
    return 'Employee Data';
  }
}
```

### Q92. What is Dependency Injection in Angular?

**Answer:**

Angular DI system-এর মাধ্যমে component বা service-এর dependency provide করা হয়।

```typescript
import { Component, inject } from '@angular/core';

@Component({
  selector: 'app-employee',
  standalone: true,
  template: `<h1>Employee</h1>`
})
export class EmployeeComponent {

  private employeeService = inject(EmployeeService);
}
```

### Q93. What is Angular Routing?

**Answer:**

একটি Angular application-এর বিভিন্ন page/component-এর মধ্যে navigation manage করার system হলো Routing।

```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: 'employees',
    loadComponent: () =>
      import('./employees/employees.component')
        .then(m => m.EmployeesComponent)
  }
];
```

### Q94. What is Lazy Loading?

**Answer:**

Application-এর সব feature একসঙ্গে load না করে প্রয়োজন অনুযায়ী load করাকে Lazy Loading বলে।

এতে initial bundle size কমানো যায়।

```typescript
{
  path: 'admin',
  loadChildren: () =>
    import('./admin/admin.routes')
      .then(m => m.ADMIN_ROUTES)
}
```

### Q95. What is Angular Signals?

**Answer:**

Signals হলো Angular-এর reactive state management primitive।

Signal-এর value পরিবর্তন হলে dependent consumers update হয়।

```typescript
import { signal, computed } from '@angular/core';

count = signal(0);

doubleCount = computed(() => this.count() * 2);

increment() {
  this.count.update(value => value + 1);
}
```

### Q96. What is RxJS?

**Answer:**

RxJS হলো Reactive Programming Library।

Angular-এ asynchronous data stream manage করার জন্য ব্যবহৃত হয়।

Important concepts:

- Observable
- Observer
- Subscription
- Subject
- BehaviorSubject
- Operators
- Pipe

### Q97. Observable vs Promise?

| Observable | Promise |
|---|---|
| Multiple values emit করতে পারে | একটি value settle করে |
| Lazy | Eager |
| Unsubscribe করা যায় | Native Promise-এ সরাসরি cancellation নেই |
| RxJS operators support করে | Native chaining methods support করে |

### Q98. What is HttpClient?

**Answer:**

Angular HttpClient ব্যবহার করে backend API-এর সঙ্গে HTTP communication করা হয়।

```typescript
import { HttpClient } from '@angular/common/http';
import { inject } from '@angular/core';

export class EmployeeService {

  private http = inject(HttpClient);

  getEmployees() {
    return this.http.get('/api/employees');
  }
}
```

### Q99. What is HTTP Interceptor?

**Answer:**

HTTP request এবং response globally intercept করার জন্য Interceptor ব্যবহার হয়।

Common use cases:

- JWT Token Attach
- Error Handling
- Loading Indicator
- Request Logging
- Refresh Token Flow

### Q100. What is Route Guard?

**Answer:**

Angular Route Guard navigation allow বা deny করতে সাহায্য করে।

Types:

- CanActivate
- CanActivateChild
- CanDeactivate
- CanMatch

Important: Angular Route Guard শুধু frontend navigation control করে। Backend authorization অবশ্যই আলাদাভাবে implement করতে হবে।

### Q101. What is Reactive Form?

**Answer:**

Reactive Forms হলো model-driven form approach।

এতে FormControl, FormGroup এবং FormArray ব্যবহার করা হয়।

```typescript
import { FormBuilder, Validators } from '@angular/forms';

export class EmployeeComponent {

  private fb = inject(FormBuilder);

  employeeForm = this.fb.group({
    name: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]]
  });
}
```

### Q102. What is Angular Lifecycle Hooks?

**Answer:**

Component-এর বিভিন্ন lifecycle stage-এ নির্দিষ্ট কাজ করার জন্য Lifecycle Hooks ব্যবহার হয়।

Important Hooks:

- ngOnChanges()
- ngOnInit()
- ngDoCheck()
- ngAfterViewInit()
- ngOnDestroy()

Angular 20-এ নতুন rendering lifecycle APIs যেমন afterNextRender এবং afterEveryRender-ও রয়েছে।

### Q103. What is Change Detection?

**Answer:**

Component-এর data পরিবর্তন হলে UI update করার process হলো Change Detection।

Angular-এ Change Detection Strategy:

- Default
- OnPush

OnPush ব্যবহার করলে unnecessary checks কমানো সম্ভব।

### Q104. What is the difference between @Input and @Output?

**Answer:**

`@Input()` Parent থেকে Child-এ data পাঠায়।

`@Output()` Child থেকে Parent-এ event communicate করে।

Modern Angular-এ input() এবং output() APIs-ও ব্যবহার করা যায়।

### Q105. What is the difference between Subject and BehaviorSubject?

| Subject | BehaviorSubject |
|---|---|
| Initial value required নয় | Initial value required |
| New subscriber previous value পায় না | New subscriber latest value পায় |
| Event broadcasting | Current state sharing |

---

# 9. Authentication & Authorization

### Q106. What is Authentication?

**Answer:**

User কে তা verify করার process হলো Authentication।

উদাহরণ: Login with username and password।

### Q107. What is Authorization?

**Answer:**

Authenticated user কোন resource access করতে পারবে তা নির্ধারণ করার process হলো Authorization।

উদাহরণ:

- Admin
- Manager
- Employee

### Q108. What is JWT?

**Answer:**

JWT-এর পূর্ণরূপ JSON Web Token।

এটি claims বহনকারী একটি compact token format।

JWT-এর তিনটি অংশ:

1. Header
2. Payload
3. Signature

Example:

```text
xxxxx.yyyyy.zzzzz
```

### Q109. What is the difference between Access Token and Refresh Token?

| Access Token | Refresh Token |
|---|---|
| API access-এর জন্য | নতুন access token পাওয়ার জন্য |
| Short-lived | Usually longer-lived |
| Request-এ পাঠানো হয় | Token endpoint-এ ব্যবহার হয় |
| Expire হলে renew করতে হয় | Expire/revoke হলে পুনরায় login প্রয়োজন হতে পারে |

### Q110. What is Role-Based Authorization?

**Answer:**

User-এর Role অনুযায়ী access control করাকে Role-Based Authorization বলে।

```csharp
[Authorize(Roles = "Admin")]
[HttpGet("admin")]
public IActionResult GetAdminData()
{
    return Ok();
}
```

### Q111. What is Claims-Based Authorization?

**Answer:**

User-এর Claims-এর ভিত্তিতে authorization করা হয়।

```csharp
[Authorize(Policy = "CanManageEmployees")]
public IActionResult ManageEmployees()
{
    return Ok();
}
```

### Q112. What is Password Hashing?

**Answer:**

Password সরাসরি database-এ store না করে secure one-way hashing algorithm দিয়ে hash করে store করতে হয়।

ASP.NET Core Identity PasswordHasher বা অন্য established password hashing implementation ব্যবহার করা উচিত।

Password encryption এবং hashing এক জিনিস নয়।

### Q113. What is Refresh Token Rotation?

**Answer:**

প্রতিবার Refresh Token ব্যবহার করে নতুন token pair তৈরি হলে পুরোনো Refresh Token revoke করে নতুন Refresh Token দেওয়া হয়।

এতে token theft detection এবং replay attack mitigation সহজ হয়।

### Q114. What is OAuth 2.0?

**Answer:**

OAuth 2.0 হলো Authorization Framework।

এটি কোনো application-কে user-এর credentials সরাসরি না দিয়েও নির্দিষ্ট permission অনুযায়ী resource access করতে সাহায্য করে।

### Q115. What is OpenID Connect?

**Answer:**

OpenID Connect হলো OAuth 2.0-এর উপর নির্মিত authentication layer।

এটি user identity verify করার জন্য ID Token প্রদান করে।

---

# 10. SOLID Principles & Design Patterns

### Q116. What is SOLID?

**Answer:**

SOLID হলো object-oriented software design-এর পাঁচটি গুরুত্বপূর্ণ principle।

1. S — Single Responsibility Principle
2. O — Open/Closed Principle
3. L — Liskov Substitution Principle
4. I — Interface Segregation Principle
5. D — Dependency Inversion Principle

### Q117. What is Single Responsibility Principle?

**Answer:**

একটি class-এর পরিবর্তনের জন্য একটি মাত্র primary responsibility থাকা উচিত।

উদাহরণ:

EmployeeService শুধু business logic handle করবে।

EmployeeRepository শুধু database access handle করবে।

### Q118. What is Open/Closed Principle?

**Answer:**

Software entities extension-এর জন্য open এবং modification-এর জন্য closed হওয়া উচিত।

নতুন functionality যোগ করতে existing stable code বারবার পরিবর্তন না করাই লক্ষ্য।

### Q119. What is Liskov Substitution Principle?

**Answer:**

Child class-কে Parent class-এর জায়গায় ব্যবহার করলেও program-এর expected behavior ঠিক থাকতে হবে।

### Q120. What is Interface Segregation Principle?

**Answer:**

একটি বড় interface-এর পরিবর্তে ছোট এবং specific interface তৈরি করা উচিত।

### Q121. What is Dependency Inversion Principle?

**Answer:**

High-level module এবং low-level module উভয়ই abstraction-এর উপর নির্ভর করবে।

Concrete implementation-এর উপর সরাসরি dependency রাখা উচিত নয়।

### Q122. What is Singleton Design Pattern?

**Answer:**

একটি class-এর application scope-এ একটি instance নিশ্চিত করার pattern হলো Singleton।

ASP.NET Core:

```csharp
builder.Services.AddSingleton<ICacheService, CacheService>();
```

### Q123. What is Factory Pattern?

**Answer:**

Object creation logic encapsulate করার জন্য Factory Pattern ব্যবহার হয়।

```csharp
public interface IPayment
{
    void Pay();
}

public class PaymentFactory
{
    public IPayment Create(string type)
    {
        return type switch
        {
            "Bkash" => new BkashPayment(),
            "Nagad" => new NagadPayment(),
            _ => throw new NotSupportedException()
        };
    }
}
```

### Q124. What is Unit of Work Pattern?

**Answer:**

একাধিক repository operation-কে একটি transaction-এর অধীনে coordinate করার pattern হলো Unit of Work।

ADO.NET application-এ এটি SqlTransaction ব্যবহার করে implement করা যায়।

---

# 11. Clean Architecture

### Q125. What is Clean Architecture?

**Answer:**

Clean Architecture হলো এমন একটি software architectural approach, যেখানে business logic-কে infrastructure এবং external dependencies থেকে আলাদা রাখা হয়।

Common Layers:

1. Domain
2. Application
3. Infrastructure
4. Presentation / API

### Q126. What is the responsibility of each layer?

**Domain:**

- Entities
- Value Objects
- Domain Rules
- Domain Events

**Application:**

- Use Cases
- DTOs
- Interfaces
- Business Workflow
- Validation

**Infrastructure:**

- ADO.NET
- SQL Server
- Repository Implementations
- External Services

**API:**

- Controllers
- Middleware
- Authentication
- Dependency Injection Configuration

### Q127. What is CQRS?

**Answer:**

CQRS-এর পূর্ণরূপ Command Query Responsibility Segregation।

Read operation এবং Write operation আলাদা করার architectural pattern।

**Command:**

Data পরিবর্তন করে।

**Query:**

Data retrieve করে।

Example:

```text
CreateEmployeeCommand
GetEmployeeByIdQuery
UpdateEmployeeCommand
DeleteEmployeeCommand
```

### Q128. What is DDD?

**Answer:**

DDD-এর পূর্ণরূপ Domain-Driven Design।

Complex business domain-এর উপর ভিত্তি করে software design করার approach।

Important Concepts:

- Entity
- Value Object
- Aggregate
- Aggregate Root
- Repository
- Domain Service
- Bounded Context

### Q129. What is Aggregate Root?

**Answer:**

DDD-তে Aggregate-এর মধ্যে থাকা entities এবং value objects-কে consistency boundary-এর মধ্যে রাখে Aggregate Root।

External code সাধারণত Aggregate Root-এর মাধ্যমে Aggregate modify করে।

### Q130. What is the difference between Monolithic and Microservices Architecture?

| Monolithic | Microservices |
|---|---|
| Single deployable application | Multiple independently deployable services |
| Simpler initial deployment | Distributed deployment |
| Usually shared process | Independent processes |
| Scaling may be application-wide | Services independently scale করা যায় |
| Simpler communication | Network communication complexity |

### Q131. What is API Gateway?

**Answer:**

API Gateway হলো client এবং backend services-এর মধ্যবর্তী entry point।

Responsibilities:

- Request Routing
- Authentication Integration
- Rate Limiting
- Load Balancing
- Logging

উদাহরণ: YARP, Ocelot।

---

# 12. Performance & Security

### Q132. What is Caching?

**Answer:**

Frequently accessed data temporary storage-এ রাখাকে Caching বলে।

Types:

- In-Memory Cache
- Distributed Cache
- Response Cache
- Output Cache

### Q133. What is Redis?

**Answer:**

Redis হলো an in-memory data store।

ব্যবহার:

- Distributed Cache
- Session Storage
- Rate Limiting Support
- Pub/Sub
- Temporary Data Storage

### Q134. What is Rate Limiting?

**Answer:**

একটি নির্দিষ্ট সময়ের মধ্যে একটি client কতগুলো request করতে পারবে তা limit করাকে Rate Limiting বলে।

এটি API abuse এবং excessive requests নিয়ন্ত্রণ করতে সাহায্য করে।

### Q135. What is Pagination?

**Answer:**

অনেক data একবারে return না করে ছোট ছোট page-এ ভাগ করে return করাকে Pagination বলে।

```sql
SELECT *
FROM Employees
ORDER BY EmployeeId
OFFSET 0 ROWS
FETCH NEXT 10 ROWS ONLY;
```

### Q136. What is Logging?

**Answer:**

Application-এর execution, error এবং important events record করার process হলো Logging।

ASP.NET Core-এ ILogger ব্যবহার করা হয়।

```csharp
public class EmployeeService
{
    private readonly ILogger<EmployeeService> _logger;

    public EmployeeService(ILogger<EmployeeService> logger)
    {
        _logger = logger;
    }

    public void Process()
    {
        _logger.LogInformation("Employee processing started");
    }
}
```

### Q137. What is XSS?

**Answer:**

Cross-Site Scripting হলো একটি web security vulnerability, যেখানে attacker malicious script inject করতে পারে।

Prevention:

- Output Encoding
- Safe Template Binding
- Content Security Policy
- Avoid unsafe HTML rendering

### Q138. What is CSRF?

**Answer:**

Cross-Site Request Forgery হলো এমন attack যেখানে authenticated user's browser দিয়ে তার অজান্তে unwanted request পাঠানো হয়।

Cookie-based authentication-এ anti-forgery protection এবং SameSite cookie configuration গুরুত্বপূর্ণ।

### Q139. What is HTTPS?

**Answer:**

HTTPS হলো HTTP-এর secure version, যা TLS encryption ব্যবহার করে client ও server-এর মধ্যে communication protect করে।

### Q140. What is Database Backup and Restore?

**Answer:**

Backup হলো database-এর copy তৈরি করা।

Restore হলো backup থেকে database পুনরুদ্ধার করা।

SQL Server:

```sql
BACKUP DATABASE DemoDb
TO DISK = 'C:\Backup\DemoDb.bak'
WITH INIT;
```

```sql
RESTORE DATABASE DemoDb
FROM DISK = 'C:\Backup\DemoDb.bak';
```

---

# 13. Real-world Scenario Based Questions

### Q141. Your API is returning 500 Internal Server Error. How will you debug it?

**Answer:**

আমি নিচের steps অনুসরণ করব:

1. API logs check করব।
2. Exception details identify করব।
3. Database connection check করব।
4. Stored Procedure execute করে দেখব।
5. Request payload validate করব।
6. Dependency Injection configuration check করব।
7. Development environment-এ debugger ব্যবহার করব।

Production-এ sensitive exception details client-কে return করব না।

### Q142. Your Angular application is getting 401 Unauthorized. What will you check?

**Answer:**

1. Access token আছে কি না।
2. Token expired কি না।
3. Authorization header পাঠানো হচ্ছে কি না।
4. Bearer scheme ঠিক আছে কি না।
5. JWT issuer এবং audience ঠিক আছে কি না।
6. Signing key ঠিক আছে কি না।
7. Authentication middleware configured কি না।
8. Middleware order ঠিক আছে কি না।

Example:

```http
Authorization: Bearer your_access_token
```

### Q143. How do you prevent SQL Injection in ADO.NET?

**Answer:**

Parameterized Query এবং Stored Procedure ব্যবহার করব।

```csharp
command.CommandText =
    "SELECT * FROM Users WHERE Id = @Id";

command.Parameters.Add("@Id", SqlDbType.Int).Value = id;
```

User input কখনো সরাসরি SQL string-এর সঙ্গে concatenate করব না।

### Q144. How do you improve a slow SQL query?

**Answer:**

1. Execution Plan analyze করব।
2. Appropriate indexes তৈরি করব।
3. Unnecessary SELECT * এড়িয়ে চলব।
4. Proper JOIN ব্যবহার করব।
5. Query statistics analyze করব।
6. Unnecessary subqueries কমাব।
7. Data filtering optimize করব।
8. Parameter sniffing এবং blocking investigate করব।

### Q145. How do you handle multiple users booking the same seat?

**Answer:**

একই seat একাধিক user book করা prevent করতে database-level concurrency control ব্যবহার করব।

Possible solution:

1. Seat status: Available, Locked, Booked।
2. Atomic Stored Procedure।
3. Transaction।
4. Conditional UPDATE।
5. Unique constraint on confirmed booking।
6. Lock expiration mechanism।

Example:

```sql
UPDATE Seats
SET Status = 'Locked'
WHERE SeatId = @SeatId
  AND Status = 'Available';

IF @@ROWCOUNT = 0
BEGIN
    THROW 50001, 'Seat is not available', 1;
END;
```

### Q146. How do you handle multiple database operations atomically?

**Answer:**

ADO.NET SqlTransaction ব্যবহার করব।

যেমন Donation System-এ:

1. Donation Insert
2. Payment Insert
3. Ledger Entry Insert
4. Fund Balance Update

সব operation সফল হলে Commit করব। কোনো operation fail হলে Rollback করব।

### Q147. How do you secure an ASP.NET Core Web API?

**Answer:**

- JWT Authentication
- Role/Policy-Based Authorization
- HTTPS
- Input Validation
- Parameterized SQL
- Password Hashing
- Rate Limiting
- CORS Configuration
- Secure Secrets Management
- Proper Exception Handling
- Audit Logging

### Q148. How do you improve Angular application performance?

**Answer:**

1. Lazy Loading ব্যবহার করব।
2. OnPush Change Detection ব্যবহার করব।
3. Signals এবং RxJS appropriately ব্যবহার করব।
4. Track tracking key ব্যবহার করে @for optimize করব।
5. Unnecessary subscriptions avoid করব।
6. Bundle size optimize করব।
7. Image optimization করব।
8. API pagination ব্যবহার করব।

### Q149. How do you implement Role-Based Access Control in Angular and ASP.NET Core?

**Answer:**

Backend:

```csharp
[Authorize(Roles = "Admin")]
[HttpGet("users")]
public IActionResult GetUsers()
{
    return Ok();
}
```

Frontend:

```typescript
{
  path: 'admin',
  canActivate: [authGuard, roleGuard],
  loadComponent: () =>
    import('./admin.component')
      .then(m => m.AdminComponent)
}
```

Angular UI navigation control করবে, কিন্তু actual security enforce করবে backend।

### Q150. How do you design a professional .NET application?

**Answer:**

আমি সাধারণত নিচের architecture অনুসরণ করব:

```text
MySolution
│
├── MyProject.Domain
│   ├── Entities
│   ├── Enums
│   └── Interfaces
│
├── MyProject.Application
│   ├── DTOs
│   ├── Services
│   ├── Features
│   └── Validators
│
├── MyProject.Infrastructure
│   ├── Data
│   ├── Repositories
│   ├── StoredProcedures
│   └── ExternalServices
│
├── MyProject.API
│   ├── Controllers
│   ├── Middleware
│   ├── Extensions
│   └── Program.cs
│
└── MyProject.Tests
```

Technology:

- ASP.NET Core Web API
- ADO.NET
- SQL Server
- Stored Procedures
- Angular 20
- JWT Authentication
- Clean Architecture
- SOLID Principles
- Unit Testing

---

# 🎯 Bonus: Important SQL Queries for Interviews

### 1. Find the second highest salary

```sql
SELECT MAX(Salary) AS SecondHighestSalary
FROM Employees
WHERE Salary < (
    SELECT MAX(Salary)
    FROM Employees
);
```

### 2. Find duplicate records

```sql
SELECT Email, COUNT(*) AS Total
FROM Employees
GROUP BY Email
HAVING COUNT(*) > 1;
```

### 3. Find employees with salary greater than average

```sql
SELECT *
FROM Employees
WHERE Salary > (
    SELECT AVG(Salary)
    FROM Employees
);
```

### 4. Find the highest salary in each department

```sql
SELECT DepartmentId, MAX(Salary) AS HighestSalary
FROM Employees
GROUP BY DepartmentId;
```

### 5. Find employees who joined in the last 30 days

```sql
SELECT *
FROM Employees
WHERE JoiningDate >= DATEADD(DAY, -30, GETDATE());
```

### 6. Find the Nth highest salary using DENSE_RANK

```sql
WITH SalaryRank AS
(
    SELECT
        Name,
        Salary,
        DENSE_RANK() OVER (
            ORDER BY Salary DESC
        ) AS RankNo
    FROM Employees
)
SELECT *
FROM SalaryRank
WHERE RankNo = 3;
```

### 7. Update employee salary

```sql
UPDATE Employees
SET Salary = 50000
WHERE EmployeeId = 1;
```

### 8. Delete duplicate rows using ROW_NUMBER

```sql
WITH DuplicateRecords AS
(
    SELECT *,
        ROW_NUMBER() OVER (
            PARTITION BY Email
            ORDER BY EmployeeId
        ) AS RowNo
    FROM Employees
)
DELETE FROM DuplicateRecords
WHERE RowNo > 1;
```

---

# 💼 Important Interview Preparation Checklist

## C# & .NET

- [ ] OOP Principles
- [ ] Value Type vs Reference Type
- [ ] Interface vs Abstract Class
- [ ] Async/Await
- [ ] LINQ
- [ ] Dependency Injection
- [ ] Exception Handling
- [ ] Garbage Collection
- [ ] SOLID Principles

## ASP.NET Core

- [ ] Middleware
- [ ] Dependency Injection
- [ ] Web API
- [ ] REST Principles
- [ ] Model Binding
- [ ] DTO
- [ ] JWT
- [ ] Authentication
- [ ] Authorization
- [ ] API Versioning

## ADO.NET & SQL Server

- [ ] SqlConnection
- [ ] SqlCommand
- [ ] SqlDataReader
- [ ] SqlDataAdapter
- [ ] SqlTransaction
- [ ] Stored Procedures
- [ ] SQL Injection
- [ ] JOIN
- [ ] Indexing
- [ ] CTE
- [ ] Views
- [ ] Triggers
- [ ] ACID
- [ ] Deadlock

## Angular 20

- [ ] Components
- [ ] Standalone Architecture
- [ ] Services
- [ ] Dependency Injection
- [ ] Routing
- [ ] Lazy Loading
- [ ] Signals
- [ ] RxJS
- [ ] HttpClient
- [ ] Interceptors
- [ ] Route Guards
- [ ] Reactive Forms
- [ ] Change Detection

## Advanced

- [ ] Clean Architecture
- [ ] CQRS
- [ ] DDD
- [ ] Repository Pattern
- [ ] Unit of Work
- [ ] Design Patterns
- [ ] Caching
- [ ] Redis
- [ ] Rate Limiting
- [ ] Docker
- [ ] Microservices
- [ ] Unit Testing

---

# 📚 Recommended Learning Resources

| Resource | Link |
|---|---|
| C# Documentation | https://learn.microsoft.com/en-us/dotnet/csharp/ |
| .NET Documentation | https://learn.microsoft.com/en-us/dotnet/ |
| ASP.NET Core | https://learn.microsoft.com/en-us/aspnet/core/ |
| ADO.NET | https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ |
| SQL Server | https://learn.microsoft.com/en-us/sql/sql-server/ |
| Angular | https://angular.dev/ |
| TypeScript | https://www.typescriptlang.org/docs/ |
| OWASP | https://owasp.org/ |

---

# ⭐ Final Interview Tips

1. শুধু definition মুখস্থ করবে না, practical example বুঝবে।
2. প্রতিটি concept-এর ছোট code example practice করবে।
3. SQL query প্রতিদিন লিখবে।
4. নিজের project-এর architecture explain করতে শিখবে।
5. API request থেকে database পর্যন্ত পুরো data flow বুঝবে।
6. Authentication এবং Authorization-এর পার্থক্য পরিষ্কার রাখবে।
7. Real-world problem solving practice করবে।
8. Interview-তে না জানলে ভুল উত্তর না দিয়ে নিজের বোঝার সীমা পরিষ্কারভাবে বলবে।

**Remember:**

> A good developer writes code. A professional developer understands why, how, and when to use it.

---

**Maintained for:** Full Stack .NET Developer Interview Preparation  
**Stack:** ASP.NET Core | Angular 20 | ADO.NET | SQL Server  
**Level:** Basic → Intermediate → Advanced
