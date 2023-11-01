


FLASH SALE



RESOURCES
- NO BOOKS :(
- interviewing.io: https://www.youtube.com/watch?v=EOe8IkoNKGw
- interviewing.io transcript: https://interviewing.io/mocks/google-system-design-design-a-free-food-app
- Mr CAP: https://www.youtube.com/watch?v=oiPT284NzSs
- discussion about fairness here: https://www.section.io/blog/building-a-fifo-virtual-waiting-room/
- How shopify handles flash sales (InfoQ): https://www.youtube.com/watch?v=MV5Kdwzwcag

"live resources":
- "Specifying Systems" by Leslie Lamport for discussion of "fairness"
- jepsen.io's content on either spanner OR cockroachDB discusses how bad clock drift can get
    - https://www.cockroachlabs.com/blog/cockroachdb-beta-passes-jepsen-testing/
    - https://news.ycombinator.com/item?id=14308387
- https://www.googlecloudcommunity.com/gc/Apigee/Determining-Maximum-TPS-of-APIGEE-Edge-Private-Cloud/m-p/397837








PROMPTS
```
Design a system to sell limited products on SALE.
Feature for ecommerce site who have millions of products. On black Friday, seller offer iPhone on 50% discount (only first 1k product). 100k people concurrently trying to buy it.
DO NOT OVERSELL or UNDER SALE this item.
Payment can fail.

Company: Amazon
Role: SDE III
```

```
Distributing 6M burgers in 10mins. People will come and click on a button on App get My Free Burger. No one should get more than 1 burger. We should not promise more than 6M people that they will get a free burger.

Company: Google?
```

```
Imagine you have 100 new iPhones and you're going to sell them at a massive discount for a promotion for $100/phone. This promotion starts at exactly 12:00. You anticipate 100 million users will try to buy the phones. The phones need to be sold first-come-first-serve. Design a system for this promotion.

Company: TikTok
```


100k
6M
100M

"concurrent" -> 100k TPS
10 minutes -> 10k TPS







REQUIREMENTS
- you have a limited inventory
- **very** high volume of requests coming in
- (depending on variation) retaining order of the requests is "important" to some extent -- "fairness"
- (depending on variation) payments might fail
- (depending on variation) whether counter needs to be EXACT
- (depending on variation) 100ms latency expectaction -- for the "order receipt", or for successfully landing a free burger???









CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)








"How important is fairness?"
Fairness:
- total lottery
- clock drift is acceptable
- total order broadcast (most strict definition)
    - North Dakota dial-up vs. Seattle/NYC gigabit internet



Approaches:
4) manually sharded postgreSQL
5) Spanner
6) federated table (just like I did with hotel booking) -- skipped in this live session, you can just refer to the hotel booking coverage









--------------------
LIVE DISCUSSION QUESTIONS:










any constraints on what db we can use?




can we use something like spanner?
"cop out" answer
auto_incr





What is job of task runner in approach1 and 2?





But in a distributed world, we cannot rely on system time for ordering. 
"Clock drift"
total order broadcast -- which spanner has





what is clock drift?
unix machines get their time off of "NTP" protocol
which has millisecond precison, but accuracy is actually off my 5ms+
jepsen.io's content on either spanner OR cockroachDB discusses how bad clock drift can get
the only solution to clock drift is "total order broadcast"




so what would be the latency if we use Kafka is it an hour or so?
10 TPS for reads, meaning 100ms minimum




clock drift just means the system time runs at different rate on different machines/nodes. so there is not such thing as synchronized clocks across a large number of nodes

"FLP impossibility" is kinda an argument that clock aren't real
(it's actually that consensus can't be achieved if you don't have clocks)
^ DDIA has some stuff on this.



Can you please explain how timestamp across different shards of Kafka maintain ordering . Shards Consumer basically have to communicate with each other right?
kafka used to rely zookeeper -- ZAB is a total order broadcast protocol
I'm fairly certain ZAB is "per shard" ordering
and then, even if ZAB is for the whole kafka topic, consumers can pull at different rates




Kafka latency depends on throughput of the Kafka node which is generally calculated as MB/s and the number of partitions/consumers
\---
100ms minimum due to the 10 TPS upperbound

since each partition is only allowed to be read by a single consumer, the best case is n partition and n consumer 




Instead of Kafka why not SQS?
\---
Backpressure.
RabbitMQ, and SQS are "message queues" instead of "streams" -- I don't believe "message queues" can scale out horizontally quite like streams, AND streams store to disk rather than RAM
kafka does DMA to the HDD

SQS also offers ordering
\---
it can only do that because it's not able to scale out horizontally quite like kafka



yes you are right. Kafka does not guarantee orders across partitions. only within a single partition for a single consumer group



what is the definition of "fairness" here?
This article? https://www.cockroachlabs.com/blog/cockroachdb-beta-passes-jepsen-testing/

from https://news.ycombinator.com/item?id=14308387



what about kinesis?
\---
Also fair game :)
kinesis is roughly the same thing as kafka
"retention policy"







Conflict-Free Replicated Data Type
Google "CRDT"
redis

Pro:
- it handles write conflict resolution for you!

Con:
- it doesn't handle dropped ACKs, meaning you can still get a double count off a CRDT!










"forgetful bloom filter" CRDT

Cosmos DB allows this, but you have to roll the CRDT yourself




for counter approach:
What about using a UUID (or something similar) generated on the client side when they start making an order so that double submits are processed as 1 request, and then sending this to the server.

