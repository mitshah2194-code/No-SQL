# Complete Tutorial: NoSQL and Azure Cosmos DB

Here's a comprehensive guide covering NoSQL fundamentals and how Azure Cosmos DB fits into the ecosystem.

---

## Part 1: NoSQL Fundamentals

### What is NoSQL?

**NoSQL** stands for "Not Only SQL." It's a category of database management systems designed to handle:
- **Large volumes of unstructured or semi-structured data**
- **High throughput and low latency** (real-time applications)
- **Horizontal scalability** (scale out by adding servers)
- **Flexible schemas** (no rigid structure required)

---

## Part 2: Types of NoSQL Databases

### 1. **Document-Based** 📄
Store and manage data as JSON/BSON documents
- **Examples:** MongoDB, CouchDB, Firebase
- **Use cases:** User profiles, catalogs, content management, e-commerce products
- **Sample structure:**
```json
{
  "user_id": 123,
  "name": "Alice Smith",
  "emails": ["alice@mail.com", "alice@work.com"],
  "address": {
    "city": "New York",
    "zip": "10001"
  },
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

### 2. **Key-Value Stores** 🔑
Simple key-to-value mapping
- **Examples:** Redis, DynamoDB, Memcached
- **Use cases:** Caching, session management, real-time leaderboards, shopping carts
- **Sample:**
```
user:123 -> { "name": "Alice", "email": "alice@mail.com" }
session:abc123 -> { "user_id": 123, "login_time": "2026-05-05T10:30:00Z" }
```

### 3. **Column-Oriented** 📊
Data stored by columns rather than rows
- **Examples:** Apache Cassandra, HBase
- **Use cases:** Analytics, time-series data, big data, IoT sensor data
- **Structure:** Optimized for analytical queries across specific columns

### 4. **Graph-Based** 🔗
Data stored as interconnected nodes and edges
- **Examples:** Neo4j, Amazon Neptune
- **Use cases:** Social networks, recommendation engines, fraud detection, knowledge graphs

---

## Part 3: Core NoSQL Principles

### **CAP Theorem**
Every distributed system can guarantee only **2 out of 3** properties:

| Property | Definition |
|----------|-----------|
| **Consistency** | All nodes see the same data simultaneously |
| **Availability** | System responds to all requests |
| **Partition Tolerance** | System works despite network failures |

**NoSQL Approach:** Most prioritize **Availability** and **Partition Tolerance** (sacrificing immediate consistency)

### **BASE vs. ACID**

| Aspect | ACID (SQL) | BASE (NoSQL) |
|--------|-----------|------------|
| **Consistency** | Strict immediately | Eventual (over time) |
| **Transactions** | ACID guaranteed | Loose transactions |
| **Use Case** | Banking, critical data | Real-time, scalability |

### **Eventual Consistency**
- Data might not be consistent immediately
- All replicas will eventually be consistent
- Good for high-availability, distributed systems

### **Horizontal Scaling (Sharding)**
- Data distributed across multiple servers
- Each server handles a partition of data
- Enables handling massive datasets

---

## Part 4: When to Use NoSQL?

### ✅ **Use NoSQL when:**
- Handling massive amounts of data (big data, IoT)
- Data structure is unstructured or rapidly changing
- Need horizontal scalability
- Real-time analytics required
- High throughput is critical
- Working with different data types

### ❌ **Avoid NoSQL when:**
- Complex multi-table joins needed
- Strict ACID transactions essential (e.g., banking)
- Data structure is rigid and well-defined
- Reporting and analytics on structured data

---

## Part 5: Azure Cosmos DB Overview

**Azure Cosmos DB** is Microsoft's fully-managed, globally distributed NoSQL database service.

### **Key Characteristics**

#### 1. **Globally Distributed** 🌍
- Multi-region replication across Azure regions
- Automatic failover
- 99.999% availability SLA

#### 2. **Multi-Model Support** 🎯
- **SQL API** (Document) – SQL queries on JSON
- **MongoDB API** – Drop-in MongoDB compatibility
- **Cassandra API** – Column-family data model
- **Gremlin API** – Graph queries
- **Table API** – Key-value storage

#### 3. **Schema-Agnostic** 📝
- Flexible schemas – add fields on the fly
- Documents can have different structures

#### 4. **Resource Governance** ⚡
- **Request Units (RU/s)** – Standardized throughput metric
- 1 RU ≈ reading 1 KB of data
- Predictable pricing and performance

#### 5. **Five Consistency Levels** 🔄
| Level | Consistency | Latency | Use Case |
|-------|-------------|---------|----------|
| **Strong** | Highest | Highest | Banking, inventory |
| **Bounded Staleness** | High | Medium | Time-series, stock prices |
| **Session** | Medium | Low | User sessions |
| **Consistent Prefix** | Low | Low | Social feeds |
| **Eventual** | Lowest | Lowest | Analytics, caching |

#### 6. **Automatic Indexing** 📇
- All fields indexed by default
- No manual index management
- Supports range, spatial, and composite indexes

#### 7. **Change Feed** 📡
- Real-time stream of database changes
- Trigger Azure Functions
- Event-driven architectures

---

## Part 6: Core Concepts in Cosmos DB

### **Database Organization**

```
Cosmos DB Account
└── Database (e.g., "ecommerce")
    ├── Container (e.g., "Products")
    │   └── Documents (JSON items)
    ├── Container (e.g., "Orders")
    │   └── Documents
    └── Container (e.g., "Users")
        └── Documents
