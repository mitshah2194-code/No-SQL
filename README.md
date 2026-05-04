# No-SQL

In modern application development, **NoSQL** databases and **Azure Cosmos DB** represent a shift away from the rigid tables of traditional relational databases (like SQL Server) toward flexible, horizontally scalable data models.

---

## 🟢 What is NoSQL?
NoSQL ("Not Only SQL") is a category of database management systems designed for large-scale data, high performance, and flexible schemas. 

*   **Schema-agnostic:** Unlike SQL, you don't have to define your columns upfront. One record can have 5 fields, and the next can have 10.
*   **Horizontal Scaling:** Instead of buying a bigger server (Vertical), you add more servers to a cluster (Horizontal) to handle more traffic.
*   **Data Models:**
    *   **Document:** Data is stored as JSON-like documents (e.g., MongoDB).
    *   **Key-Value:** Simple pairs, like a dictionary (e.g., Redis).
    *   **Column-family:** Efficient for reading specific columns across billions of rows (e.g., Cassandra).
    *   **Graph:** Focuses on the relationships between data points (e.g., Neo4j).



---

## ☁️ Azure Cosmos DB: The "Universal" Database
**Azure Cosmos DB** is Microsoft's fully managed, globally distributed NoSQL database service. It is designed to provide "single-digit millisecond" response times anywhere in the world.

### 1. Multi-Model (The "Swiss Army Knife")
Cosmos DB is unique because it supports multiple NoSQL APIs. You can store your data and interact with it using the syntax you already know:
*   **NoSQL API (formerly Core/SQL):** Use JSON documents and query them with SQL-like syntax. This is the most common choice for .NET developers.
*   **MongoDB API:** Use existing MongoDB drivers and tools.
*   **Cassandra API:** For wide-column needs.
*   **Gremlin API:** For graph data.
*   **Table API:** For simple key-value storage.

### 2. Global Distribution
With a few clicks, you can replicate your data across multiple Azure regions. If your users are in Gujarat but also in London, you can place a copy of the database in both locations to reduce latency.

### 3. Five Consistency Levels
Traditional databases usually offer two choices: **Strong** (perfectly accurate but slow) or **Eventual** (fast but might show old data for a second). Cosmos DB offers five levels, allowing you to balance speed and accuracy perfectly for your specific use case.



---

## ⚖️ When to use Cosmos DB?
*   **High Velocity Data:** IoT sensor data, telemetric logs, or real-time retail clickstreams.
*   **Unpredictable Schemas:** Such as a "User Profile" where different users have vastly different attributes (social links, preferences, bio).
*   **Global Reach:** If your app needs to serve users across different continents with zero lag.
*   **Scalability:** If you expect your data to grow from 1GB to 100TB without wanting to manage the underlying server infrastructure.

---

## 💻 Working with .NET Core
Microsoft provides the `Microsoft.Azure.Cosmos` library, which makes integration very clean.

```csharp
// Example: Storing a simple 'Cab' object in Cosmos DB
public class Cab {
    [JsonProperty("id")] // Required for Cosmos
    public string Id { get; set; }
    public string DriverName { get; set; }
    public string CurrentLocation { get; set; }
}

// Inserting a document
Container container = database.GetContainer("CabsContainer");
Cab myCab = new Cab { Id = "CAB-001", DriverName = "Mit", CurrentLocation = "Anand" };
await container.CreateItemAsync(myCab, new PartitionKey(myCab.Id));
```

### Pro-Tip: The Partition Key
In Cosmos DB, the **Partition Key** is the most important decision you will make. It determines how Azure distributes your data across physical servers. A poor choice (like a field that is the same for everyone) will create a "Hot Partition" and ruin your performance. A good choice (like `UserId` or `OrderId`) ensures your data scales evenly.

Are you thinking about using a NoSQL approach for your cab booking platform's real-time tracking or user history?

Implementing these models in **Azure Cosmos DB** is unique because while the database is physically the same, you change the **API** and the **C# SDK** you use to interact with it.

---

