# MongoDB Quick Revision Cheatsheet

## CRUD Operations

### Create

```javascript
// Insert one
db.users.insertOne({ name: "John", age: 30 });

// Insert many
db.users.insertMany([
  { name: "Jane", age: 25 },
  { name: "Bob", age: 35 }
]);
```

### Read

```javascript
// Find all
db.users.find();

// Find with filter
db.users.find({ age: { $gte: 25 } });

// Find one
db.users.findOne({ name: "John" });

// Projection (select fields)
db.users.find({ age: 30 }, { name: 1, email: 1, _id: 0 });

// Sort, limit, skip
db.users.find().sort({ age: -1 }).limit(10).skip(20);

// Count
db.users.countDocuments({ age: { $gte: 25 } });
```

### Update

```javascript
// Update one
db.users.updateOne(
  { name: "John" },
  { $set: { age: 31 } }
);

// Update many
db.users.updateMany(
  { age: { $lt: 18 } },
  { $set: { status: "minor" } }
);

// Replace one
db.users.replaceOne(
  { name: "John" },
  { name: "John", age: 31, email: "john@example.com" }
);

// Upsert (insert if not exists)
db.users.updateOne(
  { email: "new@example.com" },
  { $set: { name: "New User" } },
  { upsert: true }
);
```

### Delete

```javascript
// Delete one
db.users.deleteOne({ name: "John" });

// Delete many
db.users.deleteMany({ age: { $lt: 18 } });

// Delete all
db.users.deleteMany({});
```

## Query Operators

### Comparison

```javascript
$eq   // Equal
$ne   // Not equal
$gt   // Greater than
$gte  // Greater than or equal
$lt   // Less than
$lte  // Less than or equal
$in   // In array
$nin  // Not in array

// Examples
db.users.find({ age: { $gte: 25, $lte: 65 } });
db.users.find({ status: { $in: ["active", "pending"] } });
```

### Logical

```javascript
$and  // And
$or   // Or
$not  // Not
$nor  // Nor

// Examples
db.users.find({
  $and: [
    { age: { $gte: 25 } },
    { status: "active" }
  ]
});

db.users.find({
  $or: [
    { age: { $lt: 18 } },
    { age: { $gt: 65 } }
  ]
});
```

### Element

```javascript
$exists  // Field exists
$type    // Field type

// Examples
db.users.find({ phone: { $exists: true } });
db.users.find({ age: { $type: "number" } });
```

### Array

```javascript
$all        // Contains all elements
$elemMatch  // At least one element matches
$size       // Array size

// Examples
db.posts.find({ tags: { $all: ["mongodb", "database"] } });
db.posts.find({ tags: { $size: 3 } });
```

## Update Operators

```javascript
$set        // Set field value
$unset      // Remove field
$inc        // Increment/decrement
$mul        // Multiply
$rename     // Rename field
$min        // Update if new value is less
$max        // Update if new value is greater
$currentDate // Set to current date

// Array operators
$push       // Add element
$addToSet   // Add if not exists
$pull       // Remove matching elements
$pop        // Remove first/last element
$           // Update first matching array element
$[]         // Update all array elements

// Examples
db.users.updateOne(
  { name: "John" },
  {
    $set: { age: 31 },
    $unset: { tempField: "" },
    $inc: { loginCount: 1 },
    $push: { hobbies: "swimming" }
  }
);
```

## Aggregation Pipeline

### Common Stages

```javascript
// $match - Filter documents
db.orders.aggregate([
  { $match: { status: "completed" } }
]);

// $project - Select/compute fields
db.users.aggregate([
  {
    $project: {
      name: 1,
      age: 1,
      ageGroup: { $cond: [{ $gte: ["$age", 18] }, "adult", "minor"] }
    }
  }
]);

// $group - Group and aggregate
db.orders.aggregate([
  {
    $group: {
      _id: "$customerId",
      totalSpent: { $sum: "$amount" },
      orderCount: { $sum: 1 },
      avgOrder: { $avg: "$amount" }
    }
  }
]);

// $sort - Sort documents
db.users.aggregate([
  { $sort: { age: -1 } }
]);

// $limit - Limit results
db.users.aggregate([
  { $limit: 10 }
]);

// $skip - Skip documents
db.users.aggregate([
  { $skip: 20 }
]);

// $lookup - Join collections
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customer"
    }
  }
]);

// $unwind - Deconstruct array
db.orders.aggregate([
  { $unwind: "$items" }
]);

// $addFields - Add computed fields
db.users.aggregate([
  {
    $addFields: {
      fullName: { $concat: ["$firstName", " ", "$lastName"] }
    }
  }
]);
```

### Aggregation Operators

```javascript
// Arithmetic
$add, $subtract, $multiply, $divide, $mod, $abs

// String
$concat, $substr, $toUpper, $toLower, $split, $trim

// Array
$size, $arrayElemAt, $slice, $filter, $map, $reduce

// Comparison
$eq, $ne, $gt, $gte, $lt, $lte, $cmp

// Conditional
$cond, $ifNull, $switch

// Date
$year, $month, $dayOfMonth, $hour, $minute, $second, $dateToString

// Group accumulators
$sum, $avg, $min, $max, $push, $addToSet, $first, $last
```

## Indexing

### Create Indexes

```javascript
// Single field index
db.users.createIndex({ email: 1 }); // 1 = ascending, -1 = descending

// Compound index
db.users.createIndex({ city: 1, age: -1 });

// Unique index
db.users.createIndex({ email: 1 }, { unique: true });

// Sparse index
db.users.createIndex({ phone: 1 }, { sparse: true });

// Text index
db.articles.createIndex({ title: "text", content: "text" });

// TTL index (auto-delete)
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 3600 }
);

// Partial index
db.users.createIndex(
  { email: 1 },
  { partialFilterExpression: { status: "active" } }
);

// Hashed index (for sharding)
db.users.createIndex({ userId: "hashed" });
```