```

### **Partition Key** 🔑
- Logical grouping of data
- Used for load balancing and scaling
- Essential for query performance
- Example: `"/customerId"`, `"/region"`, `"/userId"`

**Good partition key characteristics:**
- High cardinality (many unique values)
- Evenly distributed queries
- Avoid hot partitions

### **Request Units (RU)** ⚡
Measures database operations:
- **Read 1 KB:** 1 RU
- **Write 1 KB:** 5-10 RU
- **Query:** 1-100+ RU (depending on complexity)

**Pricing models:**
- **Manual Throughput:** Fixed RU/s (pay per hour)
- **Autoscale:** RU/s scales automatically (1-100 RU per operation)
- **Serverless:** Pay per operation (best for unpredictable workloads)

---

## Part 7: Cosmos DB Compared to Other Databases

| Feature | Cosmos DB | MongoDB | DynamoDB | Cassandra |
|---------|-----------|---------|----------|-----------|
| **Global Distribution** | ✅ Native | ❌ Manual | ✅ Multi-region | ✅ Multi-region |
| **Multiple APIs** | ✅ 5 APIs | ❌ MongoDB only | ❌ Proprietary | ❌ Cassandra only |
| **Consistency Levels** | ✅ 5 levels | ❌ 1 level | ❌ Eventually consistent | ❌ Tunable |
| **Ease of Use** | ✅ Managed | ❌ Self-managed | ✅ Managed | ❌ Complex ops |
| **Cost** | 💰 Higher | 💰 Lower (self-managed) | 💰 Medium | 💰 Lower (self-managed) |
| **Learning Curve** | 📚 Easy | 📚 Easy | 📚 Medium | 📚 Hard |

---

## Part 8: Use Cases

### **Ideal for Azure Cosmos DB:**

1. **IoT & Telematics** 📡
   - Time-series sensor data
   - Real-time telemetry processing

2. **E-Commerce** 🛒
   - Product catalogs
   - User shopping carts
   - Order history

3. **Personalization** 🎯
   - User profiles
   - Recommendation engines
   - Real-time personalization

4. **Gaming** 🎮
   - Leaderboards
   - Player profiles
   - In-game inventories

5. **Social Networks** 👥
   - User connections
   - Feed management
   - Comments and notifications

6. **Analytics & Reporting** 📊
   - Real-time dashboards
   - Event analytics
   - User behavior tracking

---

## Part 9: Advantages of NoSQL

✅ **Scalability** – Handle petabytes of data  
✅ **Performance** – Low-latency reads/writes  
✅ **Flexibility** – Schema-less design  
✅ **High Availability** – Built-in replication  
✅ **Developer-Friendly** – JSON structures match code objects  

---

## Part 10: Challenges & Considerations

⚠️ **Learning Curve** – Different query patterns than SQL  
⚠️ **Data Consistency** – Eventual consistency trade-off  
⚠️ **Backup/Recovery** – Different approach than SQL  
⚠️ **Cost** – Can be expensive at scale  
⚠️ **Less Standardization** – Each database has unique syntax  

---

## Part 11: Getting Started with Azure Cosmos DB

### **Quick Start Steps:**

1. **Create Cosmos DB Account**
   - Azure Portal → Create Resource → Azure Cosmos DB
   - Select API: Core (SQL)
   - Configure region, throughput

2. **Create Database & Container**
   ```
   Database: ecommerce
   Container: products
   Partition Key: /categoryId
   ```

3. **Insert Documents**
   ```json
   {
     "id": "prod-123",
     "name": "Laptop",
     "categoryId": "electronics",
     "price": 999.99,
     "stock": 50
   }
   ```

4. **Query Data**
   ```sql
   SELECT * FROM products 
   WHERE products.price > 500 
   ORDER BY products.price DESC
   ```

---

## Part 12: Architecture Best Practices

### **Design Patterns:**

| Pattern | Description |
|---------|-------------|
| **Denormalization** | Embed related data to reduce queries |
| **Array/Subdocument** | Store collections within documents |
| **Partition Strategy** | Choose partition key carefully |
| **TTL (Time-to-Live)** | Auto-delete old data |
| **Change Feed** | Capture and process changes |

### **Performance Tips:**

- 📌 Use appropriate partition key
- 📌 Denormalize related data
- 📌 Minimize cross-partition queries
- 📌 Use indexing policies wisely
- 📌 Monitor RU consumption
- 📌 Batch operations when possible

---

## Summary Table

| Aspect | Details |
|--------|---------|
| **What is NoSQL?** | Non-relational, distributed database systems |
| **Types** | Document, Key-Value, Column, Graph |
| **Key Principle** | Horizontal scalability > strict consistency |
| **Cosmos DB** | Microsoft's managed, global NoSQL service |
| **Best For** | Real-time, high-volume, unstructured data |
| **Trade-offs** | Eventual consistency, complex queries harder |

---

## Next Steps

1. **Hands-on Lab:** Create a Cosmos DB account and insert sample data
2. **Learn C# SDK:** Build applications with Azure Cosmos DB
3. **Query Practice:** Master SQL API queries
4. **Optimization:** Understand RU consumption and scaling
5. **Real-world Project:** Build an e-commerce or IoT application

Would you like me to provide:
- ✅ **C# code samples** for Cosmos DB operations?
- ✅ **Advanced query examples** with joins and aggregations?
- ✅ **Performance optimization** guide?
- ✅ **Architecture diagrams** for specific use cases?

# Complete Guide: C# Examples, Advanced Queries, Performance & Architecture

---

## Section 1: C# Code Samples - CRUD Operations

### **Setup & Initialization**

```csharp
using Microsoft.Azure.Cosmos;
using System;
using System.Threading.Tasks;