## 1. Document Model (NoSQL API)
This is the native "Core" API. It treats data as JSON documents with deep nesting support. It is the most flexible and common choice for .NET developers.



### C# Model
```csharp
public class TripDocument
{
    [JsonProperty("id")]
    public string Id { get; set; } // Unique Trip ID
    
    [JsonProperty("partitionKey")]
    public string DriverId { get; set; } // Partition by Driver
    
    public string Status { get; set; }
    public List<Stop> Route { get; set; } // Nested list
}

public class Stop { public string Location; public DateTime Time; }
```

### Write/Read
```csharp
// Write
await container.CreateItemAsync(newTrip, new PartitionKey(newTrip.DriverId));

// Read (Point Read)
ItemResponse<TripDocument> response = await container.ReadItemAsync<TripDocument>(id, new PartitionKey(driverId));
```

---

## 2. Key-Value Model (Table API)
This is optimized for simple, high-speed lookups using a `PartitionKey` and a `RowKey`. It is less flexible but much cheaper and faster for basic data.



### C# Model
```csharp
using Azure.Data.Tables;

public class SettingEntity : ITableEntity
{
    public string PartitionKey { get; set; } // e.g., "UserConfig"
    public string RowKey { get; set; }       // e.g., "DarkModeEnabled"
    public string Value { get; set; }
    
    public ETag ETag { get; set; }
    public DateTimeOffset? Timestamp { get; set; }
}
```

### Write/Read
```csharp
// Write
await tableClient.AddEntityAsync(newSetting);

// Read
var entity = await tableClient.GetEntityAsync<SettingEntity>("UserConfig", "DarkModeEnabled");
```

---

## 3. Column-Family Model (Cassandra API)
Designed for massive write-heavy workloads (billions of rows). It organizes data into "columns" that are stored together on disk, allowing for efficient queries on specific fields across huge datasets.



### C# Model
```csharp
// Uses the Cassandra C# Driver
public class DriverStat
{
    public Guid DriverId { get; set; }
    public DateTime TripDate { get; set; }
    public decimal Earnings { get; set; }
    public int TotalMinutes { get; set; }
}
```

### Write/Read
```csharp
// Write (using CQL - Cassandra Query Language)
var insertPstmt = session.Prepare("INSERT INTO driver_stats (driver_id, trip_date, earnings) VALUES (?, ?, ?)");
session.Execute(insertPstmt.Bind(driverId, DateTime.Now, 500.50m));

// Read
var row = session.Execute("SELECT earnings FROM driver_stats WHERE driver_id = ...").First();
```

---

## 4. Graph Model (Gremlin API)
This focuses on **relationships** (Edges) between **entities** (Vertices). Excellent for fraud detection or social networks where "who is connected to whom" is the primary question.



### C# Model
In Graph, we don't usually map to a single class; we define the **Vertex** (Entity) and the **Edge** (Relationship).

```csharp
// Using Gremlin.Net
public class Person { public string Name; }
public class Knows { public DateTime Since; }
```

### Write/Read
```csharp
// Write: Create two people and a relationship between them
await g.AddV("person").Property("name", "Mit").Iterate();
await g.AddV("person").Property("name", "Jason").Iterate();
await g.V().Has("name", "Mit").AddE("knows").To(__.V().Has("name", "Jason")).Iterate();

// Read: "Who does Mit know?"
var friends = await g.V().Has("name", "Mit").Out("knows").Values<string>("name").ToList();
```

---

### Comparison for your Cab App

| Model | Use Case | Why? |
| :--- | :--- | :--- |
| **Document** | **Trips & Bookings** | Complex, nested data that changes status. |
| **Key-Value** | **App Settings** | Simple, fast, and low-cost configuration. |
| **Column-Family**| **Historical Analytics**| Efficiently calculating monthly earnings across millions of rows. |
| **Graph** | **Fraud/Referrals** | Finding if the same passenger is using 5 different "referral" accounts. |

For your freelance project, starting with the **Document Model** is almost always the best move, as it covers 90% of business needs.