### Manage Indexes

```javascript
// List indexes
db.users.getIndexes();

// Drop index
db.users.dropIndex("email_1");

// Drop all indexes
db.users.dropIndexes();

// Hide index
db.users.hideIndex("email_1");

// Explain query
db.users.find({ email: "john@example.com" }).explain("executionStats");
```

## Replication

### Replica Set

```javascript
// Initiate replica set
rs.initiate({
  _id: "myReplicaSet",
  members: [
    { _id: 0, host: "server1:27017" },
    { _id: 1, host: "server2:27017" },
    { _id: 2, host: "server3:27017" }
  ]
});

// Check status
rs.status();

// Add member
rs.add("server4:27017");

// Remove member
rs.remove("server4:27017");
```

### Read/Write Concern

```javascript
// Write concern
db.users.insertOne(
  { name: "John" },
  { writeConcern: { w: "majority", j: true } }
);
// w: acknowledgment level
// j: journal commit

// Read concern
db.users.find().readConcern("majority");
// local, available, majority, linearizable

// Read preference
db.users.find().readPref("secondary");
// primary, primaryPreferred, secondary, secondaryPreferred, nearest
```

## Sharding

```javascript
// Enable sharding on database
sh.enableSharding("mydb");

// Shard collection (range-based)
sh.shardCollection("mydb.users", { userId: 1 });

// Shard collection (hashed)
sh.shardCollection("mydb.orders", { orderId: "hashed" });

// Check sharding status
sh.status();

// Move chunk
sh.moveChunk("mydb.users", { userId: 100 }, "shard0001");
```

## Performance Tips

### Query Optimization

```javascript
// ✅ Use indexes
db.users.createIndex({ email: 1 });
db.users.find({ email: "john@example.com" });

// ✅ Use projection
db.users.find({ age: 30 }, { name: 1, email: 1 });

// ✅ Use covered queries
db.users.createIndex({ name: 1, age: 1 });
db.users.find({ name: "John" }, { name: 1, age: 1, _id: 0 });

// ✅ Use $match early in aggregation
db.orders.aggregate([
  { $match: { status: "completed" } }, // Filter first
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } }
]);

// ❌ Avoid $where (slow)
db.users.find({ $where: "this.age > 25" });

// ✅ Use query operators instead
db.users.find({ age: { $gt: 25 } });
```

### Index Strategies

```javascript
// ESR Rule: Equality, Sort, Range
// Query: find({ status: "active", age: { $gte: 25 } }).sort({ name: 1 })
// Index: createIndex({ status: 1, name: 1, age: 1 })

// Compound index supports prefix queries
db.users.createIndex({ a: 1, b: 1, c: 1 });
// Supports: {a}, {a,b}, {a,b,c}
// Not: {b}, {c}, {b,c}
```

## Common Interview Questions

### Q: What is MongoDB?
NoSQL document database that stores data in JSON-like documents.

### Q: RDBMS vs MongoDB?
- RDBMS: Tables, rows, columns, SQL, ACID
- MongoDB: Collections, documents, fields, MQL, flexible schema

### Q: What is a document?
JSON-like object stored in BSON format.

### Q: What is a collection?
Group of MongoDB documents (like a table).

### Q: What is BSON?
Binary JSON - MongoDB's internal storage format.

### Q: What is ObjectId?
12-byte unique identifier for documents.

### Q: What is aggregation?
Pipeline for processing and transforming data.

### Q: What is sharding?
Horizontal scaling by distributing data across servers.

### Q: What is replica set?
Group of MongoDB servers maintaining same data for HA.

### Q: What makes a good shard key?
- High cardinality
- Even distribution
- Query isolation

### Q: Index types?
- Single field
- Compound
- Multikey (arrays)
- Text
- Geospatial
- Hashed
- TTL

### Q: When to use indexes?
- Frequently queried fields
- Sort operations
- Join operations
- Uniqueness constraints

### Q: Write concern?
Level of acknowledgment for write operations.

### Q: Read preference?
Where to direct read operations (primary/secondary).

## Best Practices

✅ Use indexes for frequently queried fields
✅ Use projection to limit returned fields
✅ Use $match early in aggregation pipeline
✅ Follow ESR rule for compound indexes
✅ Use write concern "majority" for critical data
✅ Enable authentication and encryption
✅ Use replica sets for high availability
✅ Choose shard key carefully (can't change easily)
✅ Monitor query performance with explain()
✅ Use connection pooling
✅ Validate data at application level
✅ Use bulk operations for multiple writes
✅ Avoid large documents (>16MB limit)
✅ Use covered queries when possible

## Quick Commands

```javascript
// Database
show dbs                  // List databases
use mydb                  // Switch database
db.dropDatabase()         // Drop current database

// Collections
show collections          // List collections
db.createCollection("users")
db.users.drop()          // Drop collection

// Stats
db.stats()               // Database stats
db.users.stats()         // Collection stats

// Administration
db.serverStatus()
db.currentOp()           // Current operations
db.killOp(opId)          // Kill operation

// Profiling
db.setProfilingLevel(1, { slowms: 100 })
db.system.profile.find()
```

## Key Takeaways

- MongoDB is document-oriented NoSQL database
- Documents stored in collections, use BSON format
- Indexes dramatically improve query performance
- Aggregation pipeline for complex data processing
- Replica sets provide high availability
- Sharding enables horizontal scaling
- Write concern vs read preference tradeoff
- ESR rule for optimal compound indexes
- Use projection and indexes for performance
- Choose shard key wisely for balanced distribution