public class CosmosDbService
{
    private CosmosClient _cosmosClient;
    private Database _database;
    private Container _container;

    public async Task InitializeAsync()
    {
        string endpointUri = "https://<your-account>.documents.azure.com:443/";
        string primaryKey = "<your-primary-key>";

        _cosmosClient = new CosmosClient(endpointUri, primaryKey);
        
        // Create database if not exists
        _database = await _cosmosClient.CreateDatabaseIfNotExistsAsync("ecommerce");
        
        // Create container with partition key
        _container = await _database.CreateContainerIfNotExistsAsync(
            id: "products",
            partitionKeyPath: "/categoryId",
            throughput: 400
        );
    }
}
```

### **Define Models**

```csharp
using System;
using System.Collections.Generic;

public class Product
{
    public string id { get; set; }
    public string name { get; set; }
    public string categoryId { get; set; }
    public decimal price { get; set; }
    public int stock { get; set; }
    public List<string> tags { get; set; }
    public DateTime createdAt { get; set; }
}

public class Order
{
    public string id { get; set; }
    public string customerId { get; set; }
    public List<OrderItem> items { get; set; }
    public decimal totalAmount { get; set; }
    public string status { get; set; }
    public DateTime orderDate { get; set; }
}

public class OrderItem
{
    public string productId { get; set; }
    public string productName { get; set; }
    public int quantity { get; set; }
    public decimal unitPrice { get; set; }
}
```

---

### **CREATE - Insert Documents**

```csharp
public async Task CreateProductAsync(Product product)
{
    try
    {
        ItemResponse<Product> response = await _container.CreateItemAsync(
            product,
            new PartitionKey(product.categoryId)
        );
        
        Console.WriteLine($"Created product: {response.Resource.name}");
        Console.WriteLine($"RU consumed: {response.RequestCharge}");
    }
    catch (CosmosException ex)
    {
        Console.WriteLine($"Error: {ex.Message}");
    }
}

