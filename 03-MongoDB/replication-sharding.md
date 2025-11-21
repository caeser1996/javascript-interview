# MongoDB Replication and Sharding

## Table of Contents
- [Replication](#replication)
- [Sharding](#sharding)
- [Interview Questions](#interview-questions)

## Replication

### What is Replication?

Replication is the process of synchronizing data across multiple servers for redundancy and high availability.

**Replica Set**: A group of MongoDB servers that maintain the same data.

### Replica Set Architecture

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   PRIMARY   │─────→│  SECONDARY  │      │  SECONDARY  │
│  (Read/Write│      │  (Read Only)│      │  (Read Only)│
└─────────────┘      └─────────────┘      └─────────────┘
       │                     ↑                     ↑
       └─────────────────────┴─────────────────────┘
              Replication (async)
```

**Components**:
- **Primary**: Receives all write operations
- **Secondary**: Replicates data from primary
- **Arbiter**: Votes in elections (doesn't hold data)

### Key Features

1. **Automatic Failover**: If primary fails, secondary is elected as new primary
2. **Data Redundancy**: Multiple copies of data
3. **Read Scalability**: Read from secondaries (with read preference)
4. **Disaster Recovery**: Data backed up across servers

### Replica Set Configuration

```javascript
// Initiate replica set
rs.initiate({
  _id: "myReplicaSet",
  members: [
    { _id: 0, host: "mongodb0.example.com:27017" },
    { _id: 1, host: "mongodb1.example.com:27017" },
    { _id: 2, host: "mongodb2.example.com:27017" }
  ]
});

// Check replica set status
rs.status();

// Check replica set configuration
rs.conf();

// Add member
rs.add("mongodb3.example.com:27017");

// Remove member
rs.remove("mongodb3.example.com:27017");

// Step down primary (force election)
rs.stepDown();
```

### Read Preference

Control where reads are directed.

```javascript
// Primary (default): Read from primary only
db.collection.find().readPref("primary");

// PrimaryPreferred: Read from primary, secondary if unavailable
db.collection.find().readPref("primaryPreferred");

// Secondary: Read from secondary only
db.collection.find().readPref("secondary");

// SecondaryPreferred: Read from secondary, primary if unavailable
db.collection.find().readPref("secondaryPreferred");

// Nearest: Read from nearest member (lowest latency)
db.collection.find().readPref("nearest");
```

### Write Concern

Control acknowledgment of write operations.

```javascript
// Default: Acknowledge from primary
db.collection.insertOne(
  { name: "John" },
  { writeConcern: { w: 1 } }
);

// Majority: Acknowledge from majority of replica set
db.collection.insertOne(
  { name: "John" },
  { writeConcern: { w: "majority" } }
);

// Specific number: Acknowledge from N members
db.collection.insertOne(
  { name: "John" },
  { writeConcern: { w: 2, j: true, wtimeout: 5000 } }
);
// w: number of acknowledgments
// j: true = wait for journal
// wtimeout: max time to wait
```

### Read Concern

Control consistency of read operations.

```javascript
// Local: Returns data from primary (default)
db.collection.find().readConcern("local");

// Available: Returns data immediately (may be stale)
db.collection.find().readConcern("available");

// Majority: Returns data acknowledged by majority
db.collection.find().readConcern("majority");

// Linearizable: Returns data reflecting all successful writes
db.collection.find().readConcern("linearizable");
```

## Sharding

### What is Sharding?

Sharding distributes data across multiple machines for horizontal scaling.

**Shard**: A replica set that stores a subset of data
**Mongos**: Query router that directs operations to appropriate shards
**Config Servers**: Store metadata and configuration

### Sharding Architecture

```
┌──────────────────────────────────────┐
│         APPLICATION                  │
└────────────┬─────────────────────────┘
             │
      ┌──────▼──────┐
      │   MONGOS    │  (Query Router)
      │  (Router)   │
      └──────┬──────┘
             │
   ┌─────────┼─────────┐
   │         │         │
┌──▼───┐  ┌──▼───┐  ┌──▼───┐
│SHARD1│  │SHARD2│  │SHARD3│
│(RS)  │  │(RS)  │  │(RS)  │
└──────┘  └──────┘  └──────┘
   │         │         │
   └─────────┼─────────┘
             │
      ┌──────▼──────┐
      │Config Servers│
      │    (RS)     │
      └─────────────┘
```

### Shard Key

The field used to distribute documents across shards.

**Good Shard Key**:
- High cardinality (many unique values)
- Good write distribution
- Query isolation (queries target specific shards)

**Bad Shard Key**:
- Low cardinality (few unique values)
- Monotonically increasing (e.g., timestamp, _id)
- All writes go to one shard

### Sharding Strategies

#### 1. Range-Based Sharding

Distributes documents based on shard key value ranges.

```javascript
// Shard by age ranges
// Shard 1: age 0-30
// Shard 2: age 31-60
// Shard 3: age 61+

sh.shardCollection("mydb.users", { age: 1 });
```

**Pros**: Range queries efficient
**Cons**: Uneven distribution possible

#### 2. Hashed Sharding

Distributes documents based on hash of shard key.

```javascript
// Hash shard key for even distribution
sh.shardCollection("mydb.users", { userId: "hashed" });
```

**Pros**: Even distribution
**Cons**: Range queries less efficient

#### 3. Zone/Tag Sharding

Assign specific data ranges to specific shards.

```javascript
// Define zones based on geography
sh.addShardTag("shard0", "US");
sh.addShardTag("shard1", "EU");
sh.addShardTag("shard2", "ASIA");

sh.addTagRange(
  "mydb.users",
  { country: "US", userId: MinKey },
  { country: "US", userId: MaxKey },
  "US"
);
```

### Sharding Commands

```javascript
// Enable sharding on database
sh.enableSharding("mydb");

// Shard collection (range-based)
sh.shardCollection("mydb.users", { userId: 1 });

// Shard collection (hashed)
sh.shardCollection("mydb.orders", { orderId: "hashed" });

// Compound shard key
sh.shardCollection("mydb.logs", { userId: 1, timestamp: 1 });

// Check sharding status
sh.status();

// View chunk distribution
db.getSiblingDB("config").chunks.find({ ns: "mydb.users" });

// Move chunk to different shard
sh.moveChunk("mydb.users", { userId: 100 }, "shard0001");
```

### Chunk Management

Sharded data is divided into chunks (default 64MB).

```javascript
// View chunks
use config
db.chunks.find({ ns: "mydb.users" }).sort({ min: 1 });

// Manual split
sh.splitAt("mydb.users", { userId: 1000 });

// Split find
sh.splitFind("mydb.users", { userId: 1500 });

// Move chunk
sh.moveChunk("mydb.users", { userId: 500 }, "shard0002");
```

### Balancer

Automatically distributes chunks evenly across shards.

```javascript
// Check balancer status
sh.getBalancerState();

// Start balancer
sh.startBalancer();

// Stop balancer
sh.stopBalancer();

// Check if balancer is running
sh.isBalancerRunning();

// Set balancer window (schedule)
use config
db.settings.update(
  { _id: "balancer" },
  {
    $set: {
      activeWindow: {
        start: "01:00",
        stop: "05:00"
      }
    }
  },
  { upsert: true }
);
```

## Interview Questions

### Q1: What is a Replica Set?

**Answer**: A replica set is a group of MongoDB servers that maintain the same data. It provides:
- Redundancy (data copies)
- High availability (automatic failover)
- Read scalability (read from secondaries)

Minimum: 3 members (or 2 + 1 arbiter)

### Q2: Primary vs Secondary vs Arbiter?

**Answer**:
- **Primary**: Accepts writes, serves reads (by default)
- **Secondary**: Replicates from primary, can serve reads
- **Arbiter**: Votes in elections, doesn't store data

### Q3: What happens when Primary fails?

**Answer**:
1. Secondaries detect primary is down (heartbeat)
2. Election is triggered
3. Secondary with most recent data wins
4. New primary is elected
5. Writes can resume

Automatic failover typically takes 10-30 seconds.

### Q4: What is Write Concern?

**Answer**: Write concern is the level of acknowledgment requested from MongoDB for write operations.

- `w: 1`: Acknowledge from primary only (fast, less durable)
- `w: "majority"`: Acknowledge from majority (slower, more durable)
- `j: true`: Wait for journal write (durable)

### Q5: What is Read Preference?

**Answer**: Read preference controls where reads are directed:
- `primary`: Read from primary (consistent)
- `secondary`: Read from secondary (may be stale)
- `primaryPreferred`: Primary if available, else secondary
- `secondaryPreferred`: Secondary if available, else primary
- `nearest`: Lowest latency member

### Q6: When to use Sharding?

**Answer**: Use sharding when:
- Data size exceeds single server capacity (>1-2 TB)
- Write throughput exceeds single server capacity
- Active working set exceeds RAM
- Need geographic data distribution

### Q7: What makes a good Shard Key?

**Answer**: A good shard key has:
1. **High Cardinality**: Many unique values
2. **Even Distribution**: Writes distributed across shards
3. **Query Isolation**: Queries target specific shards

```javascript
// ✅ Good: High cardinality, even distribution
{ userId: "hashed" }
{ email: 1 }

// ❌ Bad: Low cardinality
{ status: 1 } // Only "active", "inactive", etc.

// ❌ Bad: Monotonically increasing
{ _id: 1 }    // All writes go to last chunk
{ timestamp: 1 }
```

### Q8: Range vs Hashed Sharding?

**Answer**:

**Range Sharding**:
- Documents grouped by value ranges
- Good for range queries
- Risk of hotspots with monotonic keys

**Hashed Sharding**:
- Documents distributed by hash of key
- Even distribution
- Range queries less efficient

### Q9: What are Jumbo Chunks?

**Answer**: Jumbo chunks are chunks that exceed the max size (64MB) and cannot be moved during balancing. Caused by:
- Insufficient shard key cardinality
- Many documents with same shard key value

Solution: Choose better shard key or manually split.

### Q10: Difference between Replication and Sharding?

**Answer**:

**Replication**:
- Purpose: High availability, redundancy
- Vertical scaling (same data, multiple copies)
- All members have complete dataset
- Read scaling (limited)

**Sharding**:
- Purpose: Horizontal scaling, performance
- Partition data across servers
- Each shard has subset of data
- Write and read scaling

Often used together: Sharded cluster with replica sets.

## Best Practices

### Replication

1. **Use at least 3 members** (or 2 + arbiter)
2. **Enable authentication and encryption**
3. **Use majority write concern** for critical data
4. **Monitor replica lag**
5. **Geographically distribute members**
6. **Configure proper oplog size**
7. **Use hidden members for analytics**

### Sharding

1. **Choose shard key carefully** (can't change easily)
2. **Pre-split chunks** before bulk loading
3. **Monitor balancer activity**
4. **Use compound shard keys** for better distribution
5. **Schedule balancer during off-peak hours**
6. **Start with range-based, switch to hashed if needed**
7. **Monitor chunk distribution**
8. **Use zones for geographic distribution**

## Key Takeaways

1. **Replication** provides redundancy and high availability
2. **Sharding** enables horizontal scaling
3. **Write concern** controls durability vs performance
4. **Read preference** controls consistency vs performance
5. **Shard key** is critical for sharding performance
6. **Hashed sharding** provides even distribution
7. **Range sharding** good for range queries
8. Often use both: Sharded cluster with replica sets

## Practice Problems

1. Design replica set for high availability application
2. Choose appropriate write concern for banking system
3. Design shard key for social media posts collection
4. Troubleshoot unbalanced chunk distribution
5. Configure zone sharding for multi-region deployment
6. Optimize read preference for reporting queries
7. Plan migration from single server to sharded cluster

---

**Next Section**: [Coding Problems](../04-Coding-Problems/)
