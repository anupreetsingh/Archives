Refer the complete Hello Interview [Excalidraw](<https://app.excalidraw.com/l/56zGeHiLyKZ/7AUE0BlFxMd>) for images

# Caching

Cache is a temporary storage that keeps recently used data handy so you can get it much faster next time.

Disk Read Time:
    HDD = 10 millisecond = 10^-2 sec
    SSD = 100 microsecond = 10^-4 sec
Modern data centers use a combination of HDD and SSD for persistant storage so the average response time from disk is 1 millisecond.

Memory Read Time:
    RAM = 100 nanosecond = 10^ -7

Which means reading from memory is 10^4 times faster than disk. So, if we can cache some frequently accessed data temporarily in RAM, it will be much better.

**When to Bring up caching in a system:**

- Read heavy workload
- Expensive queries
- High database CPU usage
- Latency Requirements

**How to Introduce Caching:**

- Identify the bottleneck
- Decide what to cache(selecting the keys in your cache)
- Choose your cache architecture
- Set an eviction policy
- Address the downsides

## Caching Techniques

These are various ways in which data is cached:

### External Caching

![External Caching](<Media/Caching/External Caching.png>)

Cache is stored as external service separate from from your backend application, database, etc.

Example: Redis caching database query results.

### In-process Caching

![In process Caching](<Media/Caching/In-Process caching.png>)

Your cache shares resources with the application servers.

It is faster because you don't need an expensive network call to access your cache.

Example: Python dictionary caches computed results.

### CDN

![CDN](<Media/Caching/CDN.png>)

CDN are geographically distributed network of servers that cache static content(like images, videos, HTML pages, javascript pages) near the users.

The intent here is to optimize for *network latency*. If we don't have it then our requests could take 300ms of round trip time in the worst case, which is a much bigger concern than the disk read time of 1ms.

Requests hit CDN and in case of cache miss, CDN hits the origin server(in USA here) and stores it for future requests.

Example: Amazon Cloudfront caching images, videos, etc.

### Client-side caching

![Client Side Caching](<Media/Caching/Client Side caching.png>)

This is when data is stored directly on the users device(Browser or mobile apps).

Pro is that it's super fast since there is no network call, Con is that data can go stale and there could be validation issues.

Example: Browser caches CSS, JS, images.

## Cache Architectures

### Cache - Aside

![Cache Aside](<Media/Caching/Cache Aside.png>)

Flow:

- Read from the cache.
- In case you miss, Application reads from the DB.
- Application updates the cache
- Application returns the value

#### Consistency Strategy: Invalidate on Write

When data is written to the database, the application deletes the corresponding cache entry instead of updating it. The next read misses the cache, fetches the latest value from the database, and repopulates the cache. This keeps writes simple but makes the first read after each write slower.

### Read - Through

![Read Through](<Media/Caching/Read Through.png>)

Similar to cache-aside but in this case the cache handles the database lookup instead of the application.

Flow:

- Read from the cache.
- In case you miss, cache reads from the DB.
- Cache stores the retrieved resul in itself.
- Cache returns the value to the application.

### Write - Through

![Write through](<Media/Caching/Write through.png>)

Application writes directly to the cache first. Then the cache synchronously writes to DB. Write is considered complete when both steps happen.

You need libraries like *SpringCache* or *Hazelcache* to implement the sync update to DB.

Hard to achieve this perfect consistency that this scenario requires so less commonly used. You would only use it when your reads must always return the fresh data and your system can tolerate slightly slower writes.

### Write - Behind

![Write Behind](<Media/Caching/Write behind.png>)

Instead of updating the DB synchronously, the cache flushes those updates to the database usually in batches. Write is considered complete when it happens on the cache.

Naturally, this makes the writes much faster than write-through. But the risk is that, in case the cache was to go down before the writes were processed to the DB, we would have data loss.

Used when high write throughput is more important than immediate consistency. Example: Analytics or metrics pipelines where some data loss might be acceptable.

## Cache Eviction Policies

- **Least Recently Used (LRU):** Evicts items that have not been used recently. The most common and balanced default.

- **Least Frequently Used (LFU):** Evicts items used least often, even if accessed recently. Good for highly skewed access patterns meaning some items are used way more oftenly than others.

        Example:
        Product A → 10,000 accesses
        Product B → 8,000 accesses
        Product C → 5 accesses
        Product D → 2 accesses

        Suppose A hasn't been accessed for 10 minutes, while D was accessed 10 seconds ago. LFU would evict D because it has been accessed much less historically

- **First-In, First-Out (FIFO):** Evicts the oldest item first. Simple, but rarely the right choice.

- **Time to Live (TTL):** Each item expires after a set time (for example, five minutes). Great for data that can become stale, such as API responses.

## Common Issues

These are some of the common issues and complexities that come when a cache has been used in a system.

### Cache Stampede (Thundering Herd)

![Cache Stampede](<Media/Caching/Cache Stampede.png>)

Ways to handle it:

1. Request coallescing / Single Flight
2. Cache Warming: Proactively refresh the keys in the cache so that it remain

### Cache Consistency

![Cache Consistency](<Media/Caching/Cache Consistency.png>)

A common issue in the widely used cache-aside and read-through cache architectures. But it is Less common when a write-through or write-behind has been implemented because the cache would have the updated value in the database since the database queries are flowing through it.

Ways to handle it:

1. Invalidate on write
2. Use short TTLs.
3. Be okay with stale data for a while(User profile updating after 2 mins)

### Hot Keys

![Hot Keys](<Media/Caching/Hot Keys.png>)

One hot key can overload the cache.

Ways to handle it:

1. Shard the cache(make additional instances of caches) and replicate the hot key on multiple shards.
2. Use in-process caching to add a fallback cache for the hot keys so your requests don't even need to hit the external cache.