// Usage
var newProduct = new Product
{
    id = Guid.NewGuid().ToString(),
    name = "Gaming Laptop",
    categoryId = "electronics",
    price = 1299.99m,
    stock = 50,
    tags = new List<string> { "gaming", "laptop", "high-performance" },
    createdAt = DateTime.UtcNow
};

await CreateProductAsync(newProduct);
```

### **READ - Get Single Document**

```csharp
public async Task<Product> GetProductByIdAsync(string productId, string categoryId)
{
    try
    {
        ItemResponse<Product> response = await _container.ReadItemAsync<Product>(
            productId,
            new PartitionKey(categoryId)
        );
        
        Console.WriteLine($"Read cost: {response.RequestCharge} RU");
        return response.Resource;
    }
    catch (CosmosException ex) when (ex.StatusCode == System.Net.HttpStatusCode.NotFound)
    {
        Console.WriteLine("Product not found");
        return null;
    }
}

// Usage
var product = await GetProductByIdAsync("product-123", "electronics");
```

### **UPDATE - Replace Document**

```csharp
public async Task UpdateProductAsync(Product product)
{
    try
    {
        ItemResponse<Product> response = await _container.ReplaceItemAsync(
            product,
            product.id,
            new PartitionKey(product.categoryId)
        );
        
        Console.WriteLine($"Updated: {response.Resource.name}");
        Console.WriteLine($"New price: ${response.Resource.price}");
    }
    catch (CosmosException ex)
    {
        Console.WriteLine($"Update failed: {ex.Message}");
    }
}

// Usage
var product = await GetProductByIdAsync("product-123", "electronics");
if (product != null)
{
    product.price = 1199.99m;
    product.stock -= 5;
    await UpdateProductAsync(product);
}
```

### **DELETE - Remove Document**

```csharp
public async Task DeleteProductAsync(string productId, string categoryId)
{
    try
    {
        ItemResponse<Product> response = await _container.DeleteItemAsync<Product>(
            productId,
            new PartitionKey(categoryId)
        );
        
        Console.WriteLine("Product deleted successfully");
    }
    catch (CosmosException ex)
    {
        Console.WriteLine($"Delete failed: {ex.Message}");
    }
}

// Usage
await DeleteProductAsync("product-123", "electronics");
```

### **UPSERT - Create or Update**

```csharp
public async Task UpsertProductAsync(Product product)
{
    try
    {
        ItemResponse<Product> response = await _container.UpsertItemAsync(
            product,
            new PartitionKey(product.categoryId)
        );
        
        Console.WriteLine($"Upserted: {response.Resource.name}");
    }
    catch (CosmosException ex)
    {
        Console.WriteLine($"Upsert failed: {ex.Message}");
    }
}
```

---

## Section 2: Query Operations

### **Basic Query - SELECT with WHERE**

```csharp
public async Task<List<Product>> GetProductsByPriceAsync(decimal minPrice, decimal maxPrice)
{
    var query = @"
        SELECT * 
        FROM products p 
        WHERE p.price >= @minPrice 
        AND p.price <= @maxPrice
        ORDER BY p.price ASC
    ";
    
    var queryDef = new QueryDefinition(query)
        .WithParameter("@minPrice", minPrice)
        .WithParameter("@maxPrice", maxPrice);
    
    var products = new List<Product>();
    
    using (FeedIterator<Product> resultSet = _container.GetItemQueryIterator<Product>(queryDef))
    {
        while (resultSet.HasMoreResults)
        {
            foreach (var product in await resultSet.ReadNextAsync())
            {
                products.Add(product);
                Console.WriteLine($"{product.name} - ${product.price}");
            }
        }
    }
    
    return products;
}