Preparing for a NoSQL interview requires a solid grasp of how these databases differ from traditional RDBMS, especially regarding scaling and data modeling. Since you are working with **Azure Cosmos DB** and **.NET**, you should be ready for questions that bridge theoretical concepts with practical implementation.

---

## 🏗️ Core Concepts & Architecture

### 1. What is the CAP Theorem, and how does it apply to NoSQL?
*   **The Concept:** CAP stands for **Consistency**, **Availability**, and **Partition Tolerance**. The theorem states that a distributed system can only provide two out of these three guarantees at the same time.
*   **The Nuance:** In NoSQL, since we must have Partition Tolerance (to scale horizontally), we usually choose between Consistency (CP) and Availability (AP).
*   **Cosmos DB Context:** Be prepared to mention that Cosmos DB offers five consistency levels (Strong, Bounded Staleness, Session, Consistent Prefix, and Eventual) to allow developers to "tune" where they sit on the CAP spectrum.



### 2. Explain the difference between Horizontal and Vertical Scaling.
*   **Vertical (Scale Up):** Adding more power (CPU, RAM) to a single server. It has a physical limit and causes downtime.
*   **Horizontal (Scale Out):** Adding more servers to a cluster. NoSQL is designed for this, allowing for nearly infinite growth by sharding data across multiple "nodes."

---

## 📄 Data Modeling & Design

### 3. What is "Sharding" or "Partitioning," and why is the Partition Key choice so important?
*   **Sharding:** The process of breaking up a large dataset into smaller chunks (shards) stored across different servers.
*   **Partition Key:** The property used to determine which shard a piece of data goes to.
*   **Interview Tip:** Explain that a "Good" partition key has high cardinality (many unique values) and avoids "Hot Partitions" (where one server does all the work while others are idle).

