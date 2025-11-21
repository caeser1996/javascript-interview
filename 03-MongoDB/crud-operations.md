# MongoDB CRUD Operations

## Table of Contents
- [Create Operations](#create-operations)
- [Read Operations](#read-operations)
- [Update Operations](#update-operations)
- [Delete Operations](#delete-operations)
- [Query Operators](#query-operators)
- [Common Interview Questions](#common-interview-questions)

## Create Operations

### insertOne()

Insert a single document into a collection.

```javascript
// Insert one document
db.users.insertOne({
  name: "John Doe",
  email: "john@example.com",
  age: 30,
  createdAt: new Date()
});

// Returns
{
  acknowledged: true,
  insertedId: ObjectId("507f1f77bcf86cd799439011")
}
```

### insertMany()

Insert multiple documents at once.

```javascript
db.users.insertMany([
  {
    name: "Jane Smith",
    email: "jane@example.com",
    age: 25
  },
  {
    name: "Bob Johnson",
    email: "bob@example.com",
    age: 35
  },
  {
    name: "Alice Williams",
    email: "alice@example.com",
    age: 28
  }
]);

// Returns
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId("507f1f77bcf86cd799439012"),
    '1': ObjectId("507f1f77bcf86cd799439013"),
    '2': ObjectId("507f1f77bcf86cd799439014")
  }
}

// With ordered: false (continues on error)
db.users.insertMany(
  [{ name: "User1" }, { _id: 1, name: "User2" }, { name: "User3" }],
  { ordered: false }
);
```

## Read Operations

### find()

Query documents from a collection.

```javascript
// Find all documents
db.users.find();

// Find with filter
db.users.find({ age: { $gte: 25 } });

// Find with projection (select specific fields)
db.users.find(
  { age: { $gte: 25 } },
  { name: 1, email: 1, _id: 0 }
);

// Find with limit and sort
db.users.find()
  .sort({ age: -1 })  // -1 = descending, 1 = ascending
  .limit(5);

// Find with skip (pagination)
db.users.find()
  .skip(10)
  .limit(10);

// Count documents
db.users.find({ age: { $gte: 25 } }).count();
// or
db.users.countDocuments({ age: { $gte: 25 } });
```

### findOne()

Return a single document.

```javascript
// Find one document
db.users.findOne({ email: "john@example.com" });

// Find by ObjectId
db.users.findOne({ _id: ObjectId("507f1f77bcf86cd799439011") });

// Find one with projection
db.users.findOne(
  { email: "john@example.com" },
  { name: 1, age: 1 }
);
```

### Advanced Queries

```javascript
// AND condition (implicit)
db.users.find({
  age: { $gte: 25 },
  city: "New York"
});

// OR condition
db.users.find({
  $or: [
    { age: { $lt: 25 } },
    { age: { $gt: 60 } }
  ]
});

// Complex query
db.users.find({
  $and: [
    { age: { $gte: 25 } },
    {
      $or: [
        { city: "New York" },
        { city: "Los Angeles" }
      ]
    }
  ]
});

// Regex search
db.users.find({
  name: { $regex: /^John/i }  // Case-insensitive, starts with "John"
});

// Array queries
db.users.find({
  hobbies: "reading"  // Array contains "reading"
});

db.users.find({
  hobbies: { $all: ["reading", "gaming"] }  // Contains both
});

db.users.find({
  hobbies: { $size: 3 }  // Array has exactly 3 elements
});

// Nested document queries
db.users.find({
  "address.city": "New York"
});

// Exists operator
db.users.find({
  phone: { $exists: true }
});
```

## Update Operations

### updateOne()

Update a single document.

```javascript
// Update one document
db.users.updateOne(
  { email: "john@example.com" },  // Filter
  { $set: { age: 31 } }            // Update
);

// Returns
{
  acknowledged: true,
  matchedCount: 1,
  modifiedCount: 1
}

// Multiple field update
db.users.updateOne(
  { email: "john@example.com" },
  {
    $set: {
      age: 31,
      city: "Boston",
      updatedAt: new Date()
    }
  }
);

// Upsert (insert if not exists)
db.users.updateOne(
  { email: "newuser@example.com" },
  {
    $set: {
      name: "New User",
      age: 25
    }
  },
  { upsert: true }
);
```

### updateMany()

Update multiple documents.

```javascript
// Update all users in New York
db.users.updateMany(
  { city: "New York" },
  { $set: { timezone: "EST" } }
);

// Increment age for all users
db.users.updateMany(
  {},
  { $inc: { age: 1 } }
);
```

### replaceOne()

Replace an entire document.

```javascript
db.users.replaceOne(
  { email: "john@example.com" },
  {
    name: "John Doe",
    email: "john@example.com",
    age: 31,
    city: "Boston"
    // Old fields are removed, only these remain
  }
);
```

### Update Operators

```javascript
// $set - Set field value
db.users.updateOne(
  { _id: ObjectId("...") },
  { $set: { status: "active" } }
);

// $unset - Remove field
db.users.updateOne(
  { _id: ObjectId("...") },
  { $unset: { tempField: "" } }
);

// $inc - Increment/decrement
db.users.updateOne(
  { _id: ObjectId("...") },
  { $inc: { age: 1, score: -10 } }
);

// $mul - Multiply
db.users.updateOne(
  { _id: ObjectId("...") },
  { $mul: { price: 1.1 } }  // Increase by 10%
);

// $rename - Rename field
db.users.updateOne(
  { _id: ObjectId("...") },
  { $rename: { "name": "fullName" } }
);

// $min - Update if new value is less than current
db.users.updateOne(
  { _id: ObjectId("...") },
  { $min: { lowestScore: 95 } }
);

// $max - Update if new value is greater than current
db.users.updateOne(
  { _id: ObjectId("...") },
  { $max: { highestScore: 100 } }
);

// $currentDate - Set to current date
db.users.updateOne(
  { _id: ObjectId("...") },
  { $currentDate: { lastModified: true } }
);

// Array operators
// $push - Add element to array
db.users.updateOne(
  { _id: ObjectId("...") },
  { $push: { hobbies: "swimming" } }
);

// $push with $each - Add multiple elements
db.users.updateOne(
  { _id: ObjectId("...") },
  { $push: { hobbies: { $each: ["swimming", "hiking"] } } }
);

// $addToSet - Add if not exists (no duplicates)
db.users.updateOne(
  { _id: ObjectId("...") },
  { $addToSet: { tags: "important" } }
);

// $pull - Remove elements matching condition
db.users.updateOne(
  { _id: ObjectId("...") },
  { $pull: { hobbies: "gaming" } }
);

// $pop - Remove first/last element
db.users.updateOne(
  { _id: ObjectId("...") },
  { $pop: { hobbies: 1 } }  // 1 = last, -1 = first
);

// $ positional operator - Update first matching array element
db.users.updateOne(
  { _id: ObjectId("..."), "scores.subject": "math" },
  { $set: { "scores.$.grade": 95 } }
);

// $[] - Update all array elements
db.users.updateOne(
  { _id: ObjectId("...") },
  { $inc: { "scores.$[].grade": 5 } }
);

// $[identifier] - Update matching array elements
db.users.updateOne(
  { _id: ObjectId("...") },
  { $inc: { "scores.$[elem].grade": 10 } },
  { arrayFilters: [{ "elem.grade": { $gte: 85 } }] }
);
```

## Delete Operations

### deleteOne()

Delete a single document.

```javascript
db.users.deleteOne({ email: "john@example.com" });

// Returns
{
  acknowledged: true,
  deletedCount: 1
}

// Delete by ObjectId
db.users.deleteOne({ _id: ObjectId("507f1f77bcf86cd799439011") });
```

### deleteMany()

Delete multiple documents.

```javascript
// Delete all users older than 65
db.users.deleteMany({ age: { $gt: 65 } });

// Delete all documents
db.users.deleteMany({});

// Delete with complex condition
db.users.deleteMany({
  $and: [
    { status: "inactive" },
    { lastLogin: { $lt: new Date("2023-01-01") } }
  ]
});
```

### findOneAndDelete()

Find and delete a document, returning the deleted document.

```javascript
const deletedUser = db.users.findOneAndDelete(
  { email: "john@example.com" }
);

// Returns the deleted document
{
  _id: ObjectId("..."),
  name: "John Doe",
  email: "john@example.com",
  age: 30
}
```

## Query Operators

### Comparison Operators

```javascript
// $eq - Equal to
db.users.find({ age: { $eq: 30 } });
// Same as: db.users.find({ age: 30 });

// $ne - Not equal to
db.users.find({ status: { $ne: "inactive" } });

// $gt - Greater than
db.users.find({ age: { $gt: 25 } });

// $gte - Greater than or equal to
db.users.find({ age: { $gte: 25 } });

// $lt - Less than
db.users.find({ age: { $lt: 65 } });

// $lte - Less than or equal to
db.users.find({ age: { $lte: 65 } });

// $in - Matches any value in array
db.users.find({ city: { $in: ["New York", "Los Angeles", "Chicago"] } });

// $nin - Matches none of the values in array
db.users.find({ status: { $nin: ["inactive", "banned"] } });
```

### Logical Operators

```javascript
// $and
db.users.find({
  $and: [
    { age: { $gte: 25 } },
    { age: { $lte: 65 } }
  ]
});

// $or
db.users.find({
  $or: [
    { city: "New York" },
    { city: "Los Angeles" }
  ]
});

// $not
db.users.find({
  age: { $not: { $gt: 65 } }
});

// $nor - None of the conditions are true
db.users.find({
  $nor: [
    { status: "inactive" },
    { age: { $lt: 18 } }
  ]
});
```

### Element Operators

```javascript
// $exists - Field exists or not
db.users.find({ phone: { $exists: true } });

// $type - Field type
db.users.find({ age: { $type: "number" } });
db.users.find({ age: { $type: ["number", "string"] } });

// BSON Types
// "double", "string", "object", "array", "binData", "undefined",
// "objectId", "bool", "date", "null", "regex", "int", "timestamp", "long"
```

### Array Operators

```javascript
// $all - Array contains all specified elements
db.users.find({
  hobbies: { $all: ["reading", "gaming"] }
});

// $elemMatch - At least one array element matches all conditions
db.users.find({
  scores: {
    $elemMatch: { $gte: 80, $lt: 90 }
  }
});

// $size - Array has specific length
db.users.find({
  hobbies: { $size: 3 }
});
```

## Common Interview Questions

### Q1: Insert vs InsertOne vs InsertMany

```javascript
// insertOne - Insert single document
db.users.insertOne({ name: "John" });

// insertMany - Insert multiple documents
db.users.insertMany([
  { name: "John" },
  { name: "Jane" }
]);

// insert (deprecated) - Can insert one or many
db.users.insert({ name: "John" });  // Use insertOne instead
db.users.insert([{ name: "John" }]); // Use insertMany instead
```

**Answer**: Use `insertOne()` for single documents and `insertMany()` for multiple. The old `insert()` method is deprecated.

### Q2: Update vs UpdateOne vs UpdateMany

```javascript
// updateOne - Updates first matching document
db.users.updateOne(
  { age: 30 },
  { $set: { status: "active" } }
);

// updateMany - Updates all matching documents
db.users.updateMany(
  { age: 30 },
  { $set: { status: "active" } }
);

// replaceOne - Replaces entire document
db.users.replaceOne(
  { _id: ObjectId("...") },
  { name: "New Name", age: 30 }  // Only these fields will exist
);
```

### Q3: What is Upsert?

```javascript
// Upsert: Update if exists, insert if not
db.users.updateOne(
  { email: "newuser@example.com" },
  {
    $set: {
      name: "New User",
      age: 25,
      createdAt: new Date()
    }
  },
  { upsert: true }  // Insert if document doesn't exist
);

// Use $setOnInsert for fields only set on insert
db.users.updateOne(
  { email: "user@example.com" },
  {
    $set: { lastLogin: new Date() },
    $setOnInsert: { createdAt: new Date(), status: "new" }
  },
  { upsert: true }
);
```

### Q4: How to update nested fields?

```javascript
// Dot notation
db.users.updateOne(
  { _id: ObjectId("...") },
  { $set: { "address.city": "New York" } }
);

// Update entire nested object
db.users.updateOne(
  { _id: ObjectId("...") },
  {
    $set: {
      address: {
        street: "123 Main St",
        city: "New York",
        zip: "10001"
      }
    }
  }
);

// Update array element
db.users.updateOne(
  { _id: ObjectId("..."), "scores.subject": "math" },
  { $set: { "scores.$.grade": 95 } }
);
```

### Q5: Bulk Operations

```javascript
// Bulk write for multiple operations
db.users.bulkWrite([
  {
    insertOne: {
      document: { name: "User1", age: 25 }
    }
  },
  {
    updateOne: {
      filter: { name: "User2" },
      update: { $set: { age: 30 } }
    }
  },
  {
    deleteOne: {
      filter: { name: "User3" }
    }
  }
], { ordered: false });  // Continue on error

// Bulk operations are more efficient than individual operations
```

### Q6: FindAndModify Operations

```javascript
// findOneAndUpdate - Returns document before or after update
const updatedUser = db.users.findOneAndUpdate(
  { email: "john@example.com" },
  { $set: { age: 31 } },
  { returnNewDocument: true }  // Return updated document
);

// findOneAndReplace
const replacedUser = db.users.findOneAndReplace(
  { email: "john@example.com" },
  { name: "John", email: "john@example.com", age: 31 },
  { returnNewDocument: true }
);

// findOneAndDelete - Returns deleted document
const deletedUser = db.users.findOneAndDelete(
  { email: "john@example.com" }
);
```

## Best Practices

1. **Always use projection** to limit returned fields
   ```javascript
   // ❌ Bad: Returns all fields
   db.users.find({ age: 30 });

   // ✅ Good: Returns only needed fields
   db.users.find({ age: 30 }, { name: 1, email: 1 });
   ```

2. **Use appropriate operators**
   ```javascript
   // ❌ Bad: Overwrites entire document
   db.users.update({ _id: id }, { age: 31 });

   // ✅ Good: Updates specific field
   db.users.updateOne({ _id: id }, { $set: { age: 31 } });
   ```

3. **Use indexes** for frequently queried fields
   ```javascript
   db.users.createIndex({ email: 1 });
   db.users.createIndex({ age: 1, city: 1 });
   ```

4. **Handle errors properly**
   ```javascript
   try {
     const result = db.users.insertOne({ name: "John" });
     if (result.acknowledged) {
       console.log("Inserted:", result.insertedId);
     }
   } catch (error) {
     console.error("Error:", error.message);
   }
   ```

5. **Use bulk operations** for multiple operations
   ```javascript
   // ✅ More efficient
   db.users.bulkWrite(operations);

   // ❌ Less efficient
   operations.forEach(op => db.users.insertOne(op));
   ```

## Key Takeaways

1. Use `insertOne`/`insertMany` for creating documents
2. Use `find`/`findOne` with projections for efficient reads
3. Use `updateOne`/`updateMany` with update operators
4. Upsert combines insert and update operations
5. Use bulk operations for better performance
6. Always add error handling
7. Use indexes for frequently queried fields
8. Projection reduces network transfer and improves performance

## Practice Problems

1. Write a query to find users aged 25-35 in New York
2. Update all inactive users who haven't logged in for 6 months
3. Find users with at least 3 hobbies
4. Increment login count for a user and update lastLogin
5. Remove a specific hobby from all users who have it

---

**Next Topic**: [Aggregation Pipeline](./aggregation-pipeline.md)