// Usage
var products = await GetProductsByPriceAsync(500, 2000);
```

### **Query with LINQ (Type-Safe)**

```csharp
public async Task<List<Product>> GetLowStockProductsAsync(int threshold)
{
    var iterator = _container
        .GetItemLinqQueryable<Product>(allowSynchronousQueryExecution: true)
        .Where(p => p.stock < threshold)
        .OrderBy(p => p.stock)
        .ToFeedIterator();
    
    var products = new List<Product>();
    
    while (iterator.HasMoreResults)
    {
        products.AddRange(await iterator.ReadNextAsync());
    }
    
    return products;
}

// Usage
var lowStockItems = await GetLowStockProductsAsync(20);
```

### **Query with Pagination**

```csharp
public async Task<(List<Product>, string)> GetProductsPagedAsync(
    int pageSize = 10, 
    string continuationToken = null)
{
    var query = @"
        SELECT * 
        FROM products p 
        ORDER BY p.id
    ";
    
    var queryDef = new QueryDefinition(query);
    
    var feedOptions = new QueryRequestOptions 
    { 
        MaxItemCount = pageSize 
    };
    
    using (FeedIterator<Product> resultSet = _container
        .GetItemQueryIterator<Product>(queryDef, continuationToken, feedOptions))
    {
        var products = new List<Product>();
        var nextContinuationToken = "";
        
        if (resultSet.HasMoreResults)
        {
            var response = await resultSet.ReadNextAsync();
            products.AddRange(response);
            nextContinuationToken = response.ContinuationToken;
        }
        
        return (products, nextContinuationToken);
    }
}

// Usage
var (firstPage, token) = await GetProductsPagedAsync(pageSize: 10);
Console.WriteLine($"Got {firstPage.Count} items");
var (secondPage, nextToken) = await GetProductsPagedAsync(pageSize: 10, continuationToken: token);
```

### **Query with Distinct**

```csharp
public async Task<List<string>> GetUniqueCategoriesAsync()
{
    var query = @"
        SELECT DISTINCT p.categoryId
        FROM products p
        ORDER BY p.categoryId
    ";
    
    var queryDef = new QueryDefinition(query);
    var categories = new List<string>();
    
    using (FeedIterator<dynamic> resultSet = _container.GetItemQueryIterator<dynamic>(queryDef))
    {
        while (resultSet.HasMoreResults)
        {
            foreach (dynamic item in await resultSet.ReadNextAsync())
            {
                categories.Add(item.categoryId);
            }
        }
    }
    
    return categories;
}
```

### **IN Operator - Query Multiple Values**

```csharp
public async Task<List<Product>> GetProductsByIdsAsync(List<string> productIds)
{
    string idList = string.Join(",", productIds.Select(id => $"'{id}'"));
    
    var query = $@"
        SELECT * 
        FROM products p 
        WHERE p.id IN ({idList})
    ";
    
    var products = new List<Product>();
    
    using (FeedIterator<Product> resultSet = _container.GetItemQueryIterator<Product>(query))
    {
        while (resultSet.HasMoreResults)
        {
            products.AddRange(await resultSet.ReadNextAsync());
        }
    }
    
    return products;
}
```

---

## Section 3: Advanced Queries

### **Aggregation - COUNT**

```csharp
public async Task<int> GetTotalProductCountAsync()
{
    var query = "SELECT VALUE COUNT(1) FROM products p";
    var queryDef = new QueryDefinition(query);
    
    using (FeedIterator<int> resultSet = _container.GetItemQueryIterator<int>(queryDef))
    {
        if (resultSet.HasMoreResults)
        {
            var results = await resultSet.ReadNextAsync();
            return results.FirstOrDefault();
        }
    }
    
    return 0;
}
```

### **Aggregation - SUM, AVG, MIN, MAX**

```csharp
public class ProductStats
{
    public decimal TotalValue { get; set; }
    public decimal AveragePrice { get; set; }
    public decimal MinPrice { get; set; }
    public decimal MaxPrice { get; set; }
    public int TotalStock { get; set; }
}

