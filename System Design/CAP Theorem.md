# CAP Theorem

You can only have 2 of the 3:

1. Consistency: All nodes need to see the same user data at the same time.
2. Availability: Every request gets a response, either successful or not.
3. Partition tolerance: It means the system works despite network failures between nodes.

But in a distributed system, you basically guarantee partition tolerance because network failures are unavoidable in a distributed system. So the only real choice you have when designing a distributed system is choosing whether to prioritize consistency or availability.

![Network Failure between two nodes](<Media/Network failure between nodes.png>)

## Prioritizing Strong Consistency  

- Stop serving the data
- **Examples scenario** where this might be desirable: Ticket booking platform, inventory system or financial systems where a user is buying a commodity that is limited in quantity. You don't want to double-sell it to a user buying it on one node and another user buying it on another node.
- How does it influence the design of the distributed system:
  - Impelement distributed transactions: For example, if you had a cache and a database, you need to ensure that those two things remain consistent, so write happens to one, write happens to the other.
  - Limit database to a single node
  - Discuss consensus protocols: If you don't want to limit database to a single node and want to spread it across multiple database nodes, then you need to explain how those nodes agree on the order of updates.
  - Accept higher latency
  - Example Tools: PostgreSQL, Tradition RDBMS, Spanner, NoSQL with strong consistency mode like DynamoDB

## Prioritizing strong Availability

- Risk wrong data
- **Example scenario** where this might be desirable: A social media app, a Yelp-like business review service, or a streaming service like Netflix where it's okay if your user uploads a post on one node but it's not available to be viewed on the other node for some time
- How does it influence the design of the distributed system:
  - You could be using multiple replicas.
  - CDC (Change Data Capture) and Eventual Consistency is OK.
  - ExampleTools: DynamoDB (in MultiAZ mode) or Cassandra

## One system, Multiple requirements/Priorities

Different parts of the system can have different requirements, just like different microservices have different business capabilities to deal with. So these different parts could have different priorities for consistency or availability.

Ticket Master Example:

- availability for CRUD on events (Searching, viewing, or updating events. Description. )
- consistency for booking tickets.

Tinder example:

- availability for viewing profile data
- consistency for matching

## Different types of consistency

There are different levels of consistency. When we prioritize availability over consistency, we're not saying that the system is not going to be consistent; in the least, the system is going to be eventually consistent. Our priority determines the level of consistency that we're going to propagate in the system.

1. Strong consistency: all the reads reflect the most recent write.
2. Causal consistency: related events appear in order. For example, a reply to a comment has to appear after the original comment, so they need to be updated accordingly.
3. Read your writes consistency: user sees their own updates.
4. Eventual consistency: updates will propagate eventually.
