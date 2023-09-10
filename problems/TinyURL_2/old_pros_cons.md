

# TinyURL: System Design - 7+ Approaches with LeetDesign


1) No Cache
2) Lazy-Loaded Cache
3) Eager-Loaded Cache, Synchronous
4) Eager-Loaded Cache, Async "Fire-and-Forget"
5) DB Triggers
  a. Synchronous
  b. Asynchronous
6) Monolith (with Lazy-Loaded Cache)
7) Message Broker




Approach 1: No Cache
---

Pros:
* Simplicity
* Could support read-your-own-write consistency

Cons:
* Increased read latency from lack of a cache




Approach 2: Lazy-Loaded Cache
---

Pros:
* Less latency on the write operation
* Could support read-your-own-write consistency

Cons:
* Likely some latency on the first few reads




Approach 3: Eager-Loaded Cache, Synchronous
---

Pros:
* First few reads of the shortened links have a guarantee of being in the cache ("primed cache")
* Could support read-your-own-write consistency

Cons:
* Increased latency on the write operation
* Degraded write operation availability from dependence upon TWO data stores




Approach 4: Eager-Loaded Cache, Async "Fire-and-Forget"
---

Pros:
* First few reads of the shortened links have a greater chance of being in the cache ("primed cache")
* Could support read-your-own-write consistency

Cons:
* Increased complexity




Approach 5a: DB Triggers - Synchronous
---

Pros:
* Stronger consistency guarantee between the two data stores
* Could support read-your-own-write consistency

Cons:
* This isn't really a thing
* Higher write latency
* Degraded write operation availability from dependence upon TWO data stores




Approach 5b: DB Triggers - Asynchronous
---

Pros:
* Simplicity, among CDC approaches
* Less latency on the write operation, among caching solutions
* Could support read-your-own-write consistency

Cons:
* Flakiness
* Lots of overhead for the DB




Approach 6: Monolith (with Lazy-Loaded Cache)
---

Pros:
* Simplicity
* May require less operational and KTLO work
* Could support read-your-own-write consistency

Cons:
* Loss of deployment and scalability independence




Approach 7: Message Broker
---

Pros:
* Decoupling allows for easier DB schema migrations
* Probably offers the highest availability solution for the write operations

Cons:
* Increased complexity
* Inability to support read-your-own-write consistency
























