public async Task<ProductStats> GetProductStatisticsAsync(string categoryId)
{
    var query = @"
        SELECT 
            VALUE {
                totalValue: SUM(p.price * p.stock),
                averagePrice: AVG(p.price),
                minPrice: MIN(p.price),
                maxPrice: MAX(p.price),
                totalStock: SUM(p.stock)
            }
        FROM products p
        WHERE p.categoryId = @categoryId
    ";
    
    var queryDef = new QueryDefinition(query)
        .WithParameter("@categoryId", categoryId);
    
    using (FeedIterator<ProductStats> resultSet = _container.GetItemQueryIterator<ProductStats>(queryDef))
    {
        if (resultSet.HasMoreResults)
        {
            var results = await resultSet.ReadNextAsync();
            return results.FirstOrDefault();
        }
    }
    
    return null;
}

// Usage
var stats = await GetProductStatisticsAsync("electronics");
Console.WriteLine($"Average Price: ${stats.AveragePrice}");
Console.WriteLine($"Total Stock: {stats.TotalStock}");
```

### **GROUP BY - Category Statistics**

```csharp
public class CategoryStats
{
    public string categoryId { get; set; }
    public int productCount { get; set; }
    public decimal averagePrice { get; set; }
    public int totalStock { get; set; }
}

public async Task<List<CategoryStats>> GetCategoryStatisticsAsync()
{
    var query = @"
        SELECT 
            p.categoryId,
            COUNT(1) as productCount,
            AVG(p.price) as averagePrice,
            SUM(p.stock) as totalStock
        FROM products p
        GROUP BY p.categoryId
        ORDER BY p.categoryId
    ";
    
    var stats = new List<CategoryStats>();
    
    using (FeedIterator<CategoryStats> resultSet = _container.GetItemQueryIterator<CategoryStats>(query))
    {
        while (resultSet.HasMoreResults)
        {
            stats.AddRange(await resultSet.ReadNextAsync());
        }
    }
    
    return stats;
}

// Usage
var categoryStats = await GetCategoryStatisticsAsync();
foreach (var stat in categoryStats)
{
    Console.WriteLine($"{stat.categoryId}: {stat.productCount} products, Avg: ${stat.averagePrice}");
}
```

### **JOIN - Array Operations (IntraDocument)**

```csharp
// For documents with nested arrays
public class CustomerWithOrders
{
    public string id { get; set; }
    public string name { get; set; }
    public List<Order> orders { get; set; }
}

public async Task<List<dynamic>> GetOrderDetailsAsync(string customerId)
{
    var query = @"
        SELECT 
            c.name,
            o.id as orderId,
            o.totalAmount,
            o.status,
            ARRAY_LENGTH(o.items) as itemCount
        FROM customers c
        JOIN o IN c.orders
        WHERE c.id = @customerId
        ORDER BY o.orderDate DESC
    ";
    
    var queryDef = new QueryDefinition(query)
        .WithParameter("@customerId", customerId);
    
    var results = new List<dynamic>();
    
    using (FeedIterator<dynamic> resultSet = _container.GetItemQueryIterator<dynamic>(queryDef))
    {
        while (resultSet.HasMoreResults)
        {
            results.AddRange(await resultSet.ReadNextAsync());
        }
    }
    
    return results;
}
```

### **String Functions**

```csharp
public async Task<List<Product>> SearchProductsByNameAsync(string searchTerm)
{
    var query = @"
        SELECT *
        FROM products p
        WHERE CONTAINS(UPPER(p.name), @searchTerm)
        OR STRINGEQUALS(p.tags, @tag, true)
    ";
    
    var queryDef = new QueryDefinition(query)
        .WithParameter("@searchTerm", searchTerm.ToUpper())
        .WithParameter("@tag", searchTerm);
    
    var products = new List<Product>();
    
    using (FeedIterator<Product> resultSet = _container.GetItemQueryIterator<Product>(queryDef))
    {
        while (resultSet.HasMoreResults)
        {
            products.AddRange(await resultSet.ReadNextAsync());
        }
    }
    
    return products;
}

