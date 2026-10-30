# 🚀 Full Stack .NET Developer Interview Cheat Sheet

**Basic to Advanced Interview Questions & Answers**

A comprehensive interview preparation guide for Full Stack .NET Developers.

**Technology Stack:**
- C# | .NET 8 | ASP.NET Core
- ASP.NET Core MVC | Web API
- Angular 20 | TypeScript | JavaScript
- ADO.NET | SQL Server | SSMS
- Stored Procedures | Dapper
- Entity Framework Core
- JWT | REST API
- Clean Architecture | SOLID Principles

---

## 📚 Table of Contents

1. [C# Fundamentals](#1-c-fundamentals)
2. [OOP Concepts](#2-object-oriented-programming-oop)
3. [Advanced C#](#3-advanced-c)
4. [.NET & ASP.NET Core](#4-net--aspnet-core)
5. [ASP.NET Core Web API](#5-aspnet-core-web-api)
6. [Dependency Injection & Middleware](#6-dependency-injection--middleware)
7. [ADO.NET](#7-adonet)
8. [SQL Server & SSMS](#8-sql-server--ssms)
9. [Stored Procedures & SQL Optimization](#9-stored-procedures--sql-optimization)
10. [Entity Framework Core & Dapper](#10-entity-framework-core--dapper)
11. [Angular 20](#11-angular-20)
12. [TypeScript & JavaScript](#12-typescript--javascript)
13. [RxJS & Observables](#13-rxjs--observables)
14. [Authentication & Authorization](#14-authentication--authorization)
15. [SOLID Principles & Design Patterns](#15-solid-principles--design-patterns)
16. [Clean Architecture](#16-clean-architecture)
17. [REST API & HTTP](#17-rest-api--http)
18. [Performance & Security](#18-performance--security)
19. [Git, Docker & Deployment](#19-git-docker--deployment)
20. [Scenario-Based Interview Questions](#20-scenario-based-interview-questions)

---

# 1. C# Fundamentals

### Q1. What is C#?

**Answer:**

C# is a modern, object-oriented, type-safe programming language developed by Microsoft.

It is commonly used for:
- Web Application Development
- Desktop Applications
- Web APIs
- Cloud Applications
- Enterprise Software

### Q2. What is the difference between .NET and C#?

**Answer:**

| C# | .NET |
|---|---|
| Programming language | Development platform |
| Used to write code | Used to execute applications |
| Provides language syntax | Provides runtime and libraries |
| Example: `int age = 25;` | Example: ASP.NET Core |

### Q3. What is CLR?

**Answer:**

CLR stands for Common Language Runtime.

It is the execution engine of .NET.

Responsibilities:
- Memory Management
- Garbage Collection
- Exception Handling
- Type Safety
- JIT Compilation
- Thread Management

### Q4. What is CTS?

**Answer:**

CTS stands for Common Type System.

It defines how data types are declared and used in the .NET runtime.

Example:

`int` in C# maps to `System.Int32`.

### Q5. What is CLS?

**Answer:**

CLS stands for Common Language Specification.

It defines a set of rules that .NET languages should follow to maintain interoperability.

### Q6. What is the difference between Value Type and Reference Type?

**Answer:**

| Value Type | Reference Type |
|---|---|
| Stores the value directly | Stores a reference to an object |
| Commonly stored inline | Objects commonly allocated on managed heap |
| Assignment copies the value | Assignment copies the reference |
| Example: int, bool, struct | Example: class, array, string |

Example:

```csharp
int a = 10;
int b = a;

b = 20;

Console.WriteLine(a); // 10
```

Reference example:

```csharp
class Employee
{
    public string Name { get; set; }
}

var emp1 = new Employee { Name = "Abdullah" };
var emp2 = emp1;

emp2.Name = "Rahim";

Console.WriteLine(emp1.Name); // Rahim
```

### Q7. What is the difference between var, dynamic and object?

**Answer:**

| var | dynamic | object |
|---|---|---|
| Compile-time type inference | Runtime binding | Base type of all .NET types |
| Type is fixed | Type can change | Requires casting for specific operations |
| Compile-time checking | Runtime checking | Compile-time checking after casting |

Example:

```csharp
var name = "Abdullah";

dynamic value = 100;
value = "Hello";

object data = "Hello";

string result = (string)data;
```

### Q8. What is the difference between const and readonly?

**Answer:**

| const | readonly |
|---|---|
| Compile-time constant | Runtime value can be assigned during initialization |
| Must be initialized at declaration | Can be initialized at declaration or constructor |
| Implicitly static for class members | Can be instance-level or static |
| Cannot be changed | Cannot be changed after allowed initialization |

```csharp
public const double PI = 3.1416;

public readonly int Id;

public Employee(int id)
{
    Id = id;
}
```

### Q9. What is boxing and unboxing?

**Answer:**

Boxing converts a value type into an object reference.

Unboxing converts an object back into a value type.

```csharp
int number = 100;

// Boxing
object obj = number;

// Unboxing
int result = (int)obj;
```

### Q10. What is the difference between == and Equals()?

**Answer:**

- `==` is an operator whose behavior depends on the type and its operator overload.
- `Equals()` is a method that checks equality according to the type's implementation.

For reference types, `==` normally checks reference identity unless overloaded.

For strings, both normally compare string values.

### Q11. What is string and StringBuilder?

**Answer:**

String is immutable. Every modification creates a new string.

StringBuilder is mutable and is useful for repeated string modifications.

```csharp
using System.Text;

StringBuilder sb = new StringBuilder();

sb.Append("Hello");
sb.Append(" World");

Console.WriteLine(sb.ToString());
```

### Q12. What is the difference between break, continue and return?

**Answer:**

- `break`: Terminates the current loop or switch.
- `continue`: Skips the current loop iteration.
- `return`: Exits the current method and optionally returns a value.

---

# 2. Object-Oriented Programming (OOP)

### Q13. What is OOP?

**Answer:**

Object-Oriented Programming is a programming approach based on objects and classes.

Four main pillars:

1. Encapsulation
2. Inheritance
3. Polymorphism
4. Abstraction

### Q14. What is Encapsulation?

**Answer:**

Encapsulation means protecting internal data and providing controlled access through methods or properties.

```csharp
public class BankAccount
{
    private decimal balance;

    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Invalid amount");

        balance += amount;
    }

    public decimal GetBalance()
    {
        return balance;
    }
}
```

### Q15. What is Inheritance?

**Answer:**

Inheritance allows one class to reuse the properties and methods of another class.

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

### Q16. What is Polymorphism?

**Answer:**

Polymorphism means one interface or method name can have different implementations.

Two types:

- Compile-time polymorphism: Method Overloading
- Runtime polymorphism: Method Overriding

### Q17. What is Method Overloading?

**Answer:**

Multiple methods have the same name but different parameter lists.

```csharp
public int Add(int a, int b)
{
    return a + b;
}

public int Add(int a, int b, int c)
{
    return a + b + c;
}
```

### Q18. What is Method Overriding?

**Answer:**

A child class provides a new implementation of a virtual or abstract method from its parent class.

```csharp
public class Animal
{
    public virtual void Sound()
    {
        Console.WriteLine("Animal Sound");
    }
}

public class Dog : Animal
{
    public override void Sound()
    {
        Console.WriteLine("Bark");
    }
}
```

### Q19. What is Abstraction?

**Answer:**

Abstraction means hiding implementation details and exposing only essential functionality.

It can be implemented using abstract classes and interfaces.

### Q20. What is the difference between Abstract Class and Interface?

**Answer:**

| Abstract Class | Interface |
|---|---|
| Can have instance fields | Cannot have instance fields |
| Can have constructors | Cannot have instance constructors |
| Can contain abstract and implemented methods | Can define contracts and provide default implementations |
| A class can inherit one class | A class can implement multiple interfaces |
| Supports shared state | Primarily defines capabilities/contracts |

### Q21. What is the difference between virtual, override and new?

**Answer:**

- `virtual`: Allows a method to be overridden.
- `override`: Replaces the inherited virtual or abstract implementation.
- `new`: Hides an inherited member rather than overriding it.

### Q22. What is a sealed class?

**Answer:**

A sealed class cannot be inherited.

```csharp
public sealed class SecurityService
{
}
```

### Q23. What is a static class?

**Answer:**

A static class cannot be instantiated and contains only static members.

```csharp
public static class MathHelper
{
    public static int Add(int a, int b)
    {
        return a + b;
    }
}
```

### Q24. What is Constructor?

**Answer:**

A constructor is a special method that initializes an object.

Types:
- Default Constructor
- Parameterized Constructor
- Static Constructor
- Private Constructor

### Q25. What is Composition vs Inheritance?

**Answer:**

Inheritance represents an "is-a" relationship.

Composition represents a "has-a" relationship.

Example:

- Dog is an Animal → Inheritance.
- Car has an Engine → Composition.

Composition is often preferred when flexible behavior and loose coupling are important.

---

# 3. Advanced C#

### Q26. What is Exception Handling?

**Answer:**

Exception handling manages runtime errors using:

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
    Console.WriteLine("Execution completed");
}
```

### Q27. What is the difference between throw and throw ex?

**Answer:**

`throw;` preserves the original exception stack trace.

`throw ex;` resets the stack trace from the current throw location.

Prefer:

```csharp
catch (Exception)
{
    throw;
}
```

### Q28. What is Delegate?

**Answer:**

A delegate is a type-safe reference to a method.

```csharp
public delegate int Operation(int a, int b);

static int Add(int a, int b)
{
    return a + b;
}

Operation operation = Add;

Console.WriteLine(operation(10, 20));
```

### Q29. What is an Event?

**Answer:**

An event is a notification mechanism based on delegates.

It allows one object to notify other objects when something happens.

### Q30. What is LINQ?

**Answer:**

LINQ stands for Language Integrated Query.

It provides a consistent way to query collections and other data sources.

```csharp
var numbers = new List<int>
{
    10, 20, 30, 40, 50
};

var result = numbers
    .Where(x => x > 20)
    .OrderBy(x => x)
    .ToList();
```

Important LINQ methods:

- Where()
- Select()
- FirstOrDefault()
- SingleOrDefault()
- Any()
- All()
- Count()
- OrderBy()
- GroupBy()
- Join()
- ToList()

### Q31. What is IEnumerable vs IQueryable?

**Answer:**

| IEnumerable | IQueryable |
|---|---|
| Works with in-memory collections | Supports query providers |
| LINQ operations generally execute in memory | Provider can translate expressions into SQL |
| Uses delegates for LINQ-to-Objects | Uses expression trees |
| Suitable for in-memory data | Suitable for provider-based querying |

### Q32. What is async and await?

**Answer:**

`async` and `await` support asynchronous programming.

They help avoid blocking a thread while waiting for asynchronous operations such as database or network I/O.

```csharp
public async Task<string> GetDataAsync()
{
    await Task.Delay(1000);

    return "Data Loaded";
}
```

### Q33. What is Task vs Thread?

**Answer:**

| Task | Thread |
|---|---|
| Represents an asynchronous operation | Represents an execution thread |
| Usually managed through ThreadPool | Can be created directly |
| Supports async/await | Lower-level threading abstraction |
| Suitable for most application-level async work | Useful for specialized threading scenarios |

### Q34. What is Task.WhenAll()?

**Answer:**

It asynchronously waits for multiple tasks to complete.

```csharp
Task<string> task1 = GetUserAsync();
Task<string> task2 = GetOrderAsync();

string[] results = await Task.WhenAll(task1, task2);
```

### Q35. What is CancellationToken?

**Answer:**

CancellationToken allows an operation to receive a cancellation request.

```csharp
public async Task GetDataAsync(CancellationToken token)
{
    await Task.Delay(5000, token);
}
```

### Q36. What is Garbage Collection?

**Answer:**

Garbage Collection automatically manages memory by reclaiming memory occupied by unreachable managed objects.

.NET GC uses generations:

- Generation 0
- Generation 1
- Generation 2

Large Object Heap (LOH) is used for sufficiently large allocations.

### Q37. What is IDisposable?

**Answer:**

IDisposable provides a mechanism to release unmanaged resources or owned disposable resources.

```csharp
using var connection = new SqlConnection(connectionString);
```

The using statement ensures disposal when execution leaves the scope.

### Q38. What is Nullable Type?

**Answer:**

Nullable value types can represent both a normal value and null.

```csharp
int? age = null;

if (age.HasValue)
{
    Console.WriteLine(age.Value);
}
```

### Q39. What is Record in C#?

**Answer:**

A record is a reference type by default that provides value-based equality and convenient immutable-style data modeling.

```csharp
public record Employee(int Id, string Name);
```

### Q40. What is Dependency Injection?

**Answer:**

Dependency Injection is a technique where dependencies are provided to a class rather than created directly inside it.

It improves:
- Loose Coupling
- Testability
- Maintainability
- Reusability

---

# 4. .NET & ASP.NET Core

### Q41. What is .NET?

**Answer:**

.NET is a free, open-source, cross-platform development platform.

It supports:
- Web API
- MVC
- Web Applications
- Desktop Applications
- Cloud Services
- Background Services

### Q42. What is ASP.NET Core?

**Answer:**

ASP.NET Core is a cross-platform web framework built on .NET.

It is used to develop:

- REST APIs
- MVC Applications
- Web Applications
- Real-time Applications using SignalR

### Q43. What is the difference between .NET Framework and .NET?

**Answer:**

| .NET Framework | Modern .NET |
|---|---|
| Primarily Windows-focused | Cross-platform |
| Legacy application ecosystem | Modern application development |
| Traditional ASP.NET | ASP.NET Core |
| Older development platform | Actively evolving platform |

### Q44. What is Kestrel?

**Answer:**

Kestrel is the cross-platform web server used by ASP.NET Core.

It receives HTTP requests and processes them through the ASP.NET Core application pipeline.

### Q45. What is Program.cs?

**Answer:**

Program.cs is the application entry point in modern ASP.NET Core applications.

It configures services and middleware and starts the web application.

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

var app = builder.Build();

app.MapControllers();

app.Run();
```

### Q46. What is appsettings.json?

**Answer:**

It stores application configuration settings.

Example:

```json
{
  "ConnectionStrings": {
    "Default": "Server=.;Database=MyDb;Trusted_Connection=True;TrustServerCertificate=True;"
  },
  "Jwt": {
    "Issuer": "MyAPI",
    "Audience": "MyAngularApp"
  }
}
```

Production secrets should be stored securely, not committed to GitHub.

### Q47. What is the difference between MVC and Web API?

**Answer:**

| MVC | Web API |
|---|---|
| Commonly returns Views | Commonly returns data |
| Supports Razor Views | Supports JSON/XML responses |
| Used for server-rendered UI | Used for frontend/mobile integration |
| Controller + Model + View | Controller + Request/Response Models |

### Q48. What is Model Binding?

**Answer:**

Model Binding maps incoming HTTP request data to action parameters or model properties.

```csharp
[HttpPost]
public IActionResult Create([FromBody] EmployeeDto employee)
{
    return Ok(employee);
}
```

Binding sources include:

- [FromBody]
- [FromRoute]
- [FromQuery]
- [FromHeader]
- [FromForm]

### Q49. What is Model Validation?

**Answer:**

Model Validation checks whether incoming request data satisfies defined validation rules.

```csharp
public class EmployeeDto
{
    [Required]
    public string Name { get; set; }

    [Range(18, 60)]
    public int Age { get; set; }
}
```

### Q50. What is IConfiguration?

**Answer:**

IConfiguration provides access to application configuration values.

```csharp
var connectionString =
    configuration.GetConnectionString("Default");
```

### Q51. What is IOptions?

**Answer:**

IOptions binds configuration sections to strongly typed classes.

```csharp
public class JwtOptions
{
    public string Issuer { get; set; }
    public string Audience { get; set; }
}
```

```csharp
builder.Services.Configure<JwtOptions>(
    builder.Configuration.GetSection("Jwt"));
```

### Q52. What is Hosted Service?

**Answer:**

A hosted service runs background tasks within an ASP.NET Core application.

Common interfaces:

- IHostedService
- BackgroundService

Use cases:
- Scheduled jobs
- Queue processing
- Background notifications
- Periodic cleanup

---

# 5. ASP.NET Core Web API

### Q53. What is REST API?

**Answer:**

REST stands for Representational State Transfer.

It is an architectural style for designing network-based applications.

REST APIs commonly use HTTP methods and resource-based URLs.

### Q54. What are HTTP Methods?

**Answer:**

| Method | Purpose |
|---|---|
| GET | Retrieve data |
| POST | Create a resource or perform an operation |
| PUT | Replace a resource |
| PATCH | Partially update a resource |
| DELETE | Delete a resource |

### Q55. What are HTTP Status Codes?

**Answer:**

| Code | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 202 | Accepted |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 422 | Unprocessable Content |
| 429 | Too Many Requests |
| 500 | Internal Server Error |
| 503 | Service Unavailable |

### Q56. What is the difference between 401 and 403?

**Answer:**

- 401 Unauthorized: Authentication is missing or invalid.
- 403 Forbidden: The user is authenticated but does not have permission.

### Q57. What is DTO?

**Answer:**

DTO stands for Data Transfer Object.

It transfers data between application layers or across API boundaries.

```csharp
public class EmployeeDto
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

Benefits:
- Prevents exposing internal entities
- Controls API response
- Improves security
- Separates database models from API contracts

### Q58. What is IActionResult vs ActionResult<T>?

**Answer:**

`IActionResult` allows an action to return different HTTP responses.

`ActionResult<T>` provides a typed response along with HTTP result support.

```csharp
public ActionResult<EmployeeDto> GetEmployee()
{
    var employee = new EmployeeDto
    {
        Id = 1,
        Name = "Abdullah"
    };

    return Ok(employee);
}
```

### Q59. What is API Versioning?

**Answer:**

API Versioning allows multiple API versions to coexist.

Example:

```text
/api/v1/employees
/api/v2/employees
```

It helps maintain backward compatibility.

### Q60. What is CORS?

**Answer:**

CORS stands for Cross-Origin Resource Sharing.

It controls which origins can access an API from a browser.

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AngularPolicy", policy =>
    {
        policy.WithOrigins("http://localhost:4200")
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

var app = builder.Build();

app.UseCors("AngularPolicy");
```

CORS is not an authentication or authorization mechanism.

### Q61. What is Swagger / OpenAPI?

**Answer:**

OpenAPI is a specification for describing HTTP APIs.

Swagger is a collection of tools for working with OpenAPI documents, including interactive API documentation.

It helps developers:
- Test endpoints
- Understand request/response models
- View API documentation

---

# 6. Dependency Injection & Middleware

### Q62. What is Dependency Injection in ASP.NET Core?

**Answer:**

ASP.NET Core provides a built-in DI container.

```csharp
builder.Services.AddScoped<IEmployeeService, EmployeeService>();
```

### Q63. What are the three DI lifetimes?

**Answer:**

| Lifetime | Description | Example |
|---|---|---|
| Transient | New instance each resolution | Lightweight services |
| Scoped | One instance per request scope | Database-related services |
| Singleton | One instance for application lifetime | Thread-safe shared services |

### Q64. What is Middleware?

**Answer:**

Middleware is software that is assembled into the HTTP request pipeline.

Each middleware can:
- Process a request
- Pass control to the next middleware
- Modify a response
- Short-circuit the pipeline

### Q65. Explain Middleware Pipeline.

**Answer:**

Example:

```csharp
app.UseExceptionHandler();

app.UseHttpsRedirection();

app.UseRouting();

app.UseCors("AngularPolicy");

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();
```

Middleware order matters.

Authentication must run before Authorization.

### Q66. What is Custom Middleware?

**Answer:**

Custom middleware allows developers to implement their own request/response processing logic.

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;

    public RequestLoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine(context.Request.Path);

        await _next(context);
    }
}
```

Registration:

```csharp
app.UseMiddleware<RequestLoggingMiddleware>();
```

### Q67. What is Service Lifetime Problem?

**Answer:**

A common DI issue occurs when a Singleton service directly depends on a Scoped service.

This is called a captive dependency.

For example, a Singleton should not directly capture a Scoped DbContext or request-scoped repository.

---

# 7. ADO.NET

### Q68. What is ADO.NET?

**Answer:**

ADO.NET is a data access technology in .NET used to communicate with databases.

It provides:
- SqlConnection
- SqlCommand
- SqlDataReader
- SqlDataAdapter
- DataSet
- DataTable
- SqlTransaction

### Q69. What is SqlConnection?

**Answer:**

SqlConnection establishes a connection to SQL Server.

```csharp
using Microsoft.Data.SqlClient;

using var connection =
    new SqlConnection(connectionString);

await connection.OpenAsync();
```

### Q70. What is SqlCommand?

**Answer:**

SqlCommand executes SQL statements or stored procedures.

```csharp
using var command =
    new SqlCommand("SELECT * FROM Employees", connection);
```

### Q71. What is SqlDataReader?

**Answer:**

SqlDataReader reads data from SQL Server using a forward-only, read-only stream.

```csharp
using var reader = await command.ExecuteReaderAsync();

while (await reader.ReadAsync())
{
    Console.WriteLine(reader["Name"]);
}
```

### Q72. What is SqlDataAdapter?

**Answer:**

SqlDataAdapter acts as a bridge between a database and disconnected DataSet/DataTable objects.

```csharp
using var adapter = new SqlDataAdapter(command);

DataTable table = new DataTable();

adapter.Fill(table);
```

### Q73. What is the difference between ExecuteReader, ExecuteScalar and ExecuteNonQuery?

**Answer:**

| Method | Purpose |
|---|---|
| ExecuteReader | Returns multiple rows |
| ExecuteScalar | Returns first column of first row |
| ExecuteNonQuery | Executes commands that do not return a result set |

Examples:

```csharp
// Multiple rows
using var reader = command.ExecuteReader();

// Single value
var count = command.ExecuteScalar();

// Insert / Update / Delete
int affectedRows = command.ExecuteNonQuery();
```

### Q74. What is Connected vs Disconnected Architecture?

**Answer:**

Connected Architecture:
- Requires an active connection while reading data.
- Example: SqlDataReader.

Disconnected Architecture:
- Data is loaded and can be manipulated without maintaining an open connection.
- Example: DataSet and DataTable.

### Q75. What is CommandType.StoredProcedure?

**Answer:**

It tells SqlCommand to execute a stored procedure.

```csharp
using var command = new SqlCommand(
    "sp_GetEmployees",
    connection);

command.CommandType = CommandType.StoredProcedure;
```

### Q76. How do you pass parameters in ADO.NET?

**Answer:**

```csharp
using var command = new SqlCommand(
    "sp_GetEmployeeById",
    connection);

command.CommandType = CommandType.StoredProcedure;

command.Parameters.Add("@Id", SqlDbType.Int).Value = 1;
```

### Q77. Why should we use SQL Parameters?

**Answer:**

SQL parameters:
- Help prevent SQL Injection
- Handle data types correctly
- Avoid unsafe string concatenation
- Improve query plan reuse in suitable cases

Never concatenate user input directly into SQL statements.

### Q78. What is SQL Injection?

**Answer:**

SQL Injection is an attack where malicious SQL input changes the intended database query.

Unsafe:

```csharp
string query =
    "SELECT * FROM Users WHERE Name = '" + name + "'";
```

Safe:

```csharp
using var command = new SqlCommand(
    "SELECT * FROM Users WHERE Name = @Name",
    connection);

command.Parameters.Add("@Name", SqlDbType.NVarChar, 100)
    .Value = name;
```

### Q79. How do you implement Transaction in ADO.NET?

**Answer:**

A transaction ensures multiple database operations succeed or fail together.

```csharp
using var connection = new SqlConnection(connectionString);

await connection.OpenAsync();

using var transaction =
    await connection.BeginTransactionAsync();

try
{
    using var command = new SqlCommand(
        "UPDATE Accounts SET Balance = Balance - @Amount WHERE Id = 1",
        connection,
        (SqlTransaction)transaction);

    command.Parameters.Add("@Amount", SqlDbType.Decimal).Value = 100;

    await command.ExecuteNonQueryAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

### Q80. What is Connection Pooling?

**Answer:**

Connection Pooling reuses physical database connections instead of creating a new physical connection for every request.

It improves performance and reduces connection overhead.

ADO.NET SQL Server providers support connection pooling by default.

### Q81. What is DataTable?

**Answer:**

DataTable represents an in-memory table containing rows and columns.

It is useful for:
- Disconnected data processing
- Data manipulation
- Bulk operations
- Binding tabular data

### Q82. What is SqlBulkCopy?

**Answer:**

SqlBulkCopy efficiently transfers large amounts of data into SQL Server tables.

```csharp
using var bulkCopy = new SqlBulkCopy(connection);

bulkCopy.DestinationTableName = "Employees";

await bulkCopy.WriteToServerAsync(dataTable);
```

### Q83. What is TVP?

**Answer:**

TVP stands for Table-Valued Parameter.

It allows multiple rows to be passed to a stored procedure as a structured parameter.

SQL:

```sql
CREATE TYPE dbo.IdList AS TABLE
(
    Id INT NOT NULL
);
GO

CREATE PROCEDURE dbo.sp_GetEmployeesByIds
    @Ids dbo.IdList READONLY
AS
BEGIN
    SELECT e.*
    FROM Employees e
    INNER JOIN @Ids i ON e.Id = i.Id;
END;
```

C#:

```csharp
var table = new DataTable();

table.Columns.Add("Id", typeof(int));

table.Rows.Add(1);
table.Rows.Add(2);

using var command = new SqlCommand(
    "sp_GetEmployeesByIds", connection);

command.CommandType = CommandType.StoredProcedure;

var parameter = command.Parameters.AddWithValue("@Ids", table);

parameter.SqlDbType = SqlDbType.Structured;
parameter.TypeName = "dbo.IdList";
```

---

# 8. SQL Server & SSMS

### Q84. What is SQL Server?

**Answer:**

SQL Server is a relational database management system developed by Microsoft.

It stores, manages and retrieves structured data using SQL.

### Q85. What is SSMS?

**Answer:**

SSMS stands for SQL Server Management Studio.

It is a graphical tool used to:
- Manage databases
- Write SQL queries
- Create tables
- Execute stored procedures
- Manage indexes
- Monitor SQL Server

### Q86. What is Primary Key?

**Answer:**

A Primary Key uniquely identifies each row in a table.

It cannot contain NULL values and must be unique.

### Q87. What is Foreign Key?

**Answer:**

A Foreign Key establishes a relationship between tables and maintains referential integrity.

### Q88. What is Unique Key?

**Answer:**

A Unique constraint ensures that values in a column or group of columns are unique.

Unlike a Primary Key, a table can have multiple Unique constraints.

### Q89. What is the difference between Primary Key and Unique Key?

**Answer:**

| Primary Key | Unique Constraint |
|---|---|
| One primary key per table | Multiple unique constraints possible |
| Cannot contain NULL | SQL Server allows one NULL in a single-column unique constraint |
| Identifies a row | Enforces uniqueness |

### Q90. What is Normalization?

**Answer:**

Normalization organizes database tables to reduce redundancy and improve data integrity.

Important forms:

- 1NF: Atomic column values and no repeating groups.
- 2NF: 1NF + no partial dependency on a candidate key.
- 3NF: 2NF + no transitive dependency of non-key attributes on candidate keys.

### Q91. What is Denormalization?

**Answer:**

Denormalization intentionally introduces redundancy to improve read performance in certain workloads.

It can increase storage and data consistency complexity.

### Q92. What is JOIN?

**Answer:**

JOIN combines rows from multiple tables based on related columns.

Types:

1. INNER JOIN
2. LEFT JOIN
3. RIGHT JOIN
4. FULL OUTER JOIN
5. CROSS JOIN

Example:

```sql
SELECT
    e.Id,
    e.Name,
    d.DepartmentName
FROM Employees e
INNER JOIN Departments d
    ON e.DepartmentId = d.Id;
```

### Q93. What is the difference between INNER JOIN and LEFT JOIN?

**Answer:**

- INNER JOIN returns matching rows from both tables.
- LEFT JOIN returns all rows from the left table and matching rows from the right table. Unmatched right-side columns become NULL.

### Q94. What is Index?

**Answer:**

An index is a database structure that improves data retrieval performance.

Types:
- Clustered Index
- Nonclustered Index
- Unique Index
- Composite Index
- Filtered Index

### Q95. What is Clustered vs Nonclustered Index?

**Answer:**

| Clustered Index | Nonclustered Index |
|---|---|
| Defines the logical row order of table data | Separate index structure |
| One per table | Multiple per table |
| Leaf level contains table data | Leaf level contains index keys and row locators |

### Q96. What is Composite Index?

**Answer:**

A composite index contains multiple columns.

```sql
CREATE NONCLUSTERED INDEX IX_Employees_Department_Status
ON Employees(DepartmentId, Status);
```

Column order matters because SQL Server uses the index key order when determining whether a query can efficiently seek into the index.

### Q97. What is View?

**Answer:**

A View is a saved SQL query that behaves like a virtual table.

```sql
CREATE VIEW vw_ActiveEmployees
AS
SELECT Id, Name
FROM Employees
WHERE IsActive = 1;
```

### Q98. What is CTE?

**Answer:**

CTE stands for Common Table Expression.

It defines a named temporary result set within a single SQL statement.

```sql
WITH EmployeeCTE AS
(
    SELECT *
    FROM Employees
    WHERE Salary > 30000
)
SELECT *
FROM EmployeeCTE;
```

### Q99. What is the difference between DELETE, TRUNCATE and DROP?

**Answer:**

| DELETE | TRUNCATE | DROP |
|---|---|---|
| Removes selected rows | Removes all rows | Removes the database object |
| Supports WHERE | No WHERE | Deletes object definition |
| Logs row deletions | Uses page deallocation logging | Removes object metadata and data |
| Identity usually not reset | Resets identity | Object no longer exists |

### Q100. What is UNION vs UNION ALL?

**Answer:**

- UNION removes duplicate rows.
- UNION ALL returns all rows, including duplicates.

UNION ALL is generally faster because it does not perform duplicate elimination.

### Q101. What is the difference between WHERE and HAVING?

**Answer:**

- WHERE filters rows before grouping.
- HAVING filters groups after GROUP BY.

```sql
SELECT DepartmentId, COUNT(*) AS Total
FROM Employees
WHERE IsActive = 1
GROUP BY DepartmentId
HAVING COUNT(*) > 5;
```

### Q102. What is NULL in SQL Server?

**Answer:**

NULL represents an unknown or missing value.

Use:

```sql
SELECT *
FROM Employees
WHERE DepartmentId IS NULL;
```

Do not use `= NULL`.

### Q103. What is ISNULL vs COALESCE?

**Answer:**

- ISNULL is a SQL Server-specific function that accepts two arguments.
- COALESCE returns the first non-NULL expression and supports multiple arguments.

```sql
SELECT ISNULL(Phone, 'N/A')
FROM Employees;

SELECT COALESCE(Phone, Email, 'N/A')
FROM Employees;
```

### Q104. What is Identity Column?

**Answer:**

An Identity column automatically generates numeric values.

```sql
CREATE TABLE Employees
(
    Id INT IDENTITY(1,1) PRIMARY KEY,
    Name NVARCHAR(100)
);
```

### Q105. What is ACID?

**Answer:**

ACID represents the four properties of database transactions.

- Atomicity: All operations succeed or fail together.
- Consistency: Data remains valid.
- Isolation: Concurrent transactions are controlled.
- Durability: Committed changes are persistent.

### Q106. What is SQL Transaction?

**Answer:**

A transaction groups database operations into a single logical unit.

```sql
BEGIN TRANSACTION;

BEGIN TRY

    UPDATE Accounts
    SET Balance = Balance - 500
    WHERE Id = 1;

    UPDATE Accounts
    SET Balance = Balance + 500
    WHERE Id = 2;

    COMMIT TRANSACTION;

END TRY
BEGIN CATCH

    IF @@TRANCOUNT > 0
        ROLLBACK TRANSACTION;

    THROW;

END CATCH;
```

### Q107. What is Isolation Level?

**Answer:**

Isolation Level controls how transactions interact with concurrent transactions.

SQL Server supports:

- READ UNCOMMITTED
- READ COMMITTED
- REPEATABLE READ
- SNAPSHOT
- SERIALIZABLE

### Q108. What is Deadlock?

**Answer:**

A deadlock happens when two or more transactions wait for resources held by each other.

SQL Server detects deadlocks and chooses one transaction as the victim.

Prevention:
- Access tables in a consistent order.
- Keep transactions short.
- Create appropriate indexes.
- Avoid unnecessary locks.

---

# 9. Stored Procedures & SQL Optimization

### Q109. What is Stored Procedure?

**Answer:**

A Stored Procedure is a named collection of SQL statements stored in SQL Server.

Benefits:
- Reusability
- Maintainability
- Centralized database logic
- Permission management
- Reduced network round trips
- Potential execution plan reuse

### Q110. How do you create a Stored Procedure?

**Answer:**

```sql
CREATE PROCEDURE sp_GetEmployeeById
    @Id INT
AS
BEGIN
    SET NOCOUNT ON;

    SELECT *
    FROM Employees
    WHERE Id = @Id;
END;
```

Execution:

```sql
EXEC sp_GetEmployeeById @Id = 1;
```

### Q111. What is the difference between Stored Procedure and Function?

**Answer:**

| Stored Procedure | Function |
|---|---|
| Executed using EXEC | Used in SQL expressions or queries |
| Can perform data modification | Restrictions apply to data modification |
| Can return result sets | Returns scalar or table values |
| Supports output parameters | Supports function return values |

### Q112. What is SARGable Query?

**Answer:**

SARGable means Search ARGument ABLE.

A SARGable predicate allows SQL Server to use an index efficiently when appropriate.

Less efficient:

```sql
SELECT *
FROM Employees
WHERE YEAR(CreatedDate) = 2026;
```

More index-friendly:

```sql
SELECT *
FROM Employees
WHERE CreatedDate >= '20260101'
  AND CreatedDate < '20270101';
```

### Q113. What is Execution Plan?

**Answer:**

An Execution Plan shows how SQL Server executes a query.

It helps identify:
- Table Scans
- Index Scans
- Index Seeks
- Key Lookups
- Joins
- Expensive operators

### Q114. What is Index Seek vs Index Scan?

**Answer:**

- Index Seek: Navigates to matching index keys.
- Index Scan: Reads a range or the entire index.

A scan is not always bad. It can be efficient when a large percentage of rows is required.

### Q115. How do you optimize a slow SQL query?

**Answer:**

1. Check the Actual Execution Plan.
2. Check missing or unsuitable indexes.
3. Avoid SELECT * when unnecessary.
4. Use SARGable conditions.
5. Avoid unnecessary joins.
6. Check table statistics.
7. Analyze blocking and deadlocks.
8. Use appropriate pagination.
9. Check parameter-sensitive plans.
10. Measure performance before and after changes.

### Q116. What is OFFSET FETCH?

**Answer:**

It is used for pagination.

```sql
SELECT *
FROM Employees
ORDER BY Id
OFFSET 0 ROWS
FETCH NEXT 10 ROWS ONLY;
```

### Q117. What is ROW_NUMBER()?

**Answer:**

ROW_NUMBER assigns a sequential number to each row based on the specified ordering.

```sql
SELECT
    ROW_NUMBER() OVER (ORDER BY Id) AS RowNum,
    *
FROM Employees;
```

### Q118. What is RANK vs DENSE_RANK vs ROW_NUMBER?

**Answer:**

| Function | Behavior |
|---|---|
| ROW_NUMBER | Unique sequential numbers |
| RANK | Equal values share rank; gaps appear |
| DENSE_RANK | Equal values share rank; no gaps |

### Q119. What is SQL Server Locking?

**Answer:**

Locking protects data consistency during concurrent operations.

Common lock modes:
- Shared (S)
- Exclusive (X)
- Update (U)
- Intent locks

### Q120. What is NOLOCK?

**Answer:**

NOLOCK is equivalent to READ UNCOMMITTED for a table reference.

It allows reading uncommitted data and can return dirty or inconsistent results.

It should not be used as a general performance fix.

---

# 10. Entity Framework Core & Dapper

### Q121. What is Entity Framework Core?

**Answer:**

EF Core is an Object-Relational Mapper (ORM) for .NET.

It allows developers to interact with databases using C# objects.

### Q122. What is DbContext?

**Answer:**

DbContext represents a session with the database.

It manages:
- Entity tracking
- Database queries
- Changes
- Saving changes
- Transactions

### Q123. What is DbSet?

**Answer:**

DbSet represents a collection of entities of a specific type.

```csharp
public DbSet<Employee> Employees { get; set; }
```

### Q124. What is Code First?

**Answer:**

Code First starts with C# entity classes and uses them to create or update the database schema through migrations.

### Q125. What is Database First?

**Answer:**

Database First starts with an existing database and generates entity classes and DbContext from it.

### Q126. What is AsNoTracking()?

**Answer:**

AsNoTracking disables change tracking for query results.

It is useful for read-only queries.

```csharp
var employees = await context.Employees
    .AsNoTracking()
    .ToListAsync();
```

### Q127. What is Lazy Loading vs Eager Loading?

**Answer:**

- Lazy Loading: Related data is loaded when accessed.
- Eager Loading: Related data is loaded as part of the original query using Include or related mechanisms.

### Q128. What is Dapper?

**Answer:**

Dapper is a lightweight micro-ORM for .NET.

It maps SQL query results to C# objects.

### Q129. What is the difference between Dapper and EF Core?

**Answer:**

| Dapper | EF Core |
|---|---|
| Micro-ORM | Full ORM |
| SQL control is explicit | LINQ-based querying |
| Lightweight | Provides change tracking |
| Minimal abstraction | More database abstraction |
| Requires more manual mapping | Supports entity relationships |

### Q130. Why use Dapper with Stored Procedures?

**Answer:**

Dapper provides:
- Lightweight data access
- Fast object mapping
- Stored Procedure support
- Parameterized queries
- Multiple result-set support

Example:

```csharp
using Dapper;
using System.Data;

var employees = await connection.QueryAsync<Employee>(
    "sp_GetEmployees",
    commandType: CommandType.StoredProcedure
);
```

---

# 11. Angular 20

### Q131. What is Angular?

**Answer:**

Angular is a TypeScript-based frontend framework developed by Google.

It is used to build scalable single-page applications.

### Q132. What is Angular Standalone Component?

**Answer:**

A Standalone Component can be used without declaring it in an NgModule.

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-home',
  standalone: true,
  template: `<h1>Hello Angular</h1>`
})
export class HomeComponent {}
```

Standalone is the recommended approach for modern Angular applications.

### Q133. What is Component?

**Answer:**

A Component controls a part of the UI.

It contains:
- TypeScript logic
- HTML template
- CSS styles
- Component metadata

### Q134. What is Angular Module?

**Answer:**

An NgModule groups components, directives, pipes and providers.

Modern Angular supports standalone APIs, so NgModules are not required for every application.

### Q135. What are Data Binding Types?

**Answer:**

Angular provides four common binding styles.

1. Interpolation
2. Property Binding
3. Event Binding
4. Two-way Binding

Example:

```html
<h1>{{ title }}</h1>

<input [value]="name">

<button (click)="save()">Save</button>

<input [(ngModel)]="name">
```

### Q136. What is @Input and @Output?

**Answer:**

They are used for communication between parent and child components.

- Input: Parent sends data to child.
- Output: Child emits events to parent.

Modern Angular also supports `input()` and `output()` APIs.

### Q137. What are Angular Lifecycle Hooks?

**Answer:**

Important lifecycle hooks:

- ngOnChanges()
- ngOnInit()
- ngDoCheck()
- ngAfterContentInit()
- ngAfterViewInit()
- ngOnDestroy()

### Q138. What is Angular Service?

**Answer:**

A Service contains reusable business logic or data-access functionality.

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class EmployeeService {
  getEmployees() {
    return [];
  }
}
```

### Q139. What is Dependency Injection in Angular?

**Answer:**

Angular DI provides services and other dependencies to components and classes.

```typescript
private employeeService = inject(EmployeeService);
```

### Q140. What is Angular Routing?

**Answer:**

Angular Router enables navigation between views without full-page reloads.

```typescript
export const routes: Routes = [
  {
    path: 'employees',
    loadComponent: () =>
      import('./employees/employees.component')
        .then(m => m.EmployeesComponent)
  }
];
```

### Q141. What is Lazy Loading?

**Answer:**

Lazy Loading loads application features only when required.

Benefits:
- Smaller initial bundle
- Faster initial loading
- Better application scalability

### Q142. What is Route Guard?

**Answer:**

Route Guards control navigation based on application conditions.

Examples:
- CanActivate
- CanActivateChild
- CanDeactivate
- CanMatch

Important: Client-side guards do not replace backend authorization.

### Q143. What is HTTP Interceptor?

**Answer:**

An HTTP Interceptor can modify outgoing HTTP requests and incoming HTTP responses.

Use cases:
- JWT token attachment
- Global error handling
- Loading indicators
- Request logging

### Q144. What is Reactive Forms?

**Answer:**

Reactive Forms manage form state and validation programmatically using FormGroup, FormControl and FormArray.

```typescript
form = new FormGroup({
  name: new FormControl('', Validators.required),
  email: new FormControl('', Validators.email)
});
```

### Q145. What is Template-driven Form vs Reactive Form?

**Answer:**

| Template-driven | Reactive |
|---|---|
| Template-based configuration | TypeScript-based configuration |
| Suitable for simple forms | Suitable for complex forms |
| Uses NgModel | Uses FormControl/FormGroup |
| Less explicit state management | Explicit state management |

### Q146. What are Signals in Angular?

**Answer:**

Signals are reactive primitives for managing application state.

```typescript
import { signal, computed } from '@angular/core';

count = signal(0);

doubleCount = computed(() => this.count() * 2);

increment() {
  this.count.update(value => value + 1);
}
```

### Q147. What is computed()?

**Answer:**

computed() creates a read-only derived signal based on other signals.

### Q148. What is effect()?

**Answer:**

effect() runs a side effect when the signals it reads change.

It is useful for synchronization and side effects, not as a replacement for computed state.

### Q149. What is @if, @for and @switch?

**Answer:**

Angular provides built-in control flow syntax.

```html
@if (isLoggedIn()) {
  <p>Welcome!</p>
} @else {
  <p>Please login</p>
}
```

```html
@for (employee of employees(); track employee.id) {
  <p>{{ employee.name }}</p>
}
```

```html
@switch (status()) {
  @case ('active') {
    <p>Active</p>
  }
  @case ('inactive') {
    <p>Inactive</p>
  }
}
```

### Q150. What is track in @for?

**Answer:**

The track expression identifies each item uniquely so Angular can efficiently update the DOM.

### Q151. What is Change Detection?

**Answer:**

Change Detection determines when Angular needs to update the UI based on application state changes.

Modern Angular supports different strategies and zoneless change detection.

### Q152. What is OnPush?

**Answer:**

OnPush is a change detection strategy that reduces unnecessary checking.

It is especially useful in large component trees.

### Q153. What is Angular CLI?

**Answer:**

Angular CLI is a command-line tool for creating, building, testing and serving Angular applications.

Common commands:

```bash
ng new my-app
ng serve
ng build
ng generate component employees
ng generate service employee
ng test
```

### Q154. How do you connect Angular with ASP.NET Core Web API?

**Answer:**

Step 1: Create API endpoint in ASP.NET Core.

```csharp
[HttpGet]
public IActionResult GetEmployees()
{
    return Ok(new[]
    {
        new { Id = 1, Name = "Abdullah" }
    });
}
```

Step 2: Create Angular Service.

```typescript
@Injectable({
  providedIn: 'root'
})
export class EmployeeService {

  private http = inject(HttpClient);

  getEmployees() {
    return this.http.get<Employee[]>('/api/employees');
  }
}
```

Step 3: Register HttpClient.

```typescript
bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient()
  ]
});
```

Step 4: Subscribe to the API response or use it with Angular's async/reactive patterns.

---

# 12. TypeScript & JavaScript

### Q155. What is TypeScript?

**Answer:**

TypeScript is a strongly typed superset of JavaScript.

It adds:
- Static type checking
- Interfaces
- Type aliases
- Generics
- Enums
- Advanced tooling

### Q156. What is Interface vs Type?

**Answer:**

Both define object shapes.

| Interface | Type |
|---|---|
| Supports declaration merging | Does not support declaration merging |
| Can extend interfaces | Can use intersections |
| Mainly used for object contracts | Supports unions, intersections and more |

### Q157. What is any vs unknown?

**Answer:**

- any disables most type checking.
- unknown requires type checking before using the value.

```typescript
let value: unknown = "Hello";

if (typeof value === 'string') {
  console.log(value.toUpperCase());
}
```

### Q158. What is Promise?

**Answer:**

A Promise represents the eventual completion or failure of an asynchronous operation.

States:
- Pending
- Fulfilled
- Rejected

### Q159. What is async/await in JavaScript?

**Answer:**

It provides cleaner syntax for working with Promises.

```typescript
async function getEmployees() {
  try {
    const response = await fetch('/api/employees');
    const data = await response.json();

    console.log(data);
  } catch (error) {
    console.error(error);
  }
}
```

### Q160. What is == vs ===?

**Answer:**

- == compares values after possible type coercion.
- === compares values without type coercion.

Prefer === for predictable comparisons.

### Q161. What is let, const and var?

**Answer:**

| var | let | const |
|---|---|---|
| Function scoped | Block scoped | Block scoped |
| Can be redeclared | Cannot redeclare in same scope | Cannot redeclare in same scope |
| Can be reassigned | Can be reassigned | Cannot be reassigned |

### Q162. What is Closure?

**Answer:**

A Closure allows a function to access variables from its outer lexical scope even after the outer function has completed.

### Q163. What is Hoisting?

**Answer:**

Hoisting describes how JavaScript declarations are processed before code execution.

`var` is initialized with undefined, while `let` and `const` remain in the temporal dead zone until declaration evaluation.

---

# 13. RxJS & Observables

### Q164. What is RxJS?

**Answer:**

RxJS is a library for reactive programming using Observables.

Angular uses RxJS extensively for HTTP requests, events and asynchronous data streams.

### Q165. What is Observable?

**Answer:**

An Observable represents a stream of asynchronous values over time.

```typescript
import { Observable } from 'rxjs';

const observable = new Observable(observer => {
  observer.next(10);
  observer.next(20);
  observer.complete();
});
```

### Q166. What is Observable vs Promise?

**Answer:**

| Observable | Promise |
|---|---|
| Can emit multiple values | Resolves once |
| Can be lazy | Starts execution when created |
| Can be cancelled through unsubscribe | Native Promise has no general cancellation mechanism |
| Supports RxJS operators | Uses then/catch/finally |

### Q167. What is subscribe()?

**Answer:**

subscribe() starts an Observable execution and receives its notifications.

```typescript
this.employeeService.getEmployees()
  .subscribe({
    next: data => console.log(data),
    error: error => console.error(error),
    complete: () => console.log('Completed')
  });
```

### Q168. What is pipe()?

**Answer:**

pipe() combines RxJS operators to transform or control Observable streams.

### Q169. What is map()?

**Answer:**

map() transforms each emitted value.

### Q170. What is switchMap()?

**Answer:**

switchMap() switches to a new inner Observable and unsubscribes from the previous inner subscription.

Useful for search requests where newer input should replace older requests.

### Q171. What is mergeMap()?

**Answer:**

mergeMap() runs inner Observables concurrently.

### Q172. What is concatMap()?

**Answer:**

concatMap() processes inner Observables sequentially.

### Q173. What is exhaustMap()?

**Answer:**

exhaustMap() ignores new source emissions while the current inner Observable is running.

Useful for preventing repeated login or submit requests.

### Q174. What is debounceTime()?

**Answer:**

debounceTime() waits for a period of inactivity before emitting the latest value.

Useful for search boxes.

### Q175. What is distinctUntilChanged()?

**Answer:**

It suppresses consecutive duplicate values.

### Q176. What is takeUntilDestroyed()?

**Answer:**

Angular provides takeUntilDestroyed() to automatically complete an Observable subscription when its associated Angular context is destroyed.

---

# 14. Authentication & Authorization

### Q177. What is Authentication?

**Answer:**

Authentication verifies who the user is.

Example: Login using email and password.

### Q178. What is Authorization?

**Answer:**

Authorization determines what an authenticated user is allowed to access.

Example: Only Admin can delete employees.

### Q179. What is JWT?

**Answer:**

JWT stands for JSON Web Token.

It is a compact token format commonly used for authentication and authorization.

A JWT has three parts:

1. Header
2. Payload
3. Signature

### Q180. What is Access Token?

**Answer:**

An Access Token is a credential used to access protected API endpoints.

It should have a short lifetime.

### Q181. What is Refresh Token?

**Answer:**

A Refresh Token is used to obtain a new access token without requiring the user to log in again.

A secure implementation should:
- Store refresh tokens securely.
- Rotate refresh tokens.
- Revoke old tokens.
- Detect token reuse where appropriate.
- Apply expiration.

### Q182. What is Role-Based Authorization?

**Answer:**

Access is controlled according to user roles.

```csharp
[Authorize(Roles = "Admin")]
[HttpDelete("{id}")]
public IActionResult Delete(int id)
{
    return NoContent();
}
```

### Q183. What is Claims-Based Authorization?

**Answer:**

Access is controlled based on claims associated with the authenticated user.

```csharp
[Authorize(Policy = "CanManageEmployees")]
```

### Q184. What is Policy-Based Authorization?

**Answer:**

Policy-Based Authorization defines authorization requirements centrally.

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));
});
```

### Q185. Where should JWT be stored in Angular?

**Answer:**

There is no universally safe browser storage location.

Important considerations:

- Avoid storing sensitive long-lived tokens in localStorage where possible because JavaScript can access it.
- HttpOnly, Secure, SameSite cookies can reduce token theft through JavaScript.
- Cookie-based authentication requires CSRF protection.
- Access tokens should have limited lifetimes.
- Apply strong XSS protection.

### Q186. What is Password Hashing?

**Answer:**

Password hashing stores a one-way derived representation of a password instead of the original password.

Use established password hashing algorithms such as:
- Argon2id
- PBKDF2
- bcrypt

Never store plain-text passwords.

---

# 15. SOLID Principles & Design Patterns

### Q187. What is SOLID?

**Answer:**

SOLID is a set of five object-oriented design principles.

| Principle | Meaning |
|---|---|
| S | Single Responsibility Principle |
| O | Open/Closed Principle |
| L | Liskov Substitution Principle |
| I | Interface Segregation Principle |
| D | Dependency Inversion Principle |

### Q188. What is SRP?

**Answer:**

Single Responsibility Principle means a class should have one reason to change.

Example:

Separate:
- EmployeeService
- EmployeeRepository
- EmailService

### Q189. What is OCP?

**Answer:**

Open/Closed Principle means software entities should be open for extension but closed for modification.

### Q190. What is LSP?

**Answer:**

Liskov Substitution Principle means derived classes should be usable wherever their base class is expected without breaking correctness.

### Q191. What is ISP?

**Answer:**

Interface Segregation Principle means clients should not depend on methods they do not use.

Prefer smaller, focused interfaces.

### Q192. What is DIP?

**Answer:**

Dependency Inversion Principle means high-level modules should depend on abstractions rather than concrete implementations.

### Q193. What is Repository Pattern?

**Answer:**

Repository Pattern separates data-access logic from business logic.

```csharp
public interface IEmployeeRepository
{
    Task<Employee> GetByIdAsync(int id);
}
```

### Q194. What is Unit of Work?

**Answer:**

Unit of Work coordinates multiple data operations as one logical transaction.

It helps ensure related changes are committed or rolled back together.

### Q195. What is Factory Pattern?

**Answer:**

Factory Pattern centralizes object creation and hides construction details.

### Q196. What is Singleton Pattern?

**Answer:**

Singleton ensures a single instance of a class is shared within its intended scope.

Be careful with shared mutable state and thread safety.

### Q197. What is Strategy Pattern?

**Answer:**

Strategy Pattern allows interchangeable algorithms to be selected at runtime.

Example:
- SSLCommerz Payment
- bKash Payment
- Nagad Payment

Each payment provider can implement a common interface.

### Q198. What is CQRS?

**Answer:**

CQRS stands for Command Query Responsibility Segregation.

It separates:
- Commands: Operations that change state.
- Queries: Operations that read data.

It can improve separation of concerns but adds complexity.

---

# 16. Clean Architecture

### Q199. What is Clean Architecture?

**Answer:**

Clean Architecture separates application responsibilities into independent layers.

Typical layers:

1. Domain
2. Application
3. Infrastructure
4. API / Presentation

### Q200. What is Domain Layer?

**Answer:**

The Domain Layer contains:
- Entities
- Value Objects
- Domain Services
- Business Rules
- Domain Events

It should not depend on Infrastructure or API.

### Q201. What is Application Layer?

**Answer:**

The Application Layer contains:
- Use Cases
- Commands
- Queries
- DTOs
- Interfaces
- Validation
- Application Services

### Q202. What is Infrastructure Layer?

**Answer:**

Infrastructure implements external technical concerns.

Examples:
- SQL Server
- ADO.NET
- Dapper
- Email Service
- File Storage
- External Payment Gateway

### Q203. What is API Layer?

**Answer:**

The API Layer exposes application functionality through HTTP endpoints.

It handles:
- Controllers
- Middleware
- Authentication
- Authorization
- Request/Response handling

### Q204. What is Dependency Rule?

**Answer:**

Dependencies should point inward toward the core business logic.

Domain should not depend on Infrastructure.

Infrastructure implements interfaces defined by inner layers.

### Q205. Why use Clean Architecture?

**Answer:**

Benefits:
- Maintainability
- Testability
- Separation of Concerns
- Loose Coupling
- Easier replacement of infrastructure technologies

---

# 17. REST API & HTTP

### Q206. What is Idempotency?

**Answer:**

An operation is idempotent when repeating it produces the same intended effect as performing it once.

Examples:
- GET
- PUT
- DELETE

POST is generally not idempotent by default.

### Q207. What is HTTP vs HTTPS?

**Answer:**

HTTP transfers data without transport encryption.

HTTPS uses TLS to protect communication.

### Q208. What is Request Header?

**Answer:**

Headers contain metadata about an HTTP request.

Examples:
- Authorization
- Content-Type
- Accept
- User-Agent

### Q209. What is Content-Type?

**Answer:**

Content-Type identifies the media type of the request or response body.

Example:

```http
Content-Type: application/json
```

### Q210. What is Pagination?

**Answer:**

Pagination divides a large result set into smaller pages.

Example:

```http
GET /api/employees?page=1&pageSize=10
```

Benefits:
- Lower response size
- Better performance
- Improved UI experience

### Q211. What is Rate Limiting?

**Answer:**

Rate Limiting restricts how many requests a client can make within a defined time period.

It helps protect APIs from abuse and excessive traffic.

### Q212. What is API Gateway?

**Answer:**

API Gateway provides a single entry point to multiple backend services.

Common responsibilities:
- Routing
- Authentication
- Rate Limiting
- Logging
- Load Balancing

---

# 18. Performance & Security

### Q213. What is Caching?

**Answer:**

Caching stores frequently accessed data temporarily to reduce repeated processing.

Types:
- In-Memory Cache
- Distributed Cache
- Response Caching
- Output Caching

### Q214. What is Redis?

**Answer:**

Redis is an in-memory data store often used for:
- Distributed Caching
- Session Storage
- Rate Limiting
- Queue-like workloads

### Q215. What is XSS?

**Answer:**

Cross-Site Scripting is an attack where malicious scripts execute in a user's browser.

Prevention:
- Context-aware output encoding
- Avoid unsafe DOM manipulation
- Use Content Security Policy
- Sanitize untrusted HTML when HTML rendering is necessary

### Q216. What is CSRF?

**Answer:**

Cross-Site Request Forgery tricks an authenticated browser into sending unwanted requests.

Prevention:
- Anti-forgery tokens
- SameSite cookies
- Origin/Referer validation where appropriate

### Q217. What is Rate Limiting vs Throttling?

**Answer:**

Rate Limiting enforces a request quota.

Throttling controls or slows request processing when limits or resource constraints are reached.

### Q218. How do you improve ASP.NET Core API performance?

**Answer:**

1. Use asynchronous I/O.
2. Optimize database queries.
3. Add appropriate indexes.
4. Use pagination.
5. Use caching.
6. Avoid unnecessary object allocations.
7. Reduce payload size.
8. Use compression where beneficial.
9. Monitor application performance.
10. Avoid N+1 database queries.

### Q219. What is Logging?

**Answer:**

Logging records application events for troubleshooting and monitoring.

ASP.NET Core provides ILogger.

```csharp
private readonly ILogger<EmployeeService> _logger;

public EmployeeService(ILogger<EmployeeService> logger)
{
    _logger = logger;
}

_logger.LogInformation("Employee created successfully");
```

### Q220. What is Health Check?

**Answer:**

Health Checks expose information about application and dependency health.

Examples:
- API availability
- SQL Server connectivity
- Redis connectivity
- External service status

---

# 19. Git, Docker & Deployment

### Q221. What is Git?

**Answer:**

Git is a distributed version control system used to track code changes.

### Q222. Important Git Commands

```bash
git init
git clone <repository-url>
git status
git add .
git commit -m "Initial commit"
git push
git pull
git branch
git checkout
git switch
git merge
git log
```

### Q223. What is Git Branch?

**Answer:**

A branch allows developers to work on features or fixes independently.

### Q224. What is Merge Conflict?

**Answer:**

A merge conflict happens when Git cannot automatically reconcile changes.

Developers must resolve conflicting changes manually.

### Q225. What is Docker?

**Answer:**

Docker is a containerization platform.

It packages an application and its dependencies into a container image.

### Q226. What is Docker Image vs Container?

**Answer:**

| Image | Container |
|---|---|
| Application template | Running or stopped instance of an image |
| Immutable layers | Has a writable container layer |
| Used to create containers | Executes the application |

### Q227. What is Docker Compose?

**Answer:**

Docker Compose defines and runs multi-container applications.

Example services:
- ASP.NET Core API
- SQL Server
- Redis
- Angular frontend

### Q228. What is CI/CD?

**Answer:**

CI/CD stands for Continuous Integration and Continuous Delivery/Deployment.

Typical pipeline:

1. Developer pushes code.
2. Build starts.
3. Tests execute.
4. Application is packaged.
5. Deployment happens.
6. Application is monitored.

---

# 20. Scenario-Based Interview Questions

These questions are particularly important for experienced Full Stack .NET Developers.

### Q229. Your API is returning 500 Internal Server Error. How do you debug it?

**Answer:**

1. Check API logs.
2. Check exception details.
3. Verify database connection.
4. Check SQL Stored Procedure.
5. Verify request payload.
6. Check dependency injection registration.
7. Debug the controller and service.
8. Check external dependencies.

Use centralized exception-handling middleware.

### Q230. Your Angular application receives 401 Unauthorized. What do you check?

**Answer:**

1. Is the access token available?
2. Is the token expired?
3. Is the Authorization header present?
4. Is the token signed with the correct key?
5. Are Issuer and Audience correct?
6. Is the authentication middleware registered?
7. Is UseAuthentication() before UseAuthorization()?
8. Does the API endpoint require the correct role or policy?

### Q231. Your Angular application receives 403 Forbidden. What do you check?

**Answer:**

- User authentication status.
- User role.
- Required permissions.
- Authorization policy.
- Claims inside the access token.
- Resource-level authorization.

### Q232. Your SQL Stored Procedure is slow. How do you solve it?

**Answer:**

1. Execute the procedure in SSMS.
2. Enable Actual Execution Plan.
3. Check logical reads and execution time.
4. Check indexes.
5. Analyze joins.
6. Avoid non-SARGable conditions.
7. Check parameter-sensitive plans.
8. Check blocking and deadlocks.
9. Re-test with realistic data.

### Q233. How do you prevent duplicate form submission in Angular?

**Answer:**

Disable the submit button while a request is running.

Also implement server-side idempotency for operations where duplicate execution would cause financial or business problems.

### Q234. How do you handle multiple database operations?

**Answer:**

Use a database transaction.

For ADO.NET, use SqlTransaction.

For business-critical workflows, ensure all required operations commit together or roll back together.

### Q235. How do you implement Role-Based Access Control?

**Answer:**

1. Create Users table.
2. Create Roles table.
3. Create UserRoles table.
4. Authenticate the user.
5. Generate JWT with required claims.
6. Apply authorization policies in ASP.NET Core.
7. Protect Angular routes using guards.
8. Always validate permissions on the backend.

### Q236. How do you handle large data in an API?

**Answer:**

- Server-side pagination.
- Filtering.
- Sorting.
- Appropriate SQL indexes.
- Asynchronous database access.
- Streaming or export-specific processing for large datasets.
- Caching when appropriate.

### Q237. How do you implement Multi-Tenancy?

**Answer:**

Multi-Tenancy allows multiple organizations to use the same application while keeping their data isolated.

Possible approaches:
- Shared Database, Shared Schema
- Shared Database, Separate Schema
- Separate Database per Tenant

For shared-schema systems:
- Include TenantId in relevant tables.
- Apply tenant filters to every data-access operation.
- Validate tenant access on the backend.
- Add tenant-scoped authorization tests.
- Consider composite indexes involving TenantId.

Never trust a TenantId supplied by the frontend without validating the authenticated user's access.

### Q238. How do you design a Payment System?

**Answer:**

1. Create Payment entity.
2. Create Payment Gateway abstraction.
3. Implement gateway-specific services.
4. Create a payment initiation endpoint.
5. Verify gateway callback/webhook signatures.
6. Verify transaction status with the gateway.
7. Update payment records transactionally.
8. Use idempotency to prevent duplicate processing.
9. Store audit logs.
10. Handle failed and pending payments.

### Q239. How do you handle concurrent seat booking?

**Answer:**

Use database-level concurrency control.

Possible implementation:

1. Create a seat-locking mechanism.
2. Use SQL transactions.
3. Apply appropriate locking or optimistic concurrency.
4. Set lock expiration.
5. Confirm booking only when the seat is successfully reserved.
6. Release expired locks.
7. Enforce database constraints to prevent duplicate active bookings.

### Q240. How do you secure a .NET Web API?

**Answer:**

- HTTPS.
- JWT authentication or secure cookie authentication.
- Role/Policy-based authorization.
- Password hashing.
- Input validation.
- SQL parameterization.
- Rate limiting.
- CORS configuration.
- Secure configuration and secrets.
- Logging and auditing.
- Dependency updates.
- Proper error handling.

---

# ⭐ Quick Revision: Most Important Interview Topics

Before attending an interview, revise these topics carefully.

| Priority | Topic | Important Concepts |
|---|---|---|
| High | C# | OOP, async/await, LINQ, Collections |
| High | ASP.NET Core | Middleware, DI, Routing |
| High | Web API | HTTP methods, Status codes, DTO |
| High | ADO.NET | SqlConnection, SqlCommand, Transactions |
| High | SQL Server | Joins, Indexes, Stored Procedures |
| High | Angular | Components, Services, Routing |
| High | Security | JWT, Authentication, Authorization |
| High | Architecture | SOLID, Repository, Clean Architecture |
| Medium | Advanced C# | Delegates, Events, Generics |
| Medium | Angular | Signals, RxJS, Interceptors |
| Medium | Database | Isolation, Deadlocks, Execution Plans |
| Medium | DevOps | Git, Docker, CI/CD |

---

# 💻 Important Coding Practice

Practice these problems before technical interviews.

### C# Problems

1. Reverse a string.
2. Check palindrome.
3. Find duplicate elements in an array.
4. Find the second-largest number.
5. Find factorial using recursion.
6. Check prime number.
7. Count character frequency.
8. Remove duplicates from a list.
9. Sort an array without using built-in sorting.
10. Find missing numbers in an array.

### SQL Problems

1. Find the second-highest salary.
2. Find duplicate records.
3. Delete duplicate records.
4. Find employees without departments.
5. Find the top 3 salaries in each department.
6. Find the nth-highest salary.
7. Calculate department-wise employee count.
8. Find employees hired in the last 30 days.
9. Implement pagination.
10. Find customers with no orders.

### Angular Problems

1. Build a CRUD application.
2. Implement Reactive Forms.
3. Create HTTP Interceptors.
4. Implement Route Guards.
5. Implement Lazy Loading.
6. Build a Searchable DataTable.
7. Implement Pagination.
8. Use Angular Signals.
9. Implement RxJS debounceTime and switchMap.
10. Implement JWT authentication.

---

# 🎯 Interview Preparation Strategy

### Junior .NET Developer

Focus on:
- C# Fundamentals
- OOP
- SQL Queries
- ASP.NET Core Basics
- Web API
- Angular Fundamentals
- CRUD Operations

### Mid-Level .NET Developer

Focus on:
- Advanced C#
- ADO.NET
- Stored Procedures
- SQL Optimization
- JWT Authentication
- Angular Signals and RxJS
- Dependency Injection
- SOLID Principles
- Design Patterns

### Senior .NET Developer

Focus on:
- Clean Architecture
- CQRS
- Microservices
- Distributed Systems
- Database Concurrency
- Performance Optimization
- Security Architecture
- Cloud and DevOps
- System Design

---

# 📖 Recommended Official Documentation

| Technology | Documentation |
|---|---|
| C# | https://learn.microsoft.com/en-us/dotnet/csharp/ |
| .NET | https://learn.microsoft.com/en-us/dotnet/ |
| ASP.NET Core | https://learn.microsoft.com/en-us/aspnet/core/ |
| ADO.NET | https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ |
| SQL Server | https://learn.microsoft.com/en-us/sql/sql-server/ |
| Angular | https://angular.dev/ |
| TypeScript | https://www.typescriptlang.org/docs/ |
| RxJS | https://rxjs.dev/ |
| Docker | https://docs.docker.com/ |
| Git | https://git-scm.com/doc |

---

## 🚀 Final Advice

Interview preparation should not be limited to memorizing answers.

For every topic:

1. Understand the concept.
2. Write code yourself.
3. Build a real-world example.
4. Understand why and when to use it.
5. Know the advantages and disadvantages.
6. Practice explaining the concept in your own words.

**Remember: A good .NET Developer does not just know how to write code. A good developer understands how to build secure, scalable, maintainable and performant applications.**

**Happy Coding! 💙**
