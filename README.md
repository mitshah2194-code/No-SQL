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