// Usage
var searchResults = await SearchProductsByNameAsync("gaming");
```

---

## Section 4: Transactions & Batch Operations

### **Transactional Batch - Multiple Operations**

```csharp
public async Task<bool> TransferInventoryAsync(
    string fromProductId, 
    string toProductId, 
    string categoryId, 
    int quantity)
{
    try
    {
        // Get current stock
        var fromProduct = await GetProductByIdAsync(fromProductId, categoryId);
        var toProduct = await GetProductByIdAsync(toProductId, categoryId);
        
        if (fromProduct.stock < quantity)
        {
            Console.WriteLine("Insufficient inventory");
            return false;
        }
        
        // Update stock
        fromProduct.stock -= quantity;
        toProduct.stock += quantity;
        
        // Execute as atomic batch
        var batch = _container.CreateTransactionalBatch(new PartitionKey(categoryId))
            .ReplaceItem(fromProduct.id, fromProduct)
            .ReplaceItem(toProduct.id, toProduct);
        
        TransactionalBatchResponse response = await batch.ExecuteAsync();
        
        if (response.IsSuccessStatusCode)
        {
            Console.WriteLine("Inventory transferred successfully");
            return true;
        }
        
        return false;
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Transaction failed: {ex.Message}");
        return false;
    }
}
```

### **Bulk Insert Operations**

```csharp
public async Task BulkInsertProductsAsync(List<Product> products)
{
    var tasks = products.Select(product =>
        _container.CreateItemAsync(
            product,
            new PartitionKey(product.categoryId)
        )
    );
    
    try
    {
        await Task.WhenAll(tasks);
        Console.WriteLine($"Successfully inserted {products.Count} products");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Bulk insert failed: {ex.Message}");
    }
}
```

---

## Section 5: Performance Optimization

### **Custom Indexing Policy**

```csharp
public async Task CreateContainerWithCustomIndexingAsync()
{
    var indexingPolicy = new IndexingPolicy
    {
        IndexingMode = IndexingMode.Consistent,
        // Only index these paths
        IncludedPaths = new Collection<IncludedPath>
        {
            new IncludedPath { Path = "/categoryId/?" },
            new IncludedPath { Path = "/price/?" },
            new IncludedPath { Path = "/name/?" }
        },
        // Exclude these paths from indexing
        ExcludedPaths = new Collection<ExcludedPath>
        {
            new ExcludedPath { Path = "/largeBlob/*" },
            new ExcludedPath { Path = "/temporaryData/*" }
        }
    };
    
    var containerProperties = new ContainerProperties
    {
        Id = "optimizedProducts",
        PartitionKeyPath = "/categoryId",
        IndexingPolicy = indexingPolicy
    };
    
    await _database.CreateContainerAsync(containerProperties, throughput: 400);
}
```

### **Composite Index for Multi-Property Queries**

```csharp
public async Task CreateCompositeIndexAsync()
{
    var indexingPolicy = new IndexingPolicy();
    
    // Composite index for categoryId ASC, price DESC
    indexingPolicy.CompositeIndexes.Add(
        new Collection<CompositePath>
        {
            new CompositePath { Path = "/categoryId", Order = CompositePathSortOrder.Ascending },
            new CompositePath { Path = "/price", Order = CompositePathSortOrder.Descending }
        }
    );
    
    var containerProperties = new ContainerProperties
    {
        Id = "productsWithCompositeIndex",
        PartitionKeyPath = "/categoryId",
        IndexingPolicy = indexingPolicy
    };
    
    await _database.CreateContainerAsync(containerProperties);
}

