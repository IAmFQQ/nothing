# Java, OpenSearch/Elasticsearch & Distributed Systems: Basic Interview Questions

---

## Part 1: Java Fundamentals

**1. What's the difference between `==` and `.equals()`?**
- `==` compares references (memory addresses). For primitives it compares values.
- `equals()` compares content. Classes like `String` and `Integer` override it.
- *Follow-up:* If you override `equals()`, what else should you override, and why?
  → `hashCode()`. Otherwise `HashMap` and `HashSet` won't work correctly.

**2. Why is `String` immutable in Java?**
- Security: strings are used for class names, URLs, and credentials.
- Thread safety, hashCode caching, and the String pool.
- *Follow-up:* What's the difference between `String`, `StringBuilder`, and `StringBuffer`?
  → `StringBuilder` is mutable and not thread-safe. `StringBuffer` is mutable and thread-safe (its methods are synchronized), so it's slower.

**3. How does `HashMap` work internally?**
- An array of buckets, each holding a linked list. Since Java 8, a bucket's list becomes a red-black tree once it holds more than 8 entries.
- `hashCode()` locates the bucket, and `equals()` resolves collisions.
- Default capacity is 16 and the load factor is 0.75. When the threshold is exceeded, the table resizes to 2x.
- *Follow-up:* Is `HashMap` thread-safe? What would you use instead?
  → No. Use `ConcurrentHashMap`.

**4. What's the difference between `ArrayList` and `LinkedList`?**
- `ArrayList` uses a dynamic array: random access is O(1), and inserting or deleting in the middle is O(n).
- `LinkedList` is a doubly linked list: random access is O(n), and inserting or deleting at a known node is O(1).
- In practice, `ArrayList` is used most of the time.

**5. What's the difference between checked and unchecked exceptions?**
- Checked exceptions must be handled at compile time, e.g. `IOException`.
- Unchecked exceptions are subclasses of `RuntimeException`, e.g. `NullPointerException` and `IllegalArgumentException`.
- An `Error` (e.g. `OutOfMemoryError`) should generally not be caught.
- *Follow-up:* What does `try-with-resources` do?
  → It automatically closes resources that implement `AutoCloseable`.

**6. Explain `final`, `finally`, and `finalize`.**
- `final`: a variable can't be reassigned, a method can't be overridden, and a class can't be extended.
- `finally`: the block always executes, whether or not an exception is thrown.
- `finalize`: a deprecated GC callback that shouldn't be used.

**7. What's the difference between an abstract class and an interface?**
- A class can extend only one abstract class but can implement multiple interfaces.
- Abstract classes can have state (fields) and constructors.
- Since Java 8, interfaces can have `default` and `static` methods.
- Rule of thumb: an abstract class expresses "is-a", and an interface expresses "can-do".

**8. How do you create a thread in Java? What's a thread pool, and why use one?**
- Extend `Thread`, or implement `Runnable` or `Callable`. Using an `ExecutorService` is preferred.
- A thread pool reuses threads, limits concurrency, and avoids the cost of creating threads.
- *Follow-up:* What are `corePoolSize`, `maxPoolSize`, and the work queue in `ThreadPoolExecutor`?

**9. What does `synchronized` do? How is it different from `volatile`?**
- `synchronized` provides both mutual exclusion and visibility.
- `volatile` provides visibility and prevents instruction reordering, but it does not guarantee atomicity. A `count++` on a `volatile` field is still unsafe.
- *Follow-up:* How would you make a counter thread-safe?
  → Use `AtomicInteger` or `synchronized`.

**10. Briefly explain the JVM memory areas and garbage collection.**
- The heap holds objects and is split into a young generation and an old generation.
- Each thread has its own stack, which holds local variables and stack frames.
- Metaspace holds class metadata.
- GC uses reachability analysis from GC Roots to decide which objects are alive.
- Common collectors: G1 (the default since Java 9) and ZGC.
- *Follow-up:* What could cause an `OutOfMemoryError`, and how would you troubleshoot it?
  → Analyze a heap dump, check the GC logs, and use tools such as `jmap` and `jstack`.

