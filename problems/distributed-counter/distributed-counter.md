


DISTRIBUTED COUNTER



BACKGROUND
- real Microsoft interview question
- "building block concept" for many other problems



RESOURCES
- NO BOOKS





REQUIREMENTS
- write endpoint for incrementing the counter
- some means of reading the count





VARIATIONS:
- read scale and write scale (overall scale)
- reads vs. writes (proportional)
- latency expectations (is OLAP/MapReduce an option)
- accuracy/precision expectations (CRDT vs sharded counter)
    - idempotency
- real-time rolling window (rate limiter / top K) vs. all-time aggregate (flash sale / inventory)
- push-style counter updates (i.e. web sockets)





APPROACHES:

all-time counts, no rolling window, no "super hot keys"
---
1) single postgreSQL machine
    a) just the counter (non-idempotent)
    b) request table (idempotent)
    c) just the request table (idempotent, full table scans)
2) shard postgreSQL by item ID (without sharded counter)
    - non-idempotent
    - too much write volume for one machine, but no super hot keys yet
3) OLAP (no materialized view)
    - high write volume
    - low read volume
    - slow read latency expectation

rolling-window time-range counters:
---
4) OLAP (materialized view counters)
    - turning approach 3 into "rolling window"
    - we want "exact counts"
    - Spotify Top K songs for example might do this, if it's non rolling window (or, for example, every 5 minutes run a query for looking at past hour)
5) Flink/Firehose
    - we don't care about "exact counts"

all-time counts, "super hot keys"
---
6) redis CRDTs (grow counter)
    - can't handle "exact counts"
7) forgetful bloom filter
    - probably isn't "exact counts" but likely a lot more accurate
8) sharded counter (on it's own, this is not idempotent)
    - e.g. you're stuck with a database that doesn't have CRDTs, such as DynamoDB
    - non-idempotent
9) sharded counter + request table (for idempotency)
    - idempotent
    - manually sharded postgreSQL
10) just request table / event sourcing + full table scans (MapReduce)


Not a "super hot key" approach, although it can be adapted with sharded counters to apply there as well
---
11) whatever I did in my inventory video
    - DynamoDB doesn't handle multiple tables very well, so I had a trick for "denormalizing" the request table a bit
    - you have DynamoDB but you still want "exact counts"
    - I think the way that shopping cart + inventory works at Amazon might look a little like this



12) "youtube view counter"
periodic aggregations + count-min-sketch results (below approach 3 in the diagram file -- count-min-sketch + aggregated exact counts)








CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)









--------------------
LIVE DISCUSSION QUESTIONS:







is the read browser getting pushes for hits? Or is there an implied "request" to the read bak-end service too?
---
Yes, there's an implied request


or is the read back-end service polling the counter data store every so often?
---
polling






We can use Postgres too for approach 1a, right ?
---
all of 1a through 1c is postgres







what’s a request table?
---
a table in a serializable db that records processed requests so we don't double count the same request. **the purpose is to ensure idempotency**



oh I guess in Yubi's OLAP approach it becomes somewhat of a **event sourcing** table. so instead of storing the actual count, we just store each request's ID and then count all the IDs later
---
Yes, it's definitely event sourcing



in the same request table can’t we store the counter value too
---
you can do a 2nd table,
but having it directly on the request table would require total order broadcast

i.e. if you meant:
```
Request Table:
{
    requestID: defabc-123-defabc
    itemID: 456
    count: 5
}
{
    requestID: abcdef-234-abcdef
    itemID: 456
    count: 6
}
```





How count will be aggregated in case of sharded solution, It will be scatter gather ?
---
it'd be a "key-range query"
you still have a part of the partitioning key, so it's not as bad





can you clarify what "no materialized view" means here?
---
it's basically like a 2nd table that's automatically updated by DB triggers and aggregates the counts for you
(it's from old-school 90s SQL terminology, it's still rather common in OLAP DBs)






for a variation: what if we want to include multiple views by one person as multiple "hits"? i.e. I look at post ABC, scroll down to post DEF, then scroll up to post ABC - so my hits for post ABC should be 2, and hits for post DEF should be 1

^ this would probably make using a single request ID usage more difficult

i think this is how fb/twitter count views? (or something really simple in comparison to YouTube, which requires a user to stay on a video for some N amout of time before their view gets counted)
---
> for a variation: what if we want to include multiple views by one person as multiple "hits"?
you would use different request IDs if you're staying idempotent
if not idempotent, the client just does multiple requests

> I look at post ABC, scroll down to post DEF, then scroll up to post ABC
handling duplicates?
--e.g. That's 3 different request IDs, and then ABC and DEF are item IDs




if we use let's say the postID+userID as partition key won't it suffice for -
sometimes, no for multiple views
but for "unique viewer counting", yes you could do `requestID == postID+userID`








the latency for get request on bigquery likes. is really high.
---
youtube view counter,
periodically updated

- you want "high accuracy"
- BUT, it can be eventually consistent

OR,
periodic aggregations + count-min-sketch results


youtube view counter
periodic aggregations + count-min-sketch results







Sorry, I didn’t get how we will do exact count in approach 4 ?
---






what about CRDT that implements a distributed counter over a P2P protocol. 
---
that's approach 6, I think





why is p2p important here? like gossip?

P2P stuff is just add on we need to implement a type of counter which is called G-Counter (Grow-only Counter) with a way of storing and synchronizing the counter value from other agents. 


if we ignore the p2p term, isn't that just a Redis crdt?
---
he's probably specifying that it's a thing in "leaderless" systems, and it's probably gossiping



Can this be done in any programing language ,the design system
---
it depends on what the DB supports for their write conflict resolution
- Cosmos DB -> Lua (IIRC)






How both item and sharid will be used as partition key ?
---
multi-attribute partition keys is a common feature








please post the link to the doc and design 
---
I'll do that at the end :)






But even though we are using Item ID as partition key it would still hit all the shard's correct? so it is still scatter gather correct?

my above point was for approach 8

---

That might be correct, but it does actually scale in the manner that we'd specified the partitioning key here



we'll call it "reduced search space scatter gather" :P



























































