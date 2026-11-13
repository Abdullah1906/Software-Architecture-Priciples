# Idempotency & Concurrency in Software Development

## Complete Guide: Basic to Professional Level

**Technology:** ASP.NET Core Web API | C# | SQL Server | ADO.NET | Stored Procedures

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is Idempotency?](#2-what-is-idempotency)
3. [Why Do We Need Idempotency?](#3-why-do-we-need-idempotency)
4. [How Idempotency Works](#4-how-idempotency-works)
5. [HTTP Idempotent Methods](#5-http-idempotent-methods)
6. [Implementing Idempotency in ASP.NET Core](#6-implementing-idempotency-in-aspnet-core)
7. [What is Concurrency?](#7-what-is-concurrency)
8. [Why Do We Need Concurrency Control?](#8-why-do-we-need-concurrency-control)
9. [Understanding Race Conditions](#9-understanding-race-conditions)
10. [Types of Concurrency Control](#10-types-of-concurrency-control)
11. [Implementing Concurrency in SQL Server](#11-implementing-concurrency-in-sql-server)
12. [Implementing Concurrency in ASP.NET Core](#12-implementing-concurrency-in-aspnet-core)
13. [Idempotency vs Concurrency](#13-idempotency-vs-concurrency)
14. [Real-World Examples](#14-real-world-examples)
15. [Professional Best Practices](#15-professional-best-practices)
16. [Common Mistakes](#16-common-mistakes)
17. [Interview Questions](#17-interview-questions)
18. [Final Checklist](#18-final-checklist)

---

# 1. Introduction

ধরো, তুমি একটি Bus Ticket Booking System তৈরি করেছ।

একজন Customer একটি Seat বুক করার জন্য Request পাঠাল।

কিন্তু সমস্যা হলো:

- Customer-এর Internet Connection Slow।
- API Request পাঠানো হয়েছে, কিন্তু Response আসতে দেরি হচ্ছে।
- Customer মনে করল Booking হয়নি।
- সে আবার একই Button-এ Click করল।
- একই Seat-এর জন্য একাধিক Booking Request চলে গেল।

এখন কী হতে পারে?

- একই Customer-এর একাধিক Booking তৈরি হতে পারে।
- একই Seat একাধিক Customer বুক করতে পারে।
- Payment একাধিকবার Charge হতে পারে।
- Database-এর Data ভুল হতে পারে।

এই সমস্যাগুলো প্রতিরোধ করার জন্য Software Engineering-এ দুটি গুরুত্বপূর্ণ Concept ব্যবহার করা হয়:

**1. Idempotency:** একই Request একাধিকবার পাঠালেও একই Operation-এর Duplicate Execution প্রতিরোধ করা।

**2. Concurrency Control:** একাধিক User একই সময়ে একই Data পরিবর্তন করতে চাইলে Data Consistency বজায় রাখা।

দুটির উদ্দেশ্য আলাদা, কিন্তু Production-Level Software-এ দুটিই গুরুত্বপূর্ণ।

---

# 2. What is Idempotency?

### Definition

Idempotency হলো এমন একটি Property, যেখানে একই Operation একই Input দিয়ে একাধিকবার Execute করলেও তার Final Result একই থাকে।

সহজ ভাষায়:

একটি Request ১ বার পাঠাও অথবা ১০ বার পাঠাও, একই Operation-এর জন্য Duplicate Result তৈরি হবে না।

### Real-Life Example

ধরো, তুমি একটি Online Shopping Application থেকে ৳1,000-এর Product কিনছ।

প্রথমবার Payment Request পাঠানো হলো।

```http
POST /api/payments
```

Server Payment সফল করল।

কিন্তু Network Problem-এর কারণে Client Response পেল না।

এখন Client আবার একই Request পাঠাল।

Idempotency না থাকলে:

```text
Request 1 → Payment: ৳1,000
Request 2 → Payment: ৳1,000

Total Payment = ৳2,000
```

<details>
<summary>Idempotency ছাড়া সমস্যা</summary>

```text
Customer
   |
   | POST /api/payments
   |
   v
Payment API
   |
   v
Payment Gateway
   |
   +---- Charge ৳1,000
   |
   | Response Lost
   |
   v
Customer retries request
   |
   v
Payment API
   |
   v
Payment Gateway
   |
   +---- Charge ৳1,000 again

Result: ৳2,000 charged!
```

</details>

Idempotency ব্যবহার করলে:

```text
Request 1 → Payment: ৳1,000
Request 2 → Existing Result Returned

Total Payment = ৳1,000
```

<details>
<summary>Idempotency ব্যবহার করলে</summary>

```text
Customer
   |
   | POST /api/payments
   | Idempotency-Key: abc-123
   |
   v
Payment API
   |
   v
Check Idempotency Key
   |
   v
Key Not Found
   |
   v
Process Payment
   |
   v
Save Result with Key
   |
   v
Return Success

Customer retries request
   |
   v
Payment API
   |
   v
Check Idempotency Key
   |
   v
Key Already Exists
   |
   v
Return Previous Result

No Duplicate Payment!
```

</details>

### Important Concept

Idempotency মানে প্রতিটি HTTP Request-এর Response সবসময় একই হতে হবে, এমন নয়।

বরং একই Idempotency Key দিয়ে একই Operation পুনরায় পাঠালে আগের Operation পুনরায় Execute না করে সংরক্ষিত Result ব্যবহার করতে হবে।

---

# 3. Why Do We Need Idempotency?

Idempotency তৈরি করা হয়েছে Distributed Systems এবং Network Communication-এর একটি গুরুত্বপূর্ণ সমস্যা সমাধানের জন্য।

Network-এ Request এবং Response সবসময় নির্ভরযোগ্যভাবে আদান-প্রদান হয় না।

### Common Problems

| Problem | Description |
|---|---|
| Network Timeout | Server কাজ শেষ করলেও Client Response পায় না |
| Duplicate Request | Client একই Request আবার পাঠায় |
| Double Payment | Customer-এর টাকা একাধিকবার কেটে যায় |
| Duplicate Booking | একই Booking একাধিকবার তৈরি হয় |
| Retry Mechanism | Automatic Retry-এর কারণে একই Operation আবার চলে |
| Message Redelivery | Message Queue একই Message আবার Deliver করে |

### Where is Idempotency Used?

- Online Payment Systems
- Banking Applications
- E-commerce Applications
- Bus Ticket Booking Systems
- Subscription Management
- Order Management Systems
- Distributed Systems
- Microservices
- Message Queue Consumers

---

# 4. How Idempotency Works

Idempotency বাস্তবায়নের জন্য সাধারণত একটি Unique Idempotency Key ব্যবহার করা হয়।

### Step 1: Client Generates a Key

```csharp
var idempotencyKey = Guid.NewGuid().ToString();
```

### Step 2: Send Request

```http
POST /api/payments
Content-Type: application/json
Idempotency-Key: 7c8f2e9a-1234-4567-8901
```

Request Body:

```json
{
    "orderId": 101,
    "amount": 1000,
    "currency": "BDT"
}
```

### Step 3: Server Checks the Key

Server Database-এ Check করবে:

```text
Does this Idempotency Key exist?

       |
       |
   +---+---+
   |       |
  NO      YES
   |       |
   v       v
Process   Return
Request   Previous Result
```

### Step 4: Save the Result

প্রথমবার Request সফল হলে Database-এ নিচের Information Save হবে:

```text
IdempotencyKey
UserId
RequestHash
ResponseStatusCode
ResponseBody
CreatedAt
ExpiresAt
```

### Step 5: Return the Previous Result

একই Key দিয়ে Request আবার আসলে:

- Payment আবার Execute হবে না।
- Database-এ নতুন Payment তৈরি হবে না।
- Previous Result Return করা হবে।

**Important:** একই Key দিয়ে ভিন্ন Request Body পাঠানো যাবে না। Server-কে Request Hash যাচাই করতে হবে। Request আলাদা হলে `409 Conflict` Return করা যেতে পারে।

---

# 5. HTTP Idempotent Methods

HTTP Protocol-এ কিছু Method Naturally Idempotent।

| HTTP Method | Idempotent? | Explanation |
|---|---|---|
| GET | Yes | Data Read করে |
| PUT | Yes | Resource-কে নির্দিষ্ট State-এ নিয়ে যায় |
| DELETE | Yes | Resource Delete করে; পুনরাবৃত্তিতে একই Final State |
| POST | Usually No | নতুন Resource বা Operation তৈরি করতে পারে |
| PATCH | Depends | Operation-এর ওপর নির্ভর করে |
 
### Example: PUT

```http
PUT /api/users/10
```

```json
{
    "name": "Abdullah",
    "email": "abdullah@example.com"
}
```

একই Request ৫ বার পাঠালেও User-এর Final Data একই থাকবে।

### Example: POST

```http
POST /api/orders
```

একই Request ৫ বার পাঠালে সাধারণভাবে ৫টি Order তৈরি হতে পারে।

তাই Payment বা Order Creation-এর মতো গুরুত্বপূর্ণ POST Operation-এ Idempotency Key ব্যবহার করা প্রয়োজন হতে পারে।

**Note:** HTTP Method Idempotent হওয়া এবং Business Operation Idempotent হওয়া এক বিষয় নয়। PUT-এর মতো Method ব্যবহার করলেও Business Logic সঠিকভাবে তৈরি করতে হবে।

---

# 6. Implementing Idempotency in ASP.NET Core

এখন আমরা ASP.NET Core Web API এবং SQL Server ব্যবহার করে একটি Practical Design তৈরি করব।

## Step 1: Create Database Table

```sql
CREATE TABLE IdempotencyRequests
(
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,

    UserId BIGINT NOT NULL,

    Endpoint NVARCHAR(200) NOT NULL,

    IdempotencyKey NVARCHAR(100) NOT NULL,

    RequestHash NVARCHAR(64) NOT NULL,

    ResponseStatusCode INT NULL,

    ResponseBody NVARCHAR(MAX) NULL,

    Status NVARCHAR(20) NOT NULL,

    CreatedAt DATETIME2 NOT NULL
        DEFAULT SYSUTCDATETIME(),

    ExpiresAt DATETIME2 NOT NULL
);
```

### Create Unique Index

```sql
CREATE UNIQUE INDEX UX_IdempotencyRequests_Key
ON IdempotencyRequests
(
    UserId,
    Endpoint,
    IdempotencyKey
);
```

এই Unique Index নিশ্চিত করবে একই User এবং Endpoint-এর জন্য একই Key দিয়ে একাধিক Record তৈরি হবে না।

## Step 2: Generate Request Hash

Request Hash ব্যবহার করা হয় একই Idempotency Key দিয়ে একই Request পাঠানো হচ্ছে কি না তা যাচাই করার জন্য।

```csharp
using System.Security.Cryptography;
using System.Text;

public static class RequestHashHelper
{
    public static string Generate(string requestBody)
    {
        var bytes = Encoding.UTF8.GetBytes(requestBody);

        var hash = SHA256.HashData(bytes);

        return Convert.ToHexString(hash);
    }
}
```

একই Request Body থেকে একই Hash তৈরি হবে।

## Step 3: Create Idempotency Request Model

```csharp
public class IdempotencyRequest
{
    public long UserId { get; set; }

    public string Endpoint { get; set; } = string.Empty;

    public string IdempotencyKey { get; set; } = string.Empty;

    public string RequestHash { get; set; } = string.Empty;

    public int? ResponseStatusCode { get; set; }

    public string? ResponseBody { get; set; }

    public string Status { get; set; } = "Processing";
}
```

## Step 4: API Request Example

```csharp
[HttpPost]
public async Task<IActionResult> CreatePayment(
    [FromBody] CreatePaymentRequest request)
{
    var idempotencyKey =
        Request.Headers["Idempotency-Key"].FirstOrDefault();

    if (string.IsNullOrWhiteSpace(idempotencyKey))
    {
        return BadRequest(
            "Idempotency-Key header is required.");
    }

    // Validate the key format and length.
    // Authenticate the user.
    // Calculate the request hash.
    // Check the idempotency record.
    // Execute payment only if this is a new operation.

    return Ok(new
    {
        message = "Payment processed successfully"
    });
}
```

উপরের Controller শুধু Request গ্রহণের একটি Starting Point। Production-এ Idempotency-এর পুরো Logic Controller-এর মধ্যে না রেখে Service এবং Repository Layer-এ রাখা উচিত।

## Step 5: Professional Request Processing Flow

একটি Production Implementation-এর Flow:

```text
Client Request
      |
      v
Validate Idempotency Key
      |
      v
Authenticate User
      |
      v
Generate Request Hash
      |
      v
Check Existing Key
      |
      +--------------------+
      |                    |
   New Key              Existing Key
      |                    |
      v                    v
Start DB Transaction   Compare Request Hash
      |                    |
      v                    +---- Different
Insert Key              |
      |                  Return 409
      v
Execute Business Logic
      |
      v
Save Payment / Order
      |
      v
Save Response in Idempotency Table
      |
      v
Commit Transaction
      |
      v
Return Response
```

### Important Production Considerations

1. Idempotency Key-এর ওপর Database Unique Constraint থাকতে হবে।
2. Business Operation এবং Idempotency Result একই Database Transaction-এ Save করতে হবে, যখন উভয়ই একই Database-এ থাকে।
3. একই Key-এর Concurrent Request নিরাপদভাবে Handle করতে হবে।
4. Request Hash যাচাই করতে হবে।
5. একই Key-এর জন্য Processing অবস্থায় Request এলে সেটি কীভাবে Handle হবে তা Define করতে হবে।
6. Expired Key-এর Cleanup Policy থাকতে হবে।
7. External Payment Gateway ব্যবহার করলে Gateway-এর নিজস্ব Idempotency Key-ও ব্যবহার করতে হবে।

**Note:** Payment Gateway এবং নিজের Database দুটি আলাদা System। একটি Local SQL Transaction দিয়ে দুই System-এর Atomicity নিশ্চিত করা যায় না। External Payment-এর জন্য Durable Payment State, Gateway Idempotency এবং Reconciliation/Webhook Strategy প্রয়োজন।

---

# 7. What is Concurrency?

### Definition

Concurrency হলো এমন একটি পরিস্থিতি যেখানে একাধিক User, Thread অথবা Process একই সময়ে একই Resource বা Data নিয়ে কাজ করে।

সহজ ভাষায়:

একই সময়ে একাধিক User যদি একই Database Record Read, Update অথবা Delete করতে চায়, তখন Concurrency তৈরি হয়।

### Real-Life Example: Bus Seat Booking

ধরো, তোমার Bus Booking System-এ:

```text
Bus: Dhaka to Chittagong

Total Seats: 40

Available Seat: A1
```

এখন দুইজন Customer একই সময়ে A1 Seat বুক করতে চায়।

```text
Customer A              Customer B
    |                        |
    | Book Seat A1           | Book Seat A1
    |                        |
    +-----------+------------+
                |
                v
          Booking API
                |
                v
           Database
```

যদি Database-এ সঠিক Concurrency Control না থাকে:

```text
Customer A → Seat A1 → Booked

Customer B → Seat A1 → Booked
```

এখন একই Seat দুইজনের নামে Booked!

এটিকে বলা হয় **Double Booking**।

Concurrency Control ব্যবহার করে এই সমস্যা প্রতিরোধ করতে হয়।

---

# 8. Why Do We Need Concurrency Control?

একটি Software Application-এ একই সময়ে হাজার হাজার User কাজ করতে পারে।

যেমন:

- Banking Transaction
- Inventory Management
- Bus Ticket Booking
- E-commerce Order Processing
- Payroll Management
- Hospital Management
- Multi-user Admin Dashboard

Concurrency Control না থাকলে নিচের সমস্যাগুলো হতে পারে:

| Problem | Explanation |
|---|---|
| Race Condition | একাধিক Operation-এর Execution Order-এর কারণে ভুল Result |
| Lost Update | একজনের Update অন্যজনের Update Overwrite করে |
| Dirty Read | অন্য Transaction-এর Uncommitted Data Read করা |
| Non-repeatable Read | একই Transaction-এ একই Data বারবার Read করলে ভিন্ন Result |
| Phantom Read | একই Query আবার চালালে নতুন বা বাদ পড়া Row দেখা |
| Double Booking | একই Resource একাধিক User-এর নামে Allocate হওয়া |

---

# 9. Understanding Race Conditions

Race Condition হলো এমন একটি Problem যেখানে একাধিক Operation-এর Execution Timing বা Order-এর ওপর নির্ভর করে Final Result পরিবর্তিত হয়ে যায়।

### Example: Bank Account

ধরো:

```text
Account Balance = ৳10,000
```

একই সময়ে দুটি Transaction হচ্ছে:

- Transaction A: ৳3,000 Withdraw
- Transaction B: ৳2,000 Withdraw

সমস্যাটি বুঝি:

```text
Transaction A reads Balance = 10,000

Transaction B reads Balance = 10,000

Transaction A calculates:
10,000 - 3,000 = 7,000

Transaction B calculates:
10,000 - 2,000 = 8,000

Transaction A saves 7,000

Transaction B saves 8,000
```

Final Balance:

```text
৳8,000
```

কিন্তু সঠিক Balance হওয়ার কথা:

```text
10,000 - 3,000 - 2,000 = ৳5,000
```

এখানে Transaction A-এর Update হারিয়ে গেছে।

এটিকে বলা হয় **Lost Update Problem**।

### Solution

Database-এর Atomic Update ব্যবহার করতে হবে।

```sql
UPDATE Accounts
SET Balance = Balance - @Amount
WHERE AccountId = @AccountId
  AND Balance >= @Amount;
```

এখানে Database একবারে Update এবং Balance Condition Check করবে।

C# থেকে Rows Affected Check করতে হবে।

---

# 10. Types of Concurrency Control

Concurrency Control-এর প্রধান দুটি Strategy রয়েছে।

## 10.1 Optimistic Concurrency Control

Optimistic Concurrency ধরে নেয় যে একই Data একাধিক User একই সময়ে Update করবে না।

তাই সাধারণত Read করার সময় Lock না দিয়ে Update করার সময় Check করে Data পরিবর্তিত হয়েছে কি না।

এক্ষেত্রে SQL Server-এর `rowversion` ব্যবহার করা যায়।

### Example

ধরো, Employee Table:

```sql
CREATE TABLE Employees
(
    Id INT PRIMARY KEY,

    Name NVARCHAR(100),

    Salary DECIMAL(18,2),

    Version ROWVERSION
);
```

এখন Employee Data Read করলে:

```sql
SELECT Id, Name, Salary, Version
FROM Employees
WHERE Id = 1;
```

Database একটি Version Value Return করবে।

ধরো:

```text
Id = 1

Salary = 30000

Version = ABC123
```

User যখন Update করবে তখন আগের Version-টিও পাঠাবে।

```sql
UPDATE Employees
SET Salary = @Salary
WHERE Id = @Id
AND Version = @OriginalVersion;
```

এখন কী হবে?

- Data পরিবর্তন না হলে Update হবে।
- অন্য User আগে Update করলে Version পরিবর্তিত হবে।
- তখন আগের Version দিয়ে Update করলে কোনো Row Update হবে না।

C# Example:

```csharp
var affectedRows = await command.ExecuteNonQueryAsync();

if (affectedRows == 0)
{
    throw new Exception(
        "Data was modified by another user.");
}
```

Production API-তে এটি সাধারণত HTTP `409 Conflict` অথবা উপযুক্ত ক্ষেত্রে `412 Precondition Failed` হিসেবে Return করা যায়।

**Use Cases:**

- Employee Management
- Product Management
- Profile Update
- Admin Dashboard
- Document Editing

## 10.2 Pessimistic Concurrency Control

Pessimistic Concurrency ধরে নেয় যে একই Data একাধিক User একই সময়ে পরিবর্তন করতে পারে।

তাই Transaction চলাকালীন প্রয়োজন অনুযায়ী Lock ব্যবহার করা হয়।

SQL Server-এর `UPDLOCK` এবং `HOLDLOCK` ব্যবহার করা যায়।

### Example: Seat Booking

```sql
BEGIN TRANSACTION;

DECLARE @Available BIT;

SELECT @Available = IsAvailable
FROM BusSeats WITH (UPDLOCK, HOLDLOCK)
WHERE SeatId = @SeatId;

IF @Available = 1
BEGIN
    UPDATE BusSeats
    SET IsAvailable = 0
    WHERE SeatId = @SeatId;

    INSERT INTO Bookings
    (
        SeatId,
        UserId,
        BookingDate
    )
    VALUES
    (
        @SeatId,
        @UserId,
        SYSUTCDATETIME()
    );
END
ELSE
BEGIN
    ROLLBACK TRANSACTION;

    THROW 50001, 'Seat is already booked.', 1;
END;

COMMIT TRANSACTION;
```

এখানে `UPDLOCK` Update Lock নেয় এবং `HOLDLOCK` Transaction-এর শেষ পর্যন্ত প্রয়োজনীয় Lock ধরে রাখতে সাহায্য করে।

দ্বিতীয় Transaction একই Seat নিয়ে কাজ করতে গেলে প্রথম Transaction-এর Lock Release হওয়া পর্যন্ত অপেক্ষা করবে।

**Use Cases:**

- Bus Seat Booking
- Inventory Deduction
- Limited Stock Purchase
- Financial Transactions
- Resource Allocation

### Optimistic vs Pessimistic

| Feature | Optimistic | Pessimistic |
|---|---|---|
| Locking | সাধারণত Read-এর সময় Lock নেয় না | প্রয়োজন অনুযায়ী Lock নেয় |
| Conflict Detection | Update-এর সময় | Lock-এর মাধ্যমে |
| Performance | কম Conflict হলে ভালো | বেশি Contention হলে প্রয়োজন হতে পারে |
| User Experience | Conflict হলে Retry/Refresh | Waiting হতে পারে |
| Example | Employee Update | Seat Booking |

---

# 11. Implementing Concurrency in SQL Server

এখন দুটি গুরুত্বপূর্ণ Practical Example দেখি।

## Example 1: Inventory Management

ধরো:

```text
Product: Laptop

Available Stock: 5
```

দুইজন Customer একই সময়ে 4টি করে Laptop Order করতে চায়।

সঠিকভাবে Concurrency Control না থাকলে Stock Negative হতে পারে অথবা Overselling হতে পারে।

### Step 1: Create Table

```sql
CREATE TABLE Products
(
    Id INT PRIMARY KEY,

    ProductName NVARCHAR(100),

    Stock INT NOT NULL,

    CONSTRAINT CK_Products_Stock
        CHECK (Stock >= 0)
);
```

### Step 2: Insert Data

```sql
INSERT INTO Products
(
    Id,
    ProductName,
    Stock
)
VALUES
(
    1,
    'Laptop',
    5
);
```

### Step 3: Create Stored Procedure

```sql
CREATE PROCEDURE SP_PurchaseProduct
    @ProductId INT,
    @Quantity INT
AS
BEGIN
    SET NOCOUNT ON;

    IF @Quantity <= 0
        THROW 50001, 'Invalid quantity.', 1;

    UPDATE Products
    SET Stock = Stock - @Quantity
    WHERE Id = @ProductId
      AND Stock >= @Quantity;

    IF @@ROWCOUNT = 0
        THROW 50002, 'Insufficient stock.', 1;

    SELECT
        Id,
        ProductName,
        Stock
    FROM Products
    WHERE Id = @ProductId;
END;
```

### Step 4: Execute Procedure

```sql
EXEC SP_PurchaseProduct
    @ProductId = 1,
    @Quantity = 4;
```

প্রথম Customer সফলভাবে 4টি কিনতে পারবে।

দ্বিতীয় Customer-এর জন্য পর্যাপ্ত Stock থাকবে না।

```text
Initial Stock = 5

Customer A = Purchase 4

Remaining Stock = 1

Customer B = Purchase 4

Result = Insufficient Stock
```

এখানে `UPDATE ... WHERE Stock >= @Quantity` একটি Atomic Operation হিসেবে কাজ করছে।

**Professional Tip:** Product Update এবং Order Creation একসঙ্গে করতে হলে Stored Procedure-এর মধ্যে Transaction ব্যবহার করো, যাতে Stock Deduction এবং Order Creation দুটোই সফল হয় অথবা দুটোই Rollback হয়।

## Example 2: Employee Update with RowVersion

### Step 1: Create Table

```sql
CREATE TABLE Employee
(
    Id INT PRIMARY KEY IDENTITY(1,1),

    Name NVARCHAR(100),

    Salary DECIMAL(18,2),

    RowVer ROWVERSION
);
```

### Step 2: Insert Data

```sql
INSERT INTO Employee
(
    Name,
    Salary
)
VALUES
(
    'Abdullah',
    30000
);
```

### Step 3: Create Update Procedure

```sql
CREATE PROCEDURE SP_UpdateEmployee
    @Id INT,
    @Name NVARCHAR(100),
    @Salary DECIMAL(18,2),
    @OriginalRowVer BINARY(8)
AS
BEGIN
    SET NOCOUNT ON;

    UPDATE Employee
    SET
        Name = @Name,
        Salary = @Salary
    WHERE Id = @Id
      AND RowVer = @OriginalRowVer;

    IF @@ROWCOUNT = 0
    BEGIN
        THROW 50003,
        'Concurrency conflict. Data was modified.',
        1;
    END;

    SELECT *
    FROM Employee
    WHERE Id = @Id;
END;
```

এখন দুইজন Admin একই Employee-এর Salary Update করতে চাইলে:

```text
Admin A reads RowVer = Version 1

Admin B reads RowVer = Version 1

Admin A updates Salary

Database generates Version 2

Admin B tries updating with Version 1

Result: Concurrency Conflict
```

এতে একজনের Update অন্যজনের Update-কে চুপচাপ Overwrite করতে পারবে না।

---

# 12. Implementing Concurrency in ASP.NET Core

এখন SQL Server-এর `rowversion` ASP.NET Core API-তে ব্যবহার করি।

## Step 1: Create DTO

```csharp
public class UpdateEmployeeDto
{
    public string Name { get; set; } = string.Empty;

    public decimal Salary { get; set; }

    public byte[] RowVersion { get; set; } = [];
}
```

## Step 2: Create Repository Method

```csharp
public async Task<bool> UpdateEmployeeAsync(
    UpdateEmployeeDto request,
    int id)
{
    using var connection =
        new SqlConnection(_connectionString);

    using var command =
        new SqlCommand(
            "SP_UpdateEmployee",
            connection);

    command.CommandType = CommandType.StoredProcedure;

    command.Parameters.Add(
        "@Id", SqlDbType.Int).Value = id;

    command.Parameters.Add(
        "@Name", SqlDbType.NVarChar, 100).Value =
        request.Name;

    command.Parameters.Add(
        "@Salary", SqlDbType.Decimal).Value =
        request.Salary;

    command.Parameters.Add(
        "@OriginalRowVer", SqlDbType.Binary, 8).Value =
        request.RowVersion;

    await connection.OpenAsync();

    await command.ExecuteNonQueryAsync();

    return true;
}
```

## Step 3: Handle Concurrency Exception

Stored Procedure থেকে `50003` Error আসলে সেটিকে Conflict হিসেবে Handle করো।

```csharp
try
{
    await _repository.UpdateEmployeeAsync(request, id);

    return Ok(new
    {
        message = "Employee updated successfully."
    });
}
catch (SqlException ex) when (ex.Number == 50003)
{
    return Conflict(new
    {
        message = "This record was modified by another user.",
        code = "CONCURRENCY_CONFLICT"
    });
}
```

এখানে Production Application-এ Error Handling Middleware বা Centralized Exception Handler ব্যবহার করা আরও ভালো।

### API Response

<details>
<summary>Success Response — HTTP 200</summary>

```json
{
    "message": "Employee updated successfully."
}
```

</details>

<details>
<summary>Conflict Response — HTTP 409</summary>

```json
{
    "message": "This record was modified by another user.",
    "code": "CONCURRENCY_CONFLICT"
}
```

</details>

---

# 13. Idempotency vs Concurrency

দুটির মধ্যে পার্থক্য বোঝা অত্যন্ত গুরুত্বপূর্ণ।

| Feature | Idempotency | Concurrency |
|---|---|---|
| Main Purpose | Duplicate Operation Prevent | Simultaneous Data Conflict Prevent |
| Main Problem | Same Request Repeated | Multiple Requests at Same Time |
| Focus | Request/Operation | Shared Resource/Data |
| Common Solution | Idempotency Key | Locking, RowVersion, Atomic Update |
| Example | Double Payment | Double Booking |
| Database Feature | Unique Constraint | Transaction, Lock, RowVersion |
| API Feature | Idempotency-Key Header | ETag, If-Match, Conflict Handling |

### সহজভাবে মনে রাখো

**Idempotency:**

একই Operation বারবার Execute হলেও Duplicate Result তৈরি হতে দেবে না।

**Concurrency:**

একাধিক User একই সময়ে কাজ করলেও Database-এর Data Consistent রাখবে।

### Can We Use Both Together?

হ্যাঁ। Professional Software-এ প্রায়ই দুটো একসঙ্গে ব্যবহার করা হয়।

Example: Bus Ticket Booking System

```text
Customer A sends booking request
         |
         v
Idempotency Key Validation
         |
         v
Check duplicate request
         |
         v
Concurrency Control
         |
         v
Check Seat Availability
         |
         v
Lock / Atomic Seat Update
         |
         v
Create Booking
         |
         v
Save Idempotency Result
         |
         v
Return Response
```

এখানে:

- Idempotency একই Booking Request-এর Duplicate Processing বন্ধ করবে।
- Concurrency Control নিশ্চিত করবে একই Seat একাধিক Customer বুক করতে পারবে না।

---

# 14. Real-World Examples

## 14.1 Banking System

**Problem:**

Customer ৳5,000 Transfer করল। Network Timeout-এর কারণে একই Transfer Request আবার পাঠানো হলো।

**Solution:**

- Idempotency Key দিয়ে Duplicate Transfer Prevent।
- Database Transaction দিয়ে Debit এবং Credit একসঙ্গে Commit।
- Atomic Balance Update দিয়ে Lost Update Prevent।
- Transaction Reference-এর ওপর Unique Constraint।

## 14.2 E-commerce System

**Problem:**

একটি Product-এর Stock 10টি। একসঙ্গে 20 জন Customer Purchase করতে চায়।

**Solution:**

- Atomic Stock Deduction।
- Database Transaction।
- Stock Constraint।
- Order Request-এর জন্য Idempotency Key।

## 14.3 Bus Booking System

**Problem:**

একই Seat একই সময়ে দুইজন Customer বুক করতে চায়।

**Solution:**

- Seat Status Check।
- Atomic Seat Reservation।
- Transaction ও Locking।
- Unique Constraint।
- Booking Request-এর জন্য Idempotency Key।

## 14.4 Payroll Management System

**Problem:**

Admin একই Employee-এর জন্য ভুল করে দুইবার Salary Generate Button Click করল।

**Solution:**

- Payroll Period এবং Employee-এর ওপর Unique Constraint।
- Payroll Generation-এর জন্য Idempotency।
- Transaction-এর মাধ্যমে Payroll এবং Ledger Save।
- Concurrent Update-এর জন্য RowVersion।

---

# 15. Professional Best Practices

Production-Level Application তৈরি করার সময় নিচের বিষয়গুলো অনুসরণ করা উচিত।

## Idempotency Best Practices

1. Client থেকে Unique Idempotency Key গ্রহণ করো।
2. User ID এবং Endpoint অনুযায়ী Key Scope করো।
3. Database-এ Unique Constraint ব্যবহার করো।
4. Request Hash দিয়ে Payload Validation করো।
5. Business Operation এবং Idempotency Result একই Transaction-এ Persist করো, যেখানে সম্ভব।
6. Concurrent Duplicate Request Handle করো।
7. Processing এবং Completed State আলাদা রাখো।
8. Key Retention এবং Cleanup Policy তৈরি করো।
9. External Payment Gateway-এর নিজস্ব Idempotency Support ব্যবহার করো।
10. Logs এবং Monitoring রাখো।

## Concurrency Best Practices

1. Database Transaction ব্যবহার করো।
2. Atomic SQL Update ব্যবহার করো।
3. Optimistic Concurrency-এর জন্য `rowversion` ব্যবহার করো।
4. প্রয়োজন অনুযায়ী Pessimistic Locking ব্যবহার করো।
5. Database-এ Unique Constraint তৈরি করো।
6. Foreign Key এবং Check Constraint ব্যবহার করো।
7. Transaction ছোট রাখো।
8. Deadlock Detection এবং Retry Policy তৈরি করো।
9. Conflict হলে User-কে উপযুক্ত Error Return করো।
10. Concurrent Load Testing করো।

## Recommended HTTP Status Codes

| Status Code | Usage |
|---|---|
| 200 OK | Operation সফল |
| 201 Created | নতুন Resource তৈরি |
| 400 Bad Request | Invalid Request |
| 409 Conflict | Idempotency বা Business Conflict |
| 412 Precondition Failed | If-Match / ETag Condition Failed |
| 422 Unprocessable Content | Business Validation Failed |
| 500 Internal Server Error | Unexpected Server Error |

---

# 16. Common Mistakes

| Mistake | Correct Approach |
|---|---|
| শুধু Frontend Button Disable করা | Backend Idempotency Implement |
| শুধু C#-এ Duplicate Check | Database Unique Constraint |
| Check-Then-Insert without Atomicity | Transaction + Unique Constraint |
| Stock আগে Read করে পরে Update | Atomic SQL Update |
| RowVersion ছাড়া Concurrent Update | Original RowVersion Validate |
| সব জায়গায় Pessimistic Lock | প্রয়োজন অনুযায়ী Locking Strategy |
| External Payment-কে Local Transaction-এর অংশ ভাবা | Gateway Idempotency + Reconciliation |
| Infinite Retry | Bounded Retry + Backoff |
| সব Error-কে Retry করা | Retryable এবং Non-retryable Error আলাদা করা |

---

# 17. Interview Questions

### Q1. What is Idempotency?

**Answer:** Idempotency is a property where performing the same operation multiple times produces the same final effect as performing it once.

### Q2. Why is Idempotency important in Payment Systems?

**Answer:** It prevents duplicate payments when clients retry requests due to network failures or timeouts.

### Q3. What is an Idempotency Key?

**Answer:** A unique identifier sent by a client to identify a particular operation. The server uses it to recognize duplicate requests.

### Q4. Is POST Idempotent?

**Answer:** No, POST is not inherently idempotent. However, developers can implement application-level idempotency using an Idempotency Key.

### Q5. What is Concurrency?

**Answer:** Concurrency occurs when multiple users, threads, or processes access or modify shared resources at overlapping times.

### Q6. What is a Race Condition?

**Answer:** A race condition occurs when the result of an operation depends on the timing or execution order of concurrent operations.

### Q7. What is Lost Update?

**Answer:** Lost Update happens when one transaction overwrites another transaction's changes without detecting the conflict.

### Q8. What is Optimistic Concurrency?

**Answer:** Optimistic Concurrency allows operations to proceed without locking the resource during reading and checks for conflicts during updates.

### Q9. What is Pessimistic Concurrency?

**Answer:** Pessimistic Concurrency uses locks to prevent conflicting operations from accessing or modifying a resource simultaneously.

### Q10. What is RowVersion in SQL Server?

**Answer:** RowVersion is an automatically generated binary version value that changes whenever a row is inserted or updated. It can be used for optimistic concurrency control.

### Q11. What is the difference between Idempotency and Concurrency?

**Answer:** Idempotency prevents duplicate execution of the same logical operation, while Concurrency Control protects shared data from conflicting simultaneous operations.

### Q12. Can Idempotency and Concurrency be used together?

**Answer:** Yes. Payment, Booking, Banking and E-commerce systems often use both to prevent duplicate operations and maintain data consistency.

---

# 18. Final Checklist

Before deploying a Production Application, verify:

### Idempotency

- [ ] Idempotency Key implemented.
- [ ] Duplicate Request detection implemented.
- [ ] Request Hash validation implemented.
- [ ] Unique Database Constraint created.
- [ ] Previous Response stored.
- [ ] Concurrent duplicate requests handled.
- [ ] Expiration and cleanup policy defined.

### Concurrency

- [ ] Database Transactions implemented.
- [ ] Atomic SQL Updates implemented.
- [ ] RowVersion implemented where needed.
- [ ] Pessimistic Locking used where appropriate.
- [ ] Unique Constraints implemented.
- [ ] Deadlock handling considered.
- [ ] Concurrent load testing completed.

---

# Conclusion

Idempotency এবং Concurrency হলো Reliable, Scalable এবং Production-Ready Software তৈরির দুটি গুরুত্বপূর্ণ Concept।

মনে রাখবে:

**Idempotency protects against duplicate operations.**

**Concurrency Control protects data consistency during simultaneous operations.**

একটি Professional Developer হিসেবে শুধু API তৈরি করলেই হবে না। Network Failure, Duplicate Requests, Multiple Users এবং Database Conflicts কীভাবে Handle করতে হয়, সেটিও জানতে হবে।

এই দুটি Concept সঠিকভাবে Implement করতে পারলে Payment System, E-commerce, Banking, Inventory এবং Bus Booking-এর মতো Critical Application আরও Reliable করা সম্ভব।

---

**Technologies Covered:**

`C#` | `ASP.NET Core` | `Web API` | `SQL Server` | `ADO.NET` | `Stored Procedures` | `Database Transactions` | `Optimistic Concurrency` | `Pessimistic Concurrency` | `Idempotency Keys`

**Author:** Abdullah Hossain

**Purpose:** Software Engineering Learning & Practical Implementation


