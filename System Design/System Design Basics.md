
## System Design Interview

System design interview evaluates your ability to architect complex scalable systems that solve real world problems.

![Design Problem Types](<Media/Basics/Design Types.png>)

Product Design: Involves user facing systems. Eg: Ticketmaster, Uber, etc.

Infrastructure Design: Involves non user facing systems. Eg: Rate Limiter, Message Queue, Data Processing pipeline like an ad-click aggregator.

## Framework

Follow this framework during an interview

![Approach Framework](<Media/Basics/Approach Framework.png>)

## Evaluation Points

System design interviews commonly evaluate four areas:

1. **Problem Solving:** Identify and prioritize the core challenges. Example: In Ticketmaster, Focusing on booking flow is more important than authentication.
2. **Solution Design:** Create scalable architectures with balanced trade-offs.
3. **Technical Excellence:** Demonstrate deep technical knowledge and expertise.
4. **Communication:** Clearly explain complex concepts to stakeholders.

## Fundamentals

The fundamentals to think about that are going to get you 90% of the way there:

1. Storage: Discuss Various DBMS(relational, document, key-value, etc) and their appropriate use cases. Also consider ACID vs BASE.

2. Scalability: Talking about vertical and horizontal scaling methods for compute and scaling storage using partitioning, sharding, replication, etc.

3. Networking: In system design, the three most important layers are the application, transport, and network layers.

   - **Application Layer (Layer 7):** HTTP/HTTPS, REST vs. GraphQL vs. gRPC, RESTful semantics, DNS resolution, and WebSockets vs. Server-Sent Events (SSE).
   - **Transport Layer (Layer 4):** TCP, UDP, ports, and the request-response lifecycle.
   - **Network Layer (Layer 3):** IP addressing and routing, load balancing, firewalls, and network access-control lists (ACLs).

4. Latency, Throughput, and Performance: Be able to recall approximate latency number for common operations like reads, write, etc from different pieces of hardware. Along with thinking about caching, CDNs.

5. Fault Tolerance and Redundancy: Failure Modes, failure detection mechanism, replication strategy, etc.

6. CAP Theorem: Since partition tolerance is guaranted, discuss and make sure your system upholds whatever other property you prioritize.

![CAP Consideration](<Media/CAP consideration.png>)

## Working Components

A basic example of Working components in a modern system architecture:

![Working Components](<Media/Basics/Working Components.png>)

## Problem Progression

[Hello Interview](https://www.hellointerview.com/learn/) recommends the following problem progression:

1. **Design a URL Shortener (Bitly):** Tests your understanding of hashing, databases, and caching.
2. **Design Dropbox:** Tests file storage, synchronization, and metadata management.
3. **Design Ticketmaster:** Tests concurrency, race conditions, and transactional integrity.
4. **Design a News Feed:** Tests content delivery, personalization, and real-time updates.
5. **Design WhatsApp:** Tests real-time communication, presence detection, and message delivery.
6. **Design LeetCode:** Tests code-execution environments, compute scaling, and security.
7. **Design Uber:** Tests geospatial indexing, matching algorithms, and real-time updates.
8. **Design a Web Crawler:** Tests distributed systems, scheduling, and politeness policies.
9. **Design an Ad Click Aggregator:** Tests high-throughput event processing and analytics.
10. **Design Facebook's Post Search:** Tests indexing, ranking, and search optimization.