// This query will be fast with composite index
// SELECT * FROM products WHERE categoryId = 'electronics' ORDER BY price DESC
```

### **Query Cost Analysis**

```csharp
public async Task AnalyzeQueryCostAsync()
{
    var query = @"
        SELECT *
        FROM products p
        WHERE p.price > 1000
        ORDER BY p.price DESC
    ";
    
    var queryDef = new QueryDefinition(query);
    var requestOptions = new QueryRequestOptions();
    
    using (FeedIterator<Product> resultSet = _container
        .GetItemQueryIterator<Product>(queryDef, requestOptions: requestOptions))
    {
        if (resultSet.HasMoreResults)
        {
            var response = await resultSet.ReadNextAsync();
            Console.WriteLine($"Query cost: {response.RequestCharge} RU");
            Console.WriteLine($"Items returned: {response.Count}");
        }
    }
}
```

---

## Section 6: Architecture Diagrams & Patterns

### **E-Commerce Architecture Example**

```
┌─────────────────────────────────────────────────────┐
│         Cosmos DB Account (Global)                   │
├─────────────────────────────────────────────────────┤
│                                                       │
│  Database: ecommerce                                 │
│  ├─ Container: products                              │
│  │  ├─ Partition Key: /categoryId                    │
│  │  └─ Items: Product documents                      │
│  │                                                    │
│  ├─ Container: orders                                │
│  │  ├─ Partition Key: /customerId                    │
│  │  └─ Items: Order documents                        │
│  │                                                    │
│  ├─ Container: customers                             │
│  │  ├─ Partition Key: /id                            │
│  │  └─ Items: Customer documents                     │
│  │                                                    │
│  └─ Container: analytics                             │
│     ├─ Partition Key: /date                          │
│     └─ Items: Aggregated metrics                     │
│                                                       │
│  Replication: Multi-region (US-East, Europe, Asia)  │
│  Throughput: Auto-scale (400-4000 RU/s)             │
│                                                       │
└─────────────────────────────────────────────────────┘

    ↑ ↓
    
┌────────────────────────────────┐
│    C# Application Layer         │
│  (CosmosDbService)              │
├────────────────────────────────┤
│ • CRUD Operations               │
│ • Query Execution               │
│ • Transaction Management        │
│ • Error Handling                │
└────────────────────────────────┘
```

### **Data Modeling Pattern**

```csharp
// Denormalized Product with Related Data
public class ProductWithDetails
{
    public string id { get; set; }
    public string name { get; set; }
    public string categoryId { get; set; }
    public decimal price { get; set; }
    
    // Denormalized category info (avoid cross-partition join)
    public Category category { get; set; }
    
    // Denormalized reviews (limit array size)
    [JsonProperty("topReviews")]
    public List<Review> topReviews { get; set; }
    
    // TTL - auto-delete after 30 days
    public int? ttl { get; set; } = 2592000; // 30 days in seconds
}

public class Category
{
    public string id { get; set; }
    public string name { get; set; }
    public string description { get; set; }
}

public class Review
{
    public int rating { get; set; }
    public string comment { get; set; }
    public string reviewerId { get; set; }
    public DateTime reviewDate { get; set; }
}
```

---

## Section 7: Best Practices Checklist

### **Design Phase** ✅
- [ ] Choose appropriate partition key (high cardinality, even distribution)
- [ ] Identify query patterns upfront
- [ ] Plan for denormalization
- [ ] Consider data retention (TTL)

### **Implementation** ✅
- [ ] Use parameterized queries (prevent injection)
- [ ] Handle CosmosException properly
- [ ] Implement retry logic for transient failures
- [ ] Use LINQ for type-safety where possible
- [ ] Monitor RU consumption

### **Performance** ✅
- [ ] Customize indexing policy for your queries
- [ ] Use composite indexes for multi-property filters
- [ ] Implement pagination for large result sets
- [ ] Avoid cross-partition queries when possible
- [ ] Use point reads (`ReadItemAsync`) for single items

### **Operations** ✅
- [ ] Monitor metrics in Azure Portal
- [ ] Set up alerts for throttling
- [ ] Test scaling scenarios
- [ ] Plan for disaster recovery
- [ ] Use Change Feed for event-driven patterns

---

## Summary Table

| Operation | RU Cost | Best For |
|-----------|---------|----------|
| Point Read | 1 RU | Single item lookup |
| Point Write | 5-10 RU | Single item insert/update |
| Query | 1-100+ RU | Multiple items, filtering |
| Cross-Partition Query | High | Complex queries across data |
| Transaction (batch) | Sum of operations | Atomic multi-item updates |

---

Would you like me to:
- 📊 Create a complete GitHub repository with these examples?
- 🔍 Dive deeper into specific query patterns?
- 🚀 Explain scaling strategies for high-traffic scenarios?
- 🛠️ Set up dependency injection and unit testing examples?