### 4. When should you Embed data vs. Reference data in NoSQL?
*   **Embedding (Denormalization):** Storing related data in a single document. Best for "one-to-few" relationships or data that is read together and rarely changes.
*   **Referencing (Normalization):** Storing an ID to a record in another collection. Best for "one-to-many" or "many-to-many" relationships where the related data changes frequently (e.g., a user's wallet balance).

---

## ⚡ Performance & Operations

### 5. What is a "Cross-Partition Query," and why should you avoid it?
*   **The Problem:** A query that doesn't include the Partition Key in the `WHERE` clause.
*   **The Impact:** The database has to "fan out" the query to every single physical partition, which is expensive in terms of RUs (Request Units) and increases latency.

### 6. How do you handle Concurrency in NoSQL without Transactions?
*   **Optimistic Concurrency:** Most NoSQL DBs (like Cosmos DB) use **ETags**. When you update a record, you send the ETag you last saw. If the ETag in the database is different, it means someone else updated it first, and the write fails.
*   **Atomic Operations:** Mention that while NoSQL lacks cross-table transactions, many support atomic operations within a single logical partition (e.g., Stored Procedures in Cosmos DB).

---

## 🛠️ Specialized Questions (Scenario-Based)

### 7. Scenario: "You are building a Cab Booking app. How would you store real-time GPS coordinates?"
*   **Answer:** I would use a **Document model** with **GeoJSON** support. I would use the `driverId` as the Partition Key to ensure all updates for one driver go to the same shard. I would also set a **TTL (Time to Live)** so that old coordinates expire automatically to save storage costs.

### 8. What is the "Change Feed" in Cosmos DB?
*   **The Concept:** A persistent record of changes (inserts and updates) to a container in the order they occurred.
*   **Use Case:** Mention it’s perfect for triggering **Azure Functions** to sync data, send notifications, or update a second "Materialized View" partitioned by a different key.

---

### 💡 Quick Summary Table for Review

| Concept | Key Phrase to Remember |
| :--- | :--- |
| **Base** | Basically Available, Soft state, Eventual consistency (The NoSQL equivalent of ACID). |
| **Schema-less** | Flexibility to store different structures in the same collection. |
| **Request Units (RU)** | The "currency" of Cosmos DB performance (CPU + Memory + IO). |
| **Polyglot Persistence** | Using different types of databases (SQL, NoSQL, Graph) for different parts of one app. |

Since you have an interview coming up for a Senior Manager or Senior Developer role, they might ask about **cost optimization**. Would you like to dive into how to save money on Cosmos DB using **Serverless vs. Provisioned Throughput**?

For a Senior Software Engineer with a focus on the .NET ecosystem, Azure Cosmos DB interview questions will typically move beyond basic definitions and into **performance tuning, cost management, and architectural trade-offs.**

Here are the most common and challenging questions you might encounter, organized by "Senior-level" themes.

---

## 🏗️ Architecture & Consistency

### 1. Can you explain the 5 Consistency Levels in Cosmos DB?
*   **The Question:** Why does Cosmos DB offer more than just "Strong" and "Eventual" consistency?
*   **The Answer:** It's about the **PACELC theorem** (an extension of CAP).
    *   **Strong:** Linearizability. You always read the latest write, but latency is high and it’s limited to a single region for writes.
    *   **Bounded Staleness:** Reads lag behind writes by a specific time interval or number of versions.
    *   **Session (Most Popular):** Within a single client session, you get strong consistency (read-your-own-writes). For everyone else, it's eventual.
    *   **Consistent Prefix:** Updates are seen in the order they were written, but there's a lag.
    *   **Eventual:** Fastest and cheapest, but no guarantee of order or "freshness."


### 2. How does the "Global Distribution" and "Multi-Region Writes" feature work?
*   **Context:** Mention how Cosmos DB allows you to replicate data across any Azure region globally.
*   **Nuance:** Ask about conflict resolution. If two people update the same document in different regions simultaneously, Cosmos DB uses **Last Write Wins (LWW)** by default, but you can write a custom **Merge Procedure**.

---

## 📑 Data Modeling (The "Make or Break" Category)

### 3. What are the three properties of a "Good" Partition Key?
*   **The Answer:** 
    1.  **High Cardinality:** Thousands of unique values (e.g., `UserId`, not `Gender`).
    2.  **Even Distribution:** Spreads both **Storage** and **Throughput (RU/s)** across physical partitions.
    3.  **Query Alignment:** The key appears in the `WHERE` clause of your most frequent/expensive queries to avoid "Fan-out" (Cross-partition) queries.

### 4. When should you use a "Synthetic" Partition Key?
*   **The Scenario:** You have a property like `DeviceID`, but it’s not unique enough, or you need to partition by a combination of fields.
*   **The Solution:** You concatenate two values into one property (e.g., `DeviceID_Date`) and use that as the Partition Key to ensure a more even spread of data.

---

## ⚡ Performance & RU Optimization

### 5. What are Request Units (RUs), and how do you calculate them?
*   **The Answer:** RUs are a deterministic measure of CPU, Memory, and IOPS. 1 RU is the cost of a "Point Read" (reading by ID and Partition Key) of a 1KB document.
*   **Optimization:** Mention that you reduce RUs by:
    *   Limiting the number of properties indexed (Indexing Policy).
    *   Avoiding Cross-Partition queries.
    *   Using "Point Reads" instead of SQL `SELECT` queries where possible.

### 6. Explain the "Change Feed" and its common use cases.
*   **The Concept:** A persistent record of changes to a container.
*   **The .NET Context:** Usually used with **Azure Functions** (Cosmos DB Trigger).
*   **Use Cases:** 
    *   **Materialized Views:** Syncing data to another container with a different partition key.
    *   **Data Lake:** Moving data to Synapse for analytics.
    *   **Microservices:** Triggering a notification or email service when an order status changes.


---

## 🛠️ Implementation & SDK Questions

### 7. What is the difference between `ReadItemAsync` and a SQL Query in the .NET SDK?
*   **Critical distinction:** 
    *   `ReadItemAsync` is a **Point Read**. It requires the ID and Partition Key. It is the cheapest (1 RU) and fastest way to get data.
    *   `Query` (using `GetItemQueryIterator`) involves the query engine, which costs more RUs, even if it only returns one document.

### 8. How do you handle "Rate Limiting" (429 Errors)?
*   **The Answer:** First, the .NET SDK has **built-in retry logic**. You can configure `MaxRetryAttemptsOnRateLimitedRequests`.
*   **Architectural fix:** If you see too many 429s, you either need to **scale your RU/s**, implement **Autoscale**, or fix a **Hot Partition**.

---

## 🏁 Senior/Managerial Scenario
**"We are seeing high costs but low performance on our 'Trips' container. Walk me through your debugging steps."**

1.  **Audit Queries:** Use the **Query Metrics** to find high-RU queries. Are they cross-partition?
2.  **Check Indexing:** Are we indexing large strings or arrays that we never search for? (Write-heavy optimization).
3.  **Monitor Throttling:** Are we hitting 429s? Is the traffic concentrated on one Partition Key (Hot Partition)?
4.  **Evaluate Throughput:** Are we using **Manual Throughput** for a "spiky" workload? If so, I would recommend switching to **Autoscale** to save costs during idle periods.

Are you looking for more details on a specific API, like the **NoSQL API** versus the **MongoDB API**?

Since you're working on a **cab booking platform**, let's look at a production-ready C# example that covers the three most critical scenarios: **creating a trip**, **performing a high-efficiency point read**, and **handling concurrency** to prevent two drivers from accepting the same ride.

This uses the latest `Microsoft.Azure.Cosmos` library (v3).

### 1. The Trip Model
Note the use of `JsonProperty` to map C# naming conventions to the required Cosmos DB `id` and your chosen partition key.

```csharp
using Newtonsoft.Json;

public class Trip
{
    [JsonProperty("id")]
    public string Id { get; set; } // The Unique Trip ID

    [JsonProperty("partitionKey")] 
    public string DriverId { get; set; } // Using DriverId as Partition Key

    public string PassengerName { get; set; }
    public string Status { get; set; } // Requested, Accepted, Completed
    
    [JsonProperty("_etag")]
    public string ETag { get; set; } // For Optimistic Concurrency
}
```

---

### 2. The Implementation (Service Layer)
This example demonstrates how to interact with the container using best practices for performance and safety.



```csharp
using Microsoft.Azure.Cosmos;

public class TripService
{
    private readonly Container _container;

    public TripService(CosmosClient client)
    {
        // Reference the database and container
        _container = client.GetContainer("CabPlatformDb", "Trips");
    }

    // WRITE: Create a new Trip
    public async Task CreateTripAsync(Trip trip)
    {
        // Always pass the partition key for efficiency
        await _container.CreateItemAsync(trip, new PartitionKey(trip.DriverId));
    }

    // READ: The "Point Read" (Cheapest & Fastest - 1 RU)
    public async Task<Trip> GetTripAsync(string tripId, string driverId)
    {
        try
        {
            ItemResponse<Trip> response = await _container.ReadItemAsync<Trip>(
                id: tripId, 
                partitionKey: new PartitionKey(driverId)
            );
            return response.Resource;
        }
        catch (CosmosException ex) when (ex.StatusCode == System.Net.HttpStatusCode.NotFound)
        {
            return null;
        }
    }

    // UPDATE: With Optimistic Concurrency (Preventing Race Conditions)
    public async Task<bool> AcceptTripAsync(Trip trip)
    {
        try
        {
            trip.Status = "Accepted";

            // Pass RequestOptions with the ETag. 
            // If another driver accepted first, the ETag won't match, and this will fail.
            ItemRequestOptions requestOptions = new ItemRequestOptions { 
                IfMatchEtag = trip.ETag 
            };

            await _container.ReplaceItemAsync(
                trip, 
                trip.Id, 
                new PartitionKey(trip.DriverId), 
                requestOptions
            );
            
            return true;
        }
        catch (CosmosException ex) when (ex.StatusCode == System.Net.HttpStatusCode.PreconditionFailed)
        {
            // Conflict detected: Another driver updated this record first
            return false;
        }
    }
}
```

---

### 3. Dependency Injection Setup
In your `Program.cs` (ASP.NET Core), you should register the `CosmosClient` as a **Singleton**. Creating a new client for every request is a common mistake that destroys performance.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register CosmosClient as a Singleton
builder.Services.AddSingleton((s) => {
    string connectionString = builder.Configuration["CosmosDb:ConnectionString"];
    return new CosmosClient(connectionString, new CosmosClientOptions() {
        SerializerOptions = new CosmosSerializationOptions() {
            PropertyNamingPolicy = CosmosPropertyNamingPolicy.CamelCase
        }
    });
});

builder.Services.AddScoped<TripService>();
```

### Key Highlights for your Architecture:
*   **Point Reads:** Notice `GetTripAsync` uses the ID and Partition Key. This hits the exact physical shard and costs the minimum amount of RUs.
*   **Concurrency:** The `IfMatchEtag` logic in `AcceptTripAsync` is the "secret sauce" for a cab app. It ensures that the database itself handles the locking logic so your API stays stateless.
*   **Performance:** By setting the naming policy to `CamelCase` in the client options, you can keep your C# properties capitalized (`DriverId`) while keeping your JSON standard (`driverId`).

To implement a **Geospatial query** in C#, you will use the `Microsoft.Azure.Cosmos.Spatial` namespace. This allows you to find drivers within a specific radius of a passenger using highly optimized spatial indexes.

For your cab booking platform, this is the most efficient way to perform "search for nearby drivers" logic.

### 1. Updated Model with GeoJSON
First, ensure your model uses the `Point` class from the Cosmos SDK. 

```csharp
using Microsoft.Azure.Cosmos.Spatial;
using Newtonsoft.Json;

public class DriverLocation
{
    [JsonProperty("id")]
    public string DriverId { get; set; }

    [JsonProperty("partitionKey")]
    public string City { get; set; } // Partitioning by city is common for regional searches

    public string Status { get; set; } // "Available", "Busy"

    [JsonProperty("currentLocation")]
    public Point CurrentLocation { get; set; } // GeoJSON Point [Longitude, Latitude]
}
```

---

### 2. The Proximity Search Logic
This method calculates the distance on the server side and returns only the drivers within the requested meter radius.

```csharp
public async Task<List<DriverLocation>> FindNearbyDriversAsync(double longitude, double latitude, int radiusMeters)
{
    var drivers = new List<DriverLocation>();
    
    // 1. Define the passenger's center point
    Point passengerLocation = new Point(longitude, latitude);

    // 2. Create the LINQ queryable
    // The .Distance() method is translated into a spatial SQL query by the SDK
    using FeedIterator<DriverLocation> feedIterator = _container.GetItemLinqQueryable<DriverLocation>()
        .Where(d => d.Status == "Available" && 
                    d.CurrentLocation.Distance(passengerLocation) <= radiusMeters)
        .ToFeedIterator();

    // 3. Execute and collect results
    while (feedIterator.HasMoreResults)
    {
        FeedResponse<DriverLocation> response = await feedIterator.ReadNextAsync();
        drivers.AddRange(response);
    }

    return drivers;
}
```

---

### 3. The "Under the Hood" SQL
When you run that C# code, the Cosmos SDK translates it into a specialized SQL query that looks like this:

```sql
SELECT * FROM c 
WHERE c.status = 'Available' 
AND ST_DISTANCE(c.currentLocation, {"type": "Point", "coordinates": [72.92, 22.56]}) <= 2000
```

---

### 🛠️ Critical Setup: The Indexing Policy
A spatial query will be extremely slow and expensive (high RU cost) unless you tell Cosmos DB to index the location property specifically for spatial data. 

In your Azure Portal (or via C# code during container creation), your **Indexing Policy** must look like this:

```json
{
    "indexingMode": "consistent",
    "includedPaths": [ { "path": "/*" } ],
    "spatialIndexes": [
        {
            "path": "/currentLocation/?",
            "types": [ "Point" ]
        }
    ]
}
```

### 💡 Pro-Tips for a Senior Architect:
1.  **Coordinate Order:** Always remember that GeoJSON (and Cosmos DB) uses **[Longitude, Latitude]**. Reversing these is the #1 cause of "empty results" bugs.
2.  **Distance Units:** The `Distance()` function always operates in **meters**.
3.  **Partitioning:** If your app grows globally, you might use a `City` or `Region` as a partition key. In the `FindNearbyDriversAsync` method, you would pass that `PartitionKey` to the query options to ensure it only searches one regional shard, making the search even faster.

For a Senior Developer or Lead role, interviewers want to see that you understand the **trade-offs** of NoSQL. It’s not just about knowing how to write a query; it’s about knowing *why* you chose NoSQL over a Relational database (RDBMS).

Here are the most common and challenging NoSQL interview questions, categorized by topic.

---

## 🏗️ Architecture & Theory

### 1. Explain the CAP Theorem. Which two does your preferred NoSQL DB prioritize?
*   **The Answer:** CAP stands for **Consistency**, **Availability**, and **Partition Tolerance**. In a distributed system, you can only guarantee two.
*   **The Nuance:** Most NoSQL databases (like Cosmos DB or Cassandra) are **AP** (Availability and Partition Tolerance) or **CP** (Consistency and Partition Tolerance).
*   **Senior Tip:** Mention that modern databases like Cosmos DB allow you to "tune" this. You can choose "Strong Consistency" (CP) or "Eventual Consistency" (AP) depending on the business requirement.



### 2. What is "Eventual Consistency"?
*   **The Answer:** It means that if no new updates are made to a data item, eventually all accesses to that item will return the last updated value.
*   **The Trade-off:** You get very high availability and low latency, but a user might see slightly "stale" data for a few milliseconds.
*   **Example:** A "Like" count on a social media post doesn't need to be perfectly accurate every millisecond, making it a great candidate for eventual consistency.

---

## 📑 Data Modeling & Design

### 3. How do you handle "Joins" in NoSQL?
*   **The Answer:** You don't. Joins are expensive in distributed systems. Instead, you use:
    *   **Denormalization (Embedding):** Storing related data together in one document.
    *   **Application-side Joins:** Fetching data from two collections and joining them in your C# code.
    *   **Materialized Views:** Using a tool like the **Cosmos DB Change Feed** to create a pre-joined version of the data for fast reading.

### 4. What is a "Hot Partition" and how do you prevent it?
*   **The Answer:** A Hot Partition occurs when your Partition Key is poorly chosen, causing one physical server to handle all the traffic while others are idle.
*   **Prevention:** Use a high-cardinality key (like `UserId` or `OrderId`) rather than a low-cardinality one (like `Gender` or `Country`).
*   **Example:** If you partition by `Date`, all of today's traffic hits one server. Partitioning by `TransactionID` spreads that traffic evenly.



---

## ⚡ Performance & Scalability

### 5. When would you choose NoSQL over a Relational Database (SQL Server)?
*   **Use NoSQL when:**
    *   You have **massive volumes** of data (Terabytes/Petabytes).
    *   You need **horizontal scaling** (adding more servers).
    *   Your data is **unstructured or semi-structured** (JSON).
    *   You need **rapid development** without strict schema migrations.

### 6. What are Request Units (RUs) in the context of Azure Cosmos DB?
*   **The Answer:** RUs are the "currency" of Cosmos DB. They abstract the CPU, IOPS, and memory required to perform an operation.
*   **The Cost:** A 1KB "Point Read" (reading by ID and Partition Key) costs 1 RU. Writing that same document costs more. A "Cross-Partition Query" can cost hundreds of RUs.

---

## 🛠️ Advanced & Scenario-Based

### 7. How do you implement "Transactions" in NoSQL?
*   **The Answer:** Most NoSQL DBs don't support multi-document transactions like SQL. However:
    *   **Atomicity:** Changes to a single document are atomic.
    *   **Stored Procedures:** In Cosmos DB, you can write JavaScript stored procedures that run transactionally within a **single logical partition**.
    *   **Saga Pattern:** For distributed transactions across different services, you use the Saga pattern with a Message Broker (like Service Bus).

### 8. Explain the "Change Feed" pattern.
*   **The Answer:** It's a listenable stream of changes to your database.
*   **Use Case:** If a user updates their profile in Cosmos DB, the Change Feed triggers an **Azure Function** to:
    1. Update the Search Index.
    2. Send a "Welcome" email.
    3. Push data to a Data Lake for analytics.

---

### 💡 Pro-Interview Tip: The "Why"
Whenever you answer a NoSQL question, always mention **Cost** and **Latency**. 
*   "We choose this partition key to **reduce RU cost**."
*   "We embed this data to **minimize network latency**."

This shows the interviewer you aren't just a coder, but an **Architect** who cares about the production environment.

The **Serverless vs. Provisioned Throughput** debate is one of the most common "Senior Architect" questions because it directly impacts the company's monthly Azure bill.

In Cosmos DB, you don't just pay for storage; you pay for **Request Units (RU/s)**. Here is how the two models compare and how to decide between them.

---

## 💰 The Two Pricing Models

### 1. Provisioned Throughput (Standard)
You "reserve" a specific amount of capacity (e.g., 400 RU/s). You pay for that capacity every hour, whether you use it or not.
*   **Best for:** Steady, predictable traffic.
*   **Autoscale Option:** You can set a range (e.g., 400 to 4,000 RU/s). Azure will automatically scale up when traffic hits and scale down when it's quiet, but you still pay a minimum "base" fee.

### 2. Serverless
There is no "reservation." You are only billed for the exact number of RUs your operations consume. If your app has zero users at 3 AM, your cost is **zero** for that time.
*   **Best for:** Development, testing, small-scale apps, or apps with "bursty" and unpredictable traffic.
*   **Limitations:** It cannot scale as high as Provisioned, and it lacks some advanced features like the "Integrated Cache."

---

## 🚦 Decision Matrix for a Cab App
Since you are architecting a cab booking platform, your traffic likely looks like a "Wave" (peaks during morning/evening commutes and dips at night).

| Scenario | Recommended Model | Reason |
| :--- | :--- | :--- |
| **Development & MVP** | **Serverless** | You don't want to pay \$25/month for a database that only you and a few testers are using. |
| **High-Traffic Production** | **Provisioned (Autoscale)** | Once you have thousands of drivers sending GPS updates, Provisioned is actually cheaper per-request than Serverless. |
| **Analytical Queries** | **Provisioned** | Large "Select" queries for monthly reports consume many RUs; Provisioned ensures you don't hit a "ceiling" and fail. |



---

## 📝 Practice Interview Question: Cost Optimization

**Interviewer:** *"Our Cosmos DB bill is unexpectedly high. What are the first three things you would investigate as a Senior Developer?"*

**Your Answer:**
1.  **Check for Cross-Partition Queries:** I would use the **Query Metrics** to find queries without a Partition Key. These are "expensive" because they hit every physical shard.
2.  **Evaluate Throughput Model:** I would check if we are using "Provisioned" for a low-traffic environment. If our usage is below 10% of our provisioned limit, I’d suggest switching to **Serverless**.
3.  **Review Indexing Policy:** By default, Cosmos DB indexes *every* property. For a write-heavy cab app, I would exclude properties we never query (like `Notes` or `Description`) from the index to save "Write RUs."

---

## 💻 C# Tip: Measuring RU Consumption
In your C# code, you can actually see exactly what each call is costing you. This is great for debugging "expensive" code:

```csharp
var response = await container.CreateItemAsync(myTrip, new PartitionKey(myTrip.DriverId));

// This is the "Magic Number" for cost optimization
double requestCharge = response.RequestCharge; 

Console.WriteLine($"This write cost: {requestCharge} RUs");
```

Would you like to move on to **CAP Theorem** details, or perhaps a mock **System Design** walk-through for your Cab Platform?