For incrementing the counter in db, server can send this entire UUID or hash the UUID+userId+timestamp_received_at_server, and then if ack is dropped, resending the same request won't double count one request

\---

"generated on the client side"
that allows for hackers...








bloom filters can have false positives, but no false negatives - is that what you're referring to?
YES :)




to address the hacker spoofing their timestamp on the client side: would it be sufficient to only use timestamps from when the server/service entrypoint receives the request (and ignore the timestamp from the client)?
That's approach 2.
You still get clock drift, which is "good enough" for fairness imo





I am not sure just a guess will idempotency not help here?
forgetful bloom filter can achieve that, regular grow-only counters can't do idempotency



idempotency requires transaction right? I don't think Redis supports multi-instance transactions does it?
\---
single-instance transactions are supported, but yes, I don't think it supports multi-instance




how does approach 3 with multi-instance Redis handle instance failure?
\---
it'd do retries, which is what results in double counts









Redis set also use CRDT right ? can't we store the set of user_id here ? Duplicates think will also not occur here?
\---
I think that's where I was heading with the sharded postgreSQL idea






for db failure (as opposed to server failure):
Redis does have persistence, but this will probably prompt a follow-up question on how restoring from the persisted RDB/AOF works: https://redis.io/docs/management/persistence/
^or network failure
\---
- that's why I didn't bring up the risk of data loss (persistence is an option)
- BUT, you can still get double counts








What problem does the CRDT solution solve if not giving you the exact count? Why is counter for the sale_event_id = 5 giving different counts across the nodes?
\---
the CRDT can handle the scale necessary for the counter.
Some prompts might be okay with a little flexibility (e.g. is "roughly 6 million" okay?)

My CRDT depiction was actually inacurrate there







what about sales where users are allowed to buy multiple items? e.g max 5 items/customer
\---
I'd split them into multiple orders






^throwing in a wrench for the current dedupe logic
\---
Yes :)




and say there's an option for quantity when buying, but customer submits original request for 2 orders, then decides they want another 3
\---
cancelled orders
otherwise, for ordering more, they'd have to submit a 2nd order OR cancel order and place new one with higher quantity

cancellation only works via delaying payment by 30 minutes (which is the period for which amazon allows customers to change their mind)


cancellation might actually be fair scope!
you can release held items back into the inventory similarly to PAYMENT_FAILED







WHAT about Zookeeper: global counter based on consensus. 
\---
we're going to cover spanner in approach 5




nice! this is my first choice as well. although operationally this is the most expensive. 100k TPS would require 20 physical shards





can zookeeper handle 100k tps?
\---
Good question.





How will be handling the case where we have inventory on some other shard but user will belong to another shard

Yes. The Amazon prompt for flash sale *would* result in unfairness at the tail end of inventory count as some shards run out before others... -- So, it does have some degree of unfairness


So, you can do something like hold in the back-end service a "partition ID" rather than using the "user ID" that was provided -- So, you're doing retries to try to find inventory (that actually results in scatter-gather, which doesn't really scale well)





don't shard user id. just round-robin?
\---
the last 10-20 items would definitely have a higher risk of being handled unfairly




As part of second approach timestamp was getting captured but I could not able to understand how timestamp will help in ordering as message will be still be distributed across partition





shouldnt sharding key be item id? So all order and inventory stays in same shard
\---
the issue is that the item ID is a hot partition, and so hot that you need to do a "sharded counter", the counter needs spread across multiple machines



but I guess that still has the risk of landing on a shard "unluckily". so maybe we have to try multiple times?
\---
Yes, you'd basically end up with a scatter-gather on the tail end as described earlier





Cloud Firestore can be used for distributed counter
\---
I haven't really investigated Firestore before, but that sounds like a very solid solution






quickly skimped through some zookeeper doc. it seems like with a single leader quorum setup, it can only handle 50k tsp



Won't inventory table's inventory_count be a SPOF ?
\---
you can set up "failover" with one synchronous follower (which is what I imagine DynamoDB does under-the-hood)






Sorry if you’ve answered and np if you don’t have time to answer (if you want to go over spanner) - could you give a brief overview of the two postgresql solutions and how they differ?
\---
it's just whether the 2nd step of auth-capture is made aync







this answer estimates a max of ~1000 tps/node on zookeeper - also includes lots of considerations https://www.googlecloudcommunity.com/gc/Apigee/Determining-Maximum-TPS-of-APIGEE-Edge-Private-Cloud/m-p/397837

(haven't found any exact numbers or more authoritative sources so far - might be looking  for the wrong thing?)



slightly tangential, this is kinda interesting too https://www.instaclustr.com/blog/apache-zookeeper-meets-the-dining-philosophers/



oh the 50k tps is totally different from the 1k tps here 😂









SELECT SUM(o.item_quantity)
FROM inventory i
INNER JOIN order
ON i.item_id=o.item_id
AND
(
o.order_status=AUTHORIZED OR
o.order_status=PENDING OR
o.order_status=PLACED)


my SQL query attempt^
might be totally wrong though, test in staging first


probably expensive just an example of sql approach vs spanner








why not just store each order as a row in spanner and run a transaction of count-rows-then-insert. 1M rows is not too hard to count. the row data is not too large so not too many tablets
\---
"count-rows" is scatter-gather which wouldn't scale very well
1M rows is actually a very large amount to count when the TPS is 100k







this was completely awesome discussion bunch of things to takeaway as part of homework, thanks a ton!!!


I think you are totally right. 100k tps is too much



please post the textfile link for us to go through





































