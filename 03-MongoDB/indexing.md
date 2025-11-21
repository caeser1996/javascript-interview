# MongoDB Indexing

## Table of Contents
- [What are Indexes?](#what-are-indexes)
- [Index Types](#index-types)
- [Creating Indexes](#creating-indexes)
- [Index Strategies](#index-strategies)
- [Performance Considerations](#performance-considerations)
- [Common Interview Questions](#common-interview-questions)

## What are Indexes?

Indexes are data structures that improve query performance by allowing MongoDB to quickly locate documents without scanning the entire collection.

**Without Index**: O(n) - Scans all documents
**With Index**: O(log n) - Uses B-tree structure

```javascript
// Without index
db.users.find({ email: "john@example.com" }); // Scans all documents

// With index
db.users.createIndex({ email: 1 });
db.users.find({ email: "john@example.com" }); // Uses index
```

## Index Types

### 1. Single Field Index

Index on a single field.

```javascript
// Ascending index
db.users.createIndex({ email: 1 });

// Descending index
db.users.createIndex({ age: -1 });

// Index on nested field
db.users.createIndex({ "address.city": 1 });

// Index on array field
db.posts.createIndex({ tags: 1 });
```

### 2. Compound Index

Index on multiple fields.

```javascript
// Compound index
db.users.createIndex({ city: 1, age: -1 });

// Order matters!
// This index supports:
// - { city: "NYC" }
// - { city: "NYC", age: 30 }
// - { city: "NYC", age: { $gte: 25 } }
//
// But NOT efficiently:
// - { age: 30 } (doesn't use city)
```

### 3. Multikey Index

Automatically created when indexing array fields.

```javascript
// Document
{
  _id: 1,
  name: "John",
  tags: ["mongodb", "javascript", "nodejs"]
}

// Create index
db.posts.createIndex({ tags: 1 });

// Queries that use this index
db.posts.find({ tags: "mongodb" });
db.posts.find({ tags: { $in: ["mongodb", "nodejs"] } });
```

### 4. Text Index

For text search.

```javascript
// Create text index
db.articles.createIndex({ title: "text", content: "text" });

// Search
db.articles.find({ $text: { $search: "mongodb database" } });

// Search with score
db.articles.find(
  { $text: { $search: "mongodb" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } });

// Only one text index per collection
// But can include multiple fields
db.articles.createIndex({
  title: "text",
  content: "text",
  tags: "text"
}, {
  weights: {
    title: 10,
    content: 5,
    tags: 1
  }
});
```

### 5. Geospatial Indexes

For location-based queries.

```javascript
// 2dsphere index (for coordinates)
db.places.createIndex({ location: "2dsphere" });

// Document format
{
  name: "Central Park",
  location: {
    type: "Point",
    coordinates: [-73.965355, 40.782865] // [longitude, latitude]
  }
}

// Find nearby places
db.places.find({
  location: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [-73.9667, 40.78]
      },
      $maxDistance: 1000 // meters
    }
  }
});

// Find within area
db.places.find({
  location: {
    $geoWithin: {
      $centerSphere: [[-73.9667, 40.78], 0.1] // radius in radians
    }
  }
});
```

### 6. Unique Index

Ensures field values are unique.

```javascript
// Unique index
db.users.createIndex({ email: 1 }, { unique: true });

// Compound unique index
db.users.createIndex(
  { email: 1, username: 1 },
  { unique: true }
);

// Sparse unique index (ignores null values)
db.users.createIndex(
  { phone: 1 },
  { unique: true, sparse: true }
);
```

### 7. Partial Index

Indexes only documents that meet filter criteria.

```javascript
// Index only active users
db.users.createIndex(
  { email: 1 },
  {
    partialFilterExpression: {
      status: "active",
      age: { $gte: 18 }
    }
  }
);

// Only uses index if query matches filter
db.users.find({ email: "john@example.com", status: "active" }); // Uses index
db.users.find({ email: "john@example.com" }); // Doesn't use index
```

### 8. TTL Index

Automatically deletes documents after specified time.

```javascript
// Expire after 24 hours
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 86400 } // 24 * 60 * 60
);

// Document
{
  _id: ObjectId("..."),
  sessionId: "abc123",
  createdAt: new Date()
}

// TTL thread runs every 60 seconds
// Documents are deleted when: createdAt + expireAfterSeconds < current time
```

### 9. Hashed Index

For hash-based sharding.

```javascript
// Hashed index
db.users.createIndex({ userId: "hashed" });

// Used for even distribution in sharding
// Not used for range queries
```

## Creating Indexes

### Basic Syntax

```javascript
// Create index
db.collection.createIndex(
  { field: 1 },      // Keys
  { options }        // Options
);

// Create multiple indexes
db.collection.createIndexes([
  { key: { field1: 1 } },
  { key: { field2: -1 } },
  { key: { field3: 1, field4: 1 } }
]);
```

### Index Options

```javascript
// Unique index
db.users.createIndex(
  { email: 1 },
  { unique: true }
);

// Sparse index (doesn't index null/missing values)
db.users.createIndex(
  { phone: 1 },
  { sparse: true }
);

// Background index (doesn't block reads/writes)
db.users.createIndex(
  { age: 1 },
  { background: true }
);

// Partial index
db.users.createIndex(
  { status: 1 },
  {
    partialFilterExpression: {
      age: { $gte: 18 }
    }
  }
);

// Named index
db.users.createIndex(
  { email: 1 },
  { name: "email_index" }
);

// TTL index
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 3600 }
);

// Case-insensitive index
db.users.createIndex(
  { email: 1 },
  { collation: { locale: "en", strength: 2 } }
);
```

### Managing Indexes

```javascript
// List all indexes
db.users.getIndexes();

// Drop index by name
db.users.dropIndex("email_1");

// Drop index by keys
db.users.dropIndex({ email: 1 });

// Drop all indexes (except _id)
db.users.dropIndexes();

// Rebuild indexes
db.users.reIndex();

// Hide index (makes it invisible to query planner)
db.users.hideIndex("email_1");

// Unhide index
db.users.unhideIndex("email_1");
```

## Index Strategies

### Equality, Sort, Range (ESR) Rule

Order compound index fields by: Equality → Sort → Range

```javascript
// Query
db.users.find({
  country: "USA",        // Equality
  age: { $gte: 25 }      // Range
}).sort({ name: 1 });    // Sort

// Optimal index: ESR rule
db.users.createIndex({
  country: 1,  // Equality first
  name: 1,     // Sort second
  age: 1       // Range last
});
```

### Covering Indexes

Index contains all fields needed by query (no document access needed).

```javascript
// Create index with all query fields
db.users.createIndex({ name: 1, email: 1, age: 1 });

// Covered query (only uses index)
db.users.find(
  { name: "John" },
  { name: 1, email: 1, age: 1, _id: 0 }  // Exclude _id
);

// Check if covered
db.users.find(
  { name: "John" },
  { name: 1, email: 1, age: 1, _id: 0 }
).explain("executionStats");
// Look for: totalDocsExamined: 0 (didn't access documents)
```

### Index Intersection

MongoDB can use multiple indexes for a single query.

```javascript
// Two indexes
db.users.createIndex({ age: 1 });
db.users.createIndex({ city: 1 });

// Query uses both indexes
db.users.find({ age: 30, city: "NYC" });

// But compound index is usually better
db.users.createIndex({ age: 1, city: 1 });
```

### Prefix Compression

Compound indexes support prefix queries.

```javascript
// Compound index
db.users.createIndex({ a: 1, b: 1, c: 1 });

// Queries that use this index:
db.users.find({ a: 1 });              // ✅ Uses index
db.users.find({ a: 1, b: 2 });        // ✅ Uses index
db.users.find({ a: 1, b: 2, c: 3 });  // ✅ Uses index
db.users.find({ a: 1, c: 3 });        // ✅ Partially uses index (only a)

// Queries that DON'T use this index:
db.users.find({ b: 2 });              // ❌ Doesn't start with 'a'
db.users.find({ c: 3 });              // ❌ Doesn't start with 'a'
db.users.find({ b: 2, c: 3 });        // ❌ Doesn't start with 'a'
```

## Performance Considerations

### Index Size

```javascript
// Check index size
db.users.stats().indexSizes;

// Output:
{
  "_id_": 200000,
  "email_1": 150000,
  "age_1_city_1": 180000
}

// Indexes consume RAM and disk space
// Too many indexes slow down writes
```

### Write Performance Impact

```javascript
// Each index adds overhead to writes
// Insert: Update all indexes
// Update: Update affected indexes
// Delete: Update all indexes

// Example:
// Collection with 5 indexes
// Each insert updates 6 structures (document + 5 indexes)

// Balance read vs write performance
```

### Analyze Queries

```javascript
// Explain query
db.users.find({ email: "john@example.com" }).explain("executionStats");

// Key fields to check:
// - executionTimeMillis: Query execution time
// - totalDocsExamined: Documents scanned
// - totalKeysExamined: Index entries scanned
// - executionStages.stage: "IXSCAN" = index used, "COLLSCAN" = collection scan

// Ideal: totalDocsExamined === nReturned (no extra docs scanned)
```

### Query Optimization

```javascript
// ❌ Bad: Collection scan
db.users.find({ age: { $gte: 25 } }).explain();
// Stage: COLLSCAN

// ✅ Good: Index scan
db.users.createIndex({ age: 1 });
db.users.find({ age: { $gte: 25 } }).explain();
// Stage: IXSCAN

// ❌ Bad: Non-selective index
db.users.createIndex({ isActive: 1 }); // Only 2 values: true/false
// Low cardinality = less effective

// ✅ Good: Selective index
db.users.createIndex({ email: 1 }); // High cardinality = more effective
```

## Common Interview Questions

### Q1: What is the default index in MongoDB?

**Answer**: Every collection has a default index on the `_id` field. It's unique and cannot be dropped.

```javascript
db.users.getIndexes();
// [ { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_" } ]
```

### Q2: When should you create an index?

**Answer**:
- Fields used in queries frequently
- Fields used for sorting
- Fields used in joins ($lookup)
- Fields needing uniqueness constraint

**Don't index**:
- Fields rarely queried
- Low cardinality fields (few unique values)
- Collections with frequent writes and rare reads

### Q3: Difference between single and compound indexes?

**Answer**:
- **Single**: Index on one field
- **Compound**: Index on multiple fields, supports prefix queries

```javascript
// Single index
db.users.createIndex({ age: 1 });
// Supports: { age: 30 }

// Compound index
db.users.createIndex({ city: 1, age: 1 });
// Supports: { city: "NYC" }, { city: "NYC", age: 30 }
// Not efficient: { age: 30 }
```

### Q4: What is index selectivity?

**Answer**: Selectivity is the ratio of distinct values to total documents. Higher selectivity = more effective index.

```javascript
// High selectivity (good)
// email: 10,000 unique values / 10,000 documents = 1.0

// Low selectivity (less effective)
// gender: 2 unique values / 10,000 documents = 0.0002

// Rule: Index high-cardinality fields first in compound indexes
```

### Q5: Explain covered queries

**Answer**: A covered query is one where all fields are in the index, so MongoDB doesn't need to access documents.

```javascript
// Index
db.users.createIndex({ name: 1, age: 1 });

// Covered query (fast)
db.users.find(
  { name: "John" },
  { name: 1, age: 1, _id: 0 }  // Must exclude _id
);

// Not covered (slower)
db.users.find(
  { name: "John" },
  { name: 1, age: 1, email: 1 }  // email not in index
);
```

### Q6: What is the ESR rule?

**Answer**: ESR (Equality, Sort, Range) is the optimal order for compound index fields.

```javascript
// Query
db.orders.find({
  status: "completed",      // Equality
  amount: { $gte: 100 }     // Range
}).sort({ createdAt: -1 }); // Sort

// Optimal index (ESR)
db.orders.createIndex({
  status: 1,      // Equality
  createdAt: -1,  // Sort
  amount: 1       // Range
});
```

### Q7: Impact of indexes on write performance?

**Answer**: Each index slows down writes because:
- Insert: All indexes must be updated
- Update: Affected indexes must be updated
- Delete: All indexes must be updated

Balance: Index for reads, but don't over-index.

### Q8: How to identify missing indexes?

**Answer**:
1. Use `.explain()` to find COLLSCAN
2. Enable profiling to log slow queries
3. Use MongoDB Atlas Performance Advisor
4. Monitor query patterns

```javascript
// Enable profiling (logs queries > 100ms)
db.setProfilingLevel(1, { slowms: 100 });

// View slow queries
db.system.profile.find().sort({ ts: -1 }).limit(10);

// Find queries without indexes
db.system.profile.find({ planSummary: "COLLSCAN" });
```

## Best Practices

1. **Create indexes based on query patterns**
   ```javascript
   // Analyze your queries first
   // Index fields used in WHERE, JOIN, ORDER BY
   ```

2. **Use compound indexes for multiple fields**
   ```javascript
   // Better: One compound index
   db.users.createIndex({ city: 1, age: 1 });

   // Worse: Multiple single indexes
   db.users.createIndex({ city: 1 });
   db.users.createIndex({ age: 1 });
   ```

3. **Follow ESR rule for compound indexes**

4. **Use partial indexes to save space**
   ```javascript
   db.users.createIndex(
     { email: 1 },
     { partialFilterExpression: { isActive: true } }
   );
   ```

5. **Monitor index usage**
   ```javascript
   db.users.aggregate([{ $indexStats: {} }]);
   ```

6. **Drop unused indexes**
   ```javascript
   // Find unused indexes
   db.users.aggregate([
     { $indexStats: {} },
     { $match: { "accesses.ops": { $lt: 10 } } }
   ]);
   ```

7. **Use background builds for large collections**
   ```javascript
   db.users.createIndex({ age: 1 }, { background: true });
   ```

## Key Takeaways

1. Indexes dramatically improve read performance
2. Too many indexes hurt write performance
3. Compound indexes support prefix queries
4. ESR rule optimizes compound indexes
5. Use `.explain()` to verify index usage
6. Covered queries are fastest (index-only)
7. Monitor and drop unused indexes
8. Balance read performance vs write performance

## Practice Problems

1. Design indexes for an e-commerce product search
2. Optimize a slow query using explain plan
3. Create compound index following ESR rule
4. Implement geospatial search for nearby locations
5. Set up TTL index for session management
6. Identify and drop unused indexes
7. Design indexes for a social media feed query

---

**Next Topic**: [Replication & Sharding](./replication-sharding.md)
