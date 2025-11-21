# MongoDB Aggregation Pipeline

## Table of Contents
- [What is Aggregation?](#what-is-aggregation)
- [Common Stages](#common-stages)
- [Aggregation Operators](#aggregation-operators)
- [Real-World Examples](#real-world-examples)
- [Common Interview Questions](#common-interview-questions)

## What is Aggregation?

The aggregation pipeline is a framework for data aggregation, transforming documents as they pass through a multi-stage pipeline.

```javascript
db.collection.aggregate([
  { stage1 },
  { stage2 },
  { stage3 }
]);
```

Each stage transforms documents and passes them to the next stage.

## Common Stages

### $match

Filters documents (like `find()`).

```javascript
// Find orders with status "completed"
db.orders.aggregate([
  { $match: { status: "completed" } }
]);

// Match multiple conditions
db.orders.aggregate([
  {
    $match: {
      status: "completed",
      total: { $gte: 100 }
    }
  }
]);

// Best practice: Use $match early to reduce documents processed
db.orders.aggregate([
  { $match: { status: "completed" } },  // Filter first
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } }
]);
```

### $project

Reshapes documents, includes/excludes fields.

```javascript
// Select specific fields
db.users.aggregate([
  {
    $project: {
      name: 1,
      email: 1,
      _id: 0
    }
  }
]);

// Computed fields
db.orders.aggregate([
  {
    $project: {
      orderNumber: 1,
      total: 1,
      tax: { $multiply: ["$total", 0.08] },
      grandTotal: { $multiply: ["$total", 1.08] }
    }
  }
]);

// Rename fields
db.users.aggregate([
  {
    $project: {
      fullName: "$name",
      emailAddress: "$email"
    }
  }
]);

// String operations
db.users.aggregate([
  {
    $project: {
      name: 1,
      initials: {
        $concat: [
          { $substr: ["$firstName", 0, 1] },
          { $substr: ["$lastName", 0, 1] }
        ]
      }
    }
  }
]);
```

### $group

Groups documents by expression and calculates aggregates.

```javascript
// Count documents by field
db.orders.aggregate([
  {
    $group: {
      _id: "$status",
      count: { $sum: 1 }
    }
  }
]);

// Sum by group
db.orders.aggregate([
  {
    $group: {
      _id: "$customerId",
      totalSpent: { $sum: "$amount" },
      orderCount: { $sum: 1 }
    }
  }
]);

// Average by group
db.orders.aggregate([
  {
    $group: {
      _id: "$category",
      avgPrice: { $avg: "$price" }
    }
  }
]);

// Multiple aggregations
db.orders.aggregate([
  {
    $group: {
      _id: "$customerId",
      totalSpent: { $sum: "$amount" },
      avgOrderValue: { $avg: "$amount" },
      minOrder: { $min: "$amount" },
      maxOrder: { $max: "$amount" },
      orderCount: { $sum: 1 }
    }
  }
]);

// Collect values into array
db.orders.aggregate([
  {
    $group: {
      _id: "$customerId",
      orders: { $push: "$orderNumber" },
      orderDetails: { $push: { orderNumber: "$orderNumber", amount: "$amount" } }
    }
  }
]);

// Add to set (unique values only)
db.orders.aggregate([
  {
    $group: {
      _id: "$customerId",
      uniqueProducts: { $addToSet: "$productId" }
    }
  }
]);

// Group all documents
db.orders.aggregate([
  {
    $group: {
      _id: null,  // Group all
      totalRevenue: { $sum: "$amount" },
      totalOrders: { $sum: 1 }
    }
  }
]);
```

### $sort

Sorts documents.

```javascript
// Sort ascending
db.orders.aggregate([
  { $sort: { amount: 1 } }
]);

// Sort descending
db.orders.aggregate([
  { $sort: { amount: -1 } }
]);

// Multiple sort fields
db.orders.aggregate([
  { $sort: { status: 1, amount: -1 } }
]);

// Sort after grouping
db.orders.aggregate([
  {
    $group: {
      _id: "$customerId",
      totalSpent: { $sum: "$amount" }
    }
  },
  { $sort: { totalSpent: -1 } }
]);
```

### $limit

Limits number of documents.

```javascript
// Get top 10
db.orders.aggregate([
  { $sort: { amount: -1 } },
  { $limit: 10 }
]);

// Top 5 customers by spending
db.orders.aggregate([
  {
    $group: {
      _id: "$customerId",
      totalSpent: { $sum: "$amount" }
    }
  },
  { $sort: { totalSpent: -1 } },
  { $limit: 5 }
]);
```

### $skip

Skips documents (pagination).

```javascript
// Pagination: Page 2, 10 items per page
db.orders.aggregate([
  { $sort: { createdAt: -1 } },
  { $skip: 10 },
  { $limit: 10 }
]);
```

### $lookup

Performs left outer join with another collection.

```javascript
// Basic lookup
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",        // Collection to join
      localField: "customerId", // Field in orders
      foreignField: "_id",      // Field in customers
      as: "customerInfo"        // Output array field
    }
  }
]);

// Lookup with pipeline
db.orders.aggregate([
  {
    $lookup: {
      from: "products",
      let: { productIds: "$items.productId" },
      pipeline: [
        {
          $match: {
            $expr: { $in: ["$_id", "$$productIds"] }
          }
        },
        {
          $project: { name: 1, price: 1 }
        }
      ],
      as: "productDetails"
    }
  }
]);

// Multiple lookups
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customer"
    }
  },
  {
    $lookup: {
      from: "products",
      localField: "productId",
      foreignField: "_id",
      as: "product"
    }
  }
]);
```

### $unwind

Deconstructs an array field.

```javascript
// Unwind array
db.orders.aggregate([
  {
    $unwind: "$items"  // Each item becomes separate document
  }
]);

// Example:
// Input:  { _id: 1, items: ["A", "B", "C"] }
// Output: { _id: 1, items: "A" }
//         { _id: 1, items: "B" }
//         { _id: 1, items: "C" }

// With options
db.orders.aggregate([
  {
    $unwind: {
      path: "$items",
      includeArrayIndex: "itemIndex",  // Add index
      preserveNullAndEmptyArrays: true // Keep docs with empty arrays
    }
  }
]);

// Unwind and group back
db.orders.aggregate([
  { $unwind: "$items" },
  {
    $group: {
      _id: "$items.productId",
      totalSold: { $sum: "$items.quantity" }
    }
  }
]);
```

### $addFields

Adds new fields to documents.

```javascript
// Add computed field
db.orders.aggregate([
  {
    $addFields: {
      tax: { $multiply: ["$amount", 0.08] },
      grandTotal: { $multiply: ["$amount", 1.08] }
    }
  }
]);

// Add field based on condition
db.users.aggregate([
  {
    $addFields: {
      ageGroup: {
        $cond: {
          if: { $lt: ["$age", 18] },
          then: "minor",
          else: "adult"
        }
      }
    }
  }
]);
```

### $bucket

Categorizes documents into buckets.

```javascript
// Age groups
db.users.aggregate([
  {
    $bucket: {
      groupBy: "$age",
      boundaries: [0, 18, 30, 50, 100],
      default: "Other",
      output: {
        count: { $sum: 1 },
        users: { $push: "$name" }
      }
    }
  }
]);

// Output:
// { _id: 0, count: 5, users: [...] }   // 0-17
// { _id: 18, count: 10, users: [...] } // 18-29
// { _id: 30, count: 15, users: [...] } // 30-49
// { _id: 50, count: 8, users: [...] }  // 50-99
```

### $bucketAuto

Automatically determines bucket boundaries.

```javascript
db.orders.aggregate([
  {
    $bucketAuto: {
      groupBy: "$amount",
      buckets: 5,  // Create 5 buckets
      output: {
        count: { $sum: 1 },
        avgAmount: { $avg: "$amount" }
      }
    }
  }
]);
```

### $facet

Runs multiple pipelines in parallel.

```javascript
db.products.aggregate([
  {
    $facet: {
      // Facet 1: Price ranges
      priceRanges: [
        {
          $bucket: {
            groupBy: "$price",
            boundaries: [0, 50, 100, 500],
            default: "expensive"
          }
        }
      ],
      // Facet 2: Categories
      categoryCounts: [
        {
          $group: {
            _id: "$category",
            count: { $sum: 1 }
          }
        }
      ],
      // Facet 3: Statistics
      stats: [
        {
          $group: {
            _id: null,
            avgPrice: { $avg: "$price" },
            maxPrice: { $max: "$price" },
            minPrice: { $min: "$price" }
          }
        }
      ]
    }
  }
]);
```

### $out

Writes results to a collection.

```javascript
// Create new collection with results
db.orders.aggregate([
  {
    $group: {
      _id: "$customerId",
      totalSpent: { $sum: "$amount" }
    }
  },
  { $out: "customerStats" }  // Replaces entire collection
]);
```

### $merge

Merges results into a collection.

```javascript
db.orders.aggregate([
  {
    $group: {
      _id: "$customerId",
      totalSpent: { $sum: "$amount" }
    }
  },
  {
    $merge: {
      into: "customerStats",
      whenMatched: "merge",      // merge, replace, keepExisting, fail
      whenNotMatched: "insert"
    }
  }
]);
```

## Aggregation Operators

### Arithmetic Operators

```javascript
db.orders.aggregate([
  {
    $project: {
      // $add
      total: { $add: ["$subtotal", "$tax", "$shipping"] },

      // $subtract
      discount: { $subtract: ["$originalPrice", "$salePrice"] },

      // $multiply
      tax: { $multiply: ["$subtotal", 0.08] },

      // $divide
      avgItemPrice: { $divide: ["$total", "$itemCount"] },

      // $mod
      remainder: { $mod: ["$total", 10] },

      // $abs
      absoluteValue: { $abs: "$profit" }
    }
  }
]);
```

### Comparison Operators

```javascript
db.products.aggregate([
  {
    $project: {
      name: 1,
      price: 1,
      // $cmp: -1 if a < b, 0 if equal, 1 if a > b
      compareToTarget: { $cmp: ["$price", 100] },

      // $eq, $ne, $gt, $gte, $lt, $lte
      isExpensive: { $gte: ["$price", 100] },
      inRange: {
        $and: [
          { $gte: ["$price", 50] },
          { $lte: ["$price", 150] }
        ]
      }
    }
  }
]);
```

### String Operators

```javascript
db.users.aggregate([
  {
    $project: {
      // $concat
      fullName: { $concat: ["$firstName", " ", "$lastName"] },

      // $substr
      initials: { $substr: ["$firstName", 0, 1] },

      // $toUpper, $toLower
      upperEmail: { $toUpper: "$email" },
      lowerEmail: { $toLower: "$email" },

      // $strLenCP
      nameLength: { $strLenCP: "$name" },

      // $split
      emailParts: { $split: ["$email", "@"] },

      // $trim
      cleanName: { $trim: { input: "$name" } }
    }
  }
]);
```

### Array Operators

```javascript
db.orders.aggregate([
  {
    $project: {
      // $size
      itemCount: { $size: "$items" },

      // $arrayElemAt
      firstItem: { $arrayElemAt: ["$items", 0] },
      lastItem: { $arrayElemAt: ["$items", -1] },

      // $slice
      firstThreeItems: { $slice: ["$items", 3] },

      // $filter
      expensiveItems: {
        $filter: {
          input: "$items",
          as: "item",
          cond: { $gte: ["$$item.price", 100] }
        }
      },

      // $map
      itemNames: {
        $map: {
          input: "$items",
          as: "item",
          in: "$$item.name"
        }
      },

      // $reduce
      totalAmount: {
        $reduce: {
          input: "$items",
          initialValue: 0,
          in: { $add: ["$$value", "$$this.price"] }
        }
      }
    }
  }
]);
```

### Date Operators

```javascript
db.orders.aggregate([
  {
    $project: {
      // $year, $month, $dayOfMonth, $hour, $minute, $second
      year: { $year: "$createdAt" },
      month: { $month: "$createdAt" },
      day: { $dayOfMonth: "$createdAt" },

      // $dayOfWeek (1=Sunday, 7=Saturday)
      dayOfWeek: { $dayOfWeek: "$createdAt" },

      // $dateToString
      formattedDate: {
        $dateToString: {
          format: "%Y-%m-%d",
          date: "$createdAt"
        }
      },

      // $dateDiff
      daysSinceOrder: {
        $dateDiff: {
          startDate: "$createdAt",
          endDate: "$$NOW",
          unit: "day"
        }
      }
    }
  }
]);
```

### Conditional Operators

```javascript
db.products.aggregate([
  {
    $project: {
      name: 1,
      price: 1,
      // $cond (if-then-else)
      priceCategory: {
        $cond: {
          if: { $gte: ["$price", 100] },
          then: "expensive",
          else: "affordable"
        }
      },

      // $switch (multiple conditions)
      priceRange: {
        $switch: {
          branches: [
            { case: { $lt: ["$price", 50] }, then: "budget" },
            { case: { $lt: ["$price", 100] }, then: "mid-range" },
            { case: { $lt: ["$price", 500] }, then: "premium" }
          ],
          default: "luxury"
        }
      },

      // $ifNull
      displayPrice: { $ifNull: ["$salePrice", "$regularPrice"] }
    }
  }
]);
```

## Real-World Examples

### E-Commerce: Customer Lifetime Value

```javascript
db.orders.aggregate([
  // Stage 1: Match completed orders
  { $match: { status: "completed" } },

  // Stage 2: Group by customer
  {
    $group: {
      _id: "$customerId",
      totalSpent: { $sum: "$amount" },
      orderCount: { $sum: 1 },
      avgOrderValue: { $avg: "$amount" },
      firstOrder: { $min: "$createdAt" },
      lastOrder: { $max: "$createdAt" }
    }
  },

  // Stage 3: Add computed fields
  {
    $addFields: {
      daysSinceFirstOrder: {
        $dateDiff: {
          startDate: "$firstOrder",
          endDate: "$$NOW",
          unit: "day"
        }
      }
    }
  },

  // Stage 4: Categorize customers
  {
    $addFields: {
      customerSegment: {
        $switch: {
          branches: [
            { case: { $gte: ["$totalSpent", 10000] }, then: "VIP" },
            { case: { $gte: ["$totalSpent", 5000] }, then: "Gold" },
            { case: { $gte: ["$totalSpent", 1000] }, then: "Silver" }
          ],
          default: "Bronze"
        }
      }
    }
  },

  // Stage 5: Sort by total spent
  { $sort: { totalSpent: -1 } },

  // Stage 6: Lookup customer details
  {
    $lookup: {
      from: "customers",
      localField: "_id",
      foreignField: "_id",
      as: "customerInfo"
    }
  },

  // Stage 7: Unwind customer info
  { $unwind: "$customerInfo" },

  // Stage 8: Project final fields
  {
    $project: {
      customerId: "$_id",
      name: "$customerInfo.name",
      email: "$customerInfo.email",
      totalSpent: 1,
      orderCount: 1,
      avgOrderValue: { $round: ["$avgOrderValue", 2] },
      customerSegment: 1,
      daysSinceFirstOrder: 1
    }
  }
]);
```

### Product Analytics: Best Sellers by Category

```javascript
db.orders.aggregate([
  // Unwind items
  { $unwind: "$items" },

  // Lookup product details
  {
    $lookup: {
      from: "products",
      localField: "items.productId",
      foreignField: "_id",
      as: "product"
    }
  },

  { $unwind: "$product" },

  // Group by category and product
  {
    $group: {
      _id: {
        category: "$product.category",
        productId: "$product._id",
        productName: "$product.name"
      },
      unitsSold: { $sum: "$items.quantity" },
      revenue: { $sum: { $multiply: ["$items.quantity", "$items.price"] } }
    }
  },

  // Sort within category
  { $sort: { "_id.category": 1, revenue: -1 } },

  // Group by category to get top products
  {
    $group: {
      _id: "$_id.category",
      topProducts: {
        $push: {
          name: "$_id.productName",
          unitsSold: "$unitsSold",
          revenue: "$revenue"
        }
      },
      totalRevenue: { $sum: "$revenue" }
    }
  },

  // Get top 5 products per category
  {
    $project: {
      category: "$_id",
      topProducts: { $slice: ["$topProducts", 5] },
      totalRevenue: 1
    }
  },

  // Sort by total revenue
  { $sort: { totalRevenue: -1 } }
]);
```

### Time Series: Daily Sales Report

```javascript
db.orders.aggregate([
  // Match date range
  {
    $match: {
      createdAt: {
        $gte: new Date("2024-01-01"),
        $lt: new Date("2024-02-01")
      }
    }
  },

  // Group by date
  {
    $group: {
      _id: {
        year: { $year: "$createdAt" },
        month: { $month: "$createdAt" },
        day: { $dayOfMonth: "$createdAt" }
      },
      totalSales: { $sum: "$amount" },
      orderCount: { $sum: 1 },
      uniqueCustomers: { $addToSet: "$customerId" }
    }
  },

  // Add computed fields
  {
    $addFields: {
      avgOrderValue: { $divide: ["$totalSales", "$orderCount"] },
      uniqueCustomerCount: { $size: "$uniqueCustomers" }
    }
  },

  // Sort by date
  {
    $sort: {
      "_id.year": 1,
      "_id.month": 1,
      "_id.day": 1
    }
  },

  // Format output
  {
    $project: {
      _id: 0,
      date: {
        $dateFromParts: {
          year: "$_id.year",
          month: "$_id.month",
          day: "$_id.day"
        }
      },
      totalSales: { $round: ["$totalSales", 2] },
      orderCount: 1,
      avgOrderValue: { $round: ["$avgOrderValue", 2] },
      uniqueCustomerCount: 1
    }
  }
]);
```

## Common Interview Questions

### Q1: Difference between $match and find()?

**Answer**: `$match` is an aggregation stage that filters documents within a pipeline, while `find()` is a query method. `$match` uses the same query syntax as `find()` but can be combined with other aggregation stages.

### Q2: When to use $match in the pipeline?

**Answer**: Use `$match` as early as possible to reduce the number of documents processed by subsequent stages. This improves performance.

```javascript
// ✅ Good: $match first
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } }
]);

// ❌ Bad: $match after expensive operations
db.orders.aggregate([
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } },
  { $match: { total: { $gte: 1000 } } }
]);
```

### Q3: Explain $lookup vs $unwind

**Answer**:
- `$lookup`: Performs a join with another collection (like SQL JOIN)
- `$unwind`: Deconstructs an array field into multiple documents

They're often used together to join collections with array fields.

### Q4: How to optimize aggregation pipelines?

**Answers**:
1. Use `$match` early to filter documents
2. Use `$project` to reduce field size early
3. Use indexes for `$match` and `$sort`
4. Avoid `$lookup` when possible (denormalize if needed)
5. Use `$limit` to reduce result size
6. Consider using `allowDiskUse: true` for large datasets

### Q5: What is $facet used for?

**Answer**: `$facet` runs multiple aggregation pipelines in parallel on the same set of documents. Useful for creating dashboards or generating multiple views of data in a single query.

## Key Takeaways

1. Aggregation pipeline processes documents in stages
2. Each stage transforms and passes documents to the next
3. Use `$match` early for better performance
4. `$group` is powerful for analytics and reports
5. `$lookup` enables joins between collections
6. `$unwind` deconstructs arrays for further processing
7. Operators enable complex transformations
8. Multiple pipelines can run in parallel with `$facet`

## Practice Problems

1. Find top 10 customers by total spending
2. Calculate average order value by month
3. Find products that are never ordered
4. Generate sales report by category and subcategory
5. Find customers who haven't ordered in the last 90 days
6. Calculate conversion funnel metrics
7. Generate cohort analysis for user retention
8. Find products frequently bought together

---

**Next Topic**: [Indexing](./indexing.md)
