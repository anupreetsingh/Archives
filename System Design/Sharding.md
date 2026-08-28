Refer the complete Hello Interview [Sharding Excalidraw](<https://app.excalidraw.com/l/56zGeHiLyKZ/1Seg2UzvFHp>) for images

Refer the complete Hello Interview [Consistent Hashing Excalidraw](<https://app.excalidraw.com/l/56zGeHiLyKZ/4UfYVC7olb2>) for images

**Vertical Scaling**

![Vertical DB scaling](<Media/Sharding/Vertical DB scaling.png>)

When the database starts to fill up or suffer from high traffic, up to a certain extent we can vertically scale it by adding bigger compute.

## Sharding

Sharding is the process of splitting your data across multiple machine so that no single DB holds everything.

Each shard is it's own standalone DB meaning each has its own CPU, memory, disk storage and own connection pool. All of them combine to form the whole database for our system.

![Sharding Intro](<Media/Sharding/Sharding intro.png>)

**When to bring sharding in System Design Interview?**
Bring it up in deep dives when talking about scaling.

- Storage (> 20 TB)
- Write Throughput (> 50K writes/sec)
- Read Throughput (> 200k reads/sec)

These numbers are high end limits of modern systems for vertical scaling so **don't** suggest sharding if your systems requirements are especially orders of magnitude below them.

**What do you say?**

- Propose a shard key based on your access patterns
- Choose your distribution strategy
- Call out the trade-offs
- Address how you'll handle growth.

## How to shard(split) your data?

### What to shard by? This involves selecting the shard-key

Choosing a good shard key involves considering a variety of factors:

- You would want it to have **high cardinalit**y(High frequency of unique values) like the `user_id`
- The values of key should naturally spread out so we have **evenly distributed data**.
- It aligns with **common query** patterns and avoids cross shard operations. Ideally, we choose a shard key that lets related records from different tables be placed on the same shard. For example, if both the User and Posts tables are sharded by user_id, then queries involving a user and that user's posts can usually be handled entirely by one shard instead of searching across or combining data from multiple shards. Some cross-shard queries are unavoidable, but a good shard key minimizes them.

Choosing a bad shard key can involve:

- Selecting a boolean flag like `isPremium` for an Ecommerce site as the shard key. It has low cardinality and uneven distribution
- Selecting `creation_date` on Ecommerce website and imagine you split by year on different sharding. Then likely scenario is that shard of year 2026 would be bombarded with queries cause people want to know about recent order but previous shards are pretty much idle.

### How to distribute data?

When deciding how to split data across different shards, there are a couple strategies you can use here:

#### Range based sharding

Example

```text
Shard 1: 0 - 10M
Shard 2: 10- 20M
Shard 3: 20- 30M
```

Pretty simple but some complication could be :

- In the beginning days of the app only shard 1 is under load
- In a parallel scenario only new users use the app for only shard 3 is under load.

#### Hash Based Sharding

![Hash based sharding](<Media/Sharding/Hash based sharding.png>)

This is the industry standard. But some of the complications could be:

- When adding a new shard the MOD value would change(increment/decrement) and so we would need to adjust the exist entries in the previous shards.

The Solution is using **Consistent Hashing**

##### Consistent Hashing

![Consistent Hashing: Adding a shard](<Media/Sharding/Consistent Hashing1.png>)

- We create a Hash Ring(the circle)
- We evenly distribute our shards across the hash ring.
- We hash, find hash value on the ring and travel clockwise to find the corresponding DB.
- If we add or remove a shard on the shard ring the others are reached just the same by moving clockwise.

How do we make sure the data is evenly distributed among remaining nodes when a node is removed?

![Consistent Hashing: Adding a shard](<Media/Sharding/Consistent Hashing2.png>)

We map virtual nodes so that in case a nodes are removed it's traffic goes to the virtual nodes responsible for the range in that segment and not just the next node.

> Redis, Cassandra, DynamoDB and many CDNs uses consistent hashing.

#### Directory Based Sharding

![Directory based sharding](<Media/Sharding/Directory based sharding.png>)

You look at the directory for figuring out which key maps to which shard.
The downside is the added latency of the directory lookup. Another downside is the directory is single point of failure.

## Challenges that come with Sharding

### Hot spots

Even with a good shard key like user_id. The shard that a celebrity like Messi lands on is going to be order of magnitude larger in its write throughput than the other shards.

Solutions:

- A solution would be using a **compounded shard key** like hash(user_id + createdAt)
- Using a separate dedicated celebrity shard.

### Cross-shard operations

![cross shard operation](<Media/Sharding/Cross shard operations.png>)

Solutions:

- Using a cache for a cross-shard operations.
- Denormalization of data. Normalization means removing duplicates and denormalization here means creating duplicates.

    Example scenario: Say there is some query where you are reading from both shard 1 and shard 2. You copy data onto shard 1 so you only have to read from one shard. But remember, you have to update the data on both shards in case of writes.

### Maintaining Consistency

![Maintaining Consistency](<Media/Sharding/Maintaining Consistency.png>)

Solutions:

- Two Phased Commit(2PC): There is a central coordinator that asks all shards if they are ready for a transaction. In practice it's not as robust because shards could still go down midway transaction.
- Saga Pattern: Use a sequence of smaller operations, where each action has a compensating action that can be run in case things fail.