**11. (Bonus) What Java 8+ features do you use often?**
- Lambdas, the Stream API, `Optional`, `CompletableFuture`, and `java.time`.
- *Follow-up:* What's the difference between `map` and `flatMap` in streams?

---

## Part 2: OpenSearch / Elasticsearch Basics

**1. What is Elasticsearch/OpenSearch, and what is it typically used for?**
- A distributed search and analytics engine built on Apache Lucene.
- Typical uses: full-text search, log analytics (the ELK stack), metrics monitoring, and e-commerce search.

**2. Explain these concepts: cluster, node, index, shard, and replica.**
- An index is roughly like a table in a database. It is split into primary shards, and each primary can have replicas.
- Replicas provide high availability and can serve read requests.
- A primary and its replica are never placed on the same node.
- *Follow-up:* Can you change the number of primary shards after an index is created?
  → Not directly. You need to reindex, or use the split or shrink API.

**3. What is an inverted index?**
- A mapping from each term to the list of documents that contain it. This is why full-text search is fast.
- *Follow-up:* How is this different from a B-tree index in MySQL?

**4. What's the difference between the `text` and `keyword` field types?**
- `text` fields are analyzed (tokenized) and used for full-text search.
- `keyword` fields are not analyzed and are used for exact matching, sorting, and aggregations.
- *Follow-up:* Which one would you use for an email address or a status field?
  → `keyword`.

**5. What is an analyzer?**
- It has three parts: character filters, a tokenizer, and token filters (e.g. lowercasing, stop-word removal, stemming).
- *Follow-up:* Why might a search for "Running" not match "run"?

**6. What's the difference between query context and filter context?**
- Query context calculates a relevance score (`_score`).
- Filter context only answers yes or no. It doesn't score, its results can be cached, and it's faster.
- *Follow-up:* In a `bool` query, what do `must`, `should`, `filter`, and `must_not` mean?

**7. What's the difference between `match` and `term` queries?**
- `match` analyzes the query text first, so it suits `text` fields.
- `term` is an exact match with no analysis, so it suits `keyword` fields.
- A common pitfall: running a `term` query against a `text` field returns no results.

**8. What do green, yellow, and red cluster health mean?**
- Green: all shards are allocated.
- Yellow: all primaries are allocated, but some replicas are not. This is common on single-node clusters.
- Red: some primary shards are unassigned, so some data is unavailable.
- *Follow-up:* How would you troubleshoot a yellow or red cluster?
  → Use `_cluster/allocation/explain` and `_cat/shards`.

**9. Why is Elasticsearch called "near real-time"?**
- Newly written documents become searchable only after a refresh, which happens every 1 second by default.
- The translog ensures durability, and a flush persists Lucene segments to disk.
- *Follow-up:* How can you speed up bulk indexing?
  → Use the bulk API, increase `refresh_interval` (or set it to -1), and temporarily set replicas to 0.

**10. What is a mapping, and what is dynamic mapping?**
- A mapping is the schema: it defines the field types.
- Dynamic mapping infers field types automatically, which can cause a mapping explosion or wrong types.
- In production, it's best to define mappings explicitly.

**11. (Bonus) What are aggregations? Give an example.**
- Bucket aggregations (e.g. `terms`, `date_histogram`) and metric aggregations (e.g. `avg`, `sum`, `cardinality`).
- Example: count products per category and compute the average price.
- *Follow-up:* Why is deep pagination with `from`/`size` a problem, and what's the alternative?
  → Use `search_after`, scroll, or a point in time (PIT).

---

## Part 3: Distributed Systems Basics

**1. What is the CAP theorem?**
- It covers Consistency, Availability, and Partition tolerance. During a network partition, you must choose between consistency and availability.
- Examples: ZooKeeper leans CP, and Cassandra leans AP.
- *Follow-up:* Where would you place Elasticsearch?

**2. What's the difference between strong consistency and eventual consistency?**
- Strong consistency: once a write succeeds, every read sees the latest value.
- Eventual consistency: replicas converge over time. DNS and caches are examples.

**3. What's the difference between horizontal and vertical scaling?**
- Vertical scaling adds CPU or memory. It's simple but has a hard limit.
- Horizontal scaling adds machines, which means you have to handle sharding and state.
- *Follow-up:* What makes a service easy to scale horizontally?
  → Being stateless.

**4. How does a load balancer work? Name some algorithms.**
- Algorithms: round robin, weighted round robin, least connections, and consistent hashing.
- L4 balancers route based on TCP, while L7 balancers route based on HTTP (for example, by path).
- Health checks remove unhealthy nodes from rotation.

**5. What is caching, and what are common caching problems?**
- Common patterns: cache-aside and write-through.
- Classic problems:
  - **Cache penetration** (queries for data that doesn't exist): fix with a Bloom filter or by caching null values.
  - **Cache breakdown** (a hot key expires): fix with a mutex lock.
  - **Cache avalanche** (many keys expire at once): fix with randomized TTLs.
- *Follow-up:* How do you keep the cache and the database consistent?

**6. What is sharding (partitioning)? How do you choose a shard key?**
- Data can be partitioned by range or by hash.
- A good shard key avoids hotspots and distributes data evenly.
- *Follow-up:* What is consistent hashing, and why is it useful?
  → When nodes are added or removed, only a small portion of the data has to move.

**7. What is replication? Compare leader-follower and leaderless replication.**
- Leader-follower: writes go to the leader and reads can go to followers. Followers can fall behind (replication lag).
- Leaderless: uses quorum reads and writes, with the condition W + R > N.
- *Follow-up:* What happens if the leader fails?
  → A new leader is elected, and the system must avoid split-brain.

**8. What is idempotency, and why does it matter?**
- An operation is idempotent if running it multiple times has the same effect as running it once.
- It matters because network retries can cause duplicate requests.
- Common implementations: idempotency keys (request IDs) and database unique constraints.
- *Follow-up:* Is `POST` idempotent? What about `PUT` and `DELETE`?

**9. What's the role of a message queue? Name some use cases.**
- It decouples services, smooths out traffic spikes, and enables async processing.
- Common examples: Kafka and RabbitMQ.
- Delivery semantics: at-most-once, at-least-once, and exactly-once.
- *Follow-up:* How does Kafka guarantee ordering?
  → Only within a single partition.

**10. How do you handle failures between services?**
- Timeouts, retries with exponential backoff, circuit breakers, fallbacks (graceful degradation), and rate limiting.
- *Follow-up:* Why can unlimited retries be dangerous?
  → They can cause retry storms that overwhelm downstream services.

**11. What is a distributed lock, and how would you implement one?**
- It can be built on Redis (`SET NX PX`), ZooKeeper, or etcd.
- Pitfalls: always set an expiration, verify the lock owner before releasing, and handle GC pauses (for example, with fencing tokens).

**12. (Bonus) How do you monitor a distributed system?**
- The three pillars: metrics, logs, and traces.
- The golden signals: latency, traffic, errors, and saturation.
- Distributed tracing: a trace ID is propagated across the whole call chain.

---

## Tips for the Interviewer

- **Pacing:** Pick 3–4 questions per section. If the candidate answers well, go deeper. If not, move on.
- **To tell memorization from real understanding**, ask:
  - "Why?"
  - "Can you give me an example from your project?"
  - "What would happen if...?"
- **Quick leveling guide:**
  - **Junior:** explains the concepts clearly.
  - **Mid:** discusses trade-offs and connects them to real projects.
  - **Senior:** brings up pitfalls, monitoring, and failure handling without being asked.
