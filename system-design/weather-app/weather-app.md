


WEATHER APP WITH MILLIONS OF TEMPERATURE SENSORS



RESOURCES
- NO BOOKS








PROMPT
```
Design a system that takes temperature readings from 1,000,000 sensors across WA state. They communicate over HTTP. Display the data as a bitmap image, assuming each sensor represents exactly a single pixel (no need to interpolate). Also need to be able to get the maximum and minimum for any particular day across the whole state.

Company: Amazon
Role: SDE III
```






REQUIREMENTS
- millions of sensors
- display a bitmap image for a map of the output
- record min and max for the day


- device failures


VARIATIONS
- Real-time vs. once per day image







CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)








- live updates of the image
- dead sensor detection










--------------------
LIVE DISCUSSION QUESTIONS:








is it going to be real time service? or near real time is fine
\---
I'm going to cover both






Man I got that as a mid level engineer interview at Amazon
\---
prompts can be varied and targeted at several different levels




1 million sensor reading is not a lot of data is it? what does the data look like? just an int?
\---

1M @ once per day -> 10 TPS
5M @ once per day -> 50 TPS



1M @ once per hour -> 240 TPS
5M @ once per hour -> \~1250 TPS





machine count at sr, latency count at staff
\---
^ yup, that's in google's system design rubric





"sr staff is expected to talk about multiple solutions, pros and cons for each one"
\---
speculation or serious answer?


"about the sr staff comment, yes I am serious "










i was asked 1M req per 5 secs at amazon for L6

1M req per 5 secs
200k / s




1M @ once per day -> 10 TPS
5M @ once per day -> 50 TPS

1M @ once per hour -> 240 TPS
5M @ once per hour -> \~1250 TPS

200k TPS -> 20 machines for your database





yeah let's take 200k TPS for today is that okay?
\---
Yes :)






Since we are reading from 1,000,000 sensor and they communicate over http ( /v1/getTempReading).What will be the response format?does it respond the reading for all 1M sensor when we make http get?
\---
Do we call the sensors or do the sensors call us?

we're going to do "the sensors calling the service" (rather than the back-end server calling the sensors)

I would personally even consider the approach of server calling sensors "Naive"
- job scheduler style approach from the server's perspective
- the sensors would be running as web servers
- the server end has to hold a connection open for a more extended period of time





this problem is kinda similar to the real-time navigation question
\---
I'd love to see a prompt or link for that problem



sorry can you please explain how does map reduce help here - am not that clear on that
\---
translation of sensor ID to x, y pixel of the image
"hey, we have to do a full table scan" is more of what I intended to communicate



can we not use CDC rather? to talk with S3 (just putting my views)
\---
the 200k TPS of changes to the image is the issue with trying that





I think it's important to discuss what the timestamp is, is it processing time or is it event time. and of course the pros and cons of both approaches 
\---
**sensor timestamp** vs. server/backend timestamp


sensor timestamp
Pro:
- more accurate for any delays that could occur over network (particularly if I throw kafka ahead of any data store)
Con:
- takes up more bandwidth with an extra int going over the network call






alright got what you are doing with map reduce job but maybe CDC can be used here, correct me if am wrong here...




there are pros and cons to pull vs push models. this is the same discussion as the metrics system question

More pros & cons for this:
- server polling the devices could be better for device failure detection
- retries can easily be handled from either the server or sensor side



we can send 202 async processing instead of sending 200 and waiting to write complete in DB
- what if you get a dropped write to the DB? you could've retried from the sensor instead
- multi-threading within a single request on the server adds significant complexity









do we need to worry about time drift here? probably not, since the data from one sensor should always have consistent (monotonically increasing) times, correct? Or do we need to worry about time adjustment here too?
\---
do we need to worry about time drift here? -> I don't think so, mainly a concern for transactions,

"Understanding Distributed Systems" -- Roberto Vetillo
    "monotonic operations" means you don't need the stuff spanner does,
    and that's also our case here -- if any of the writes are re-ordered it won't make a difference
When is eventual consistency not good enough?
When can you not do compensating transactions?






Can we enqueue writes to a queue for a different approach? Might be jumping ahead here









- live updates of the image
- dead sensor detection
- where to store max/min outputs








Can we enqueue writes to a queue for a different approach? Might be jumping ahead here
\---
Yes you were jumping ahead :)





Also would people want to know our tolerance for dropped requests from the sensors? May have missed that if we discussed requirements already
\---
dead sensor detection could help prevent the SLO of data staleness from getting violated,
maybe assume up to 24 hours of staleness is fine






if you wanna use CDC, then you need to do something like CQRS, aka separate the data write from the image read(generation)






the main advantage of using pull (from sensor) is the ability to detect down sensors more easily
\---
I think so :)




oh also, sampling becomes easier using pull as well. then there's the ability to scale the backend independently from the number of sensors deployed







@yubi what's task of task jobs in approach 3?




@Yubi, separate read optimized data store is what I meant with CQRS.

since the image is fixed at 1 million pixels,
I would actually store that entirely in memory
\---
maybe CDNs do that under the hood

maybe ElastiCache is a good option here too








found this for redis pub/sub benchmarks: https://groups.google.com/g/redis-db/c/R09u__3Jzfk
\---
^ context for the above is that I didn't think redis pub/sub could handle throughput as well as kafka, but redis pub/sub does have the advantage for end-to-end latency



Salvatore Sanfilippo (antirez) is the creator of Redis and he responded here, so the answer should be reliable - this is from 2011-2013 though, so might be out of date now


also found https://wiki.openstack.org/wiki/Zaqar/Performance/PubSub/Redis and https://stackoverflow.com/questions/59873784/redis-pub-sub-max-subscribers-and-publishers












sorry my comment wasn't clear. I was more thinking on-demand image generation. so store the image in memory. and whenever an image is requested we use memory data to dump a s3 file so we save storage





pub/sub is push based. it's not designed for throughput 



Kafka throughput is based on message size. a single node can handle about 600MB/s which means 100k(s) messages easily






lets say we go one step more and expect 1 million sensors sending data per second. What changes are we looking at? Cassandra still good?
\---
we just use 5X as many machines











Reids is not as high:

"Different kind of clients have different default limits:

Normal clients have a default limit of 0, that means, no limit at all, because most normal clients use blocking implementations sending a single command and waiting for the reply to be completely read before sending the next command, so it is always not desirable to close the connection in case of a normal client.
Pub/Sub clients have a default hard limit of 32 megabytes and a soft limit of 8 megabytes per 60 seconds.
Replicas have a default hard limit of 256 megabytes and a soft limit of 64 megabyte per 60 seconds."

from https://redis.io/docs/reference/clients/






since we partition by sensor id, Cassandra just linearly scales. at some point, compaction might be an issue though. I think discord ran into something like this










we may need to touch on data purging. Some background job that would do some sampling and remove data

{
    sensor_id: 345,678                --   (partitioning key
    timestamp: 1670004000.       --   sort key) - primary key
    temp: 75.2
    lat: -45.3
    long: 23.2
}

^ 5 numbers

4 bytes per number



20 bytes per record

200k TPS

4000k
4M bytes per second

4k KB
4 MB per second

100k seconds per day

400k MB per day

400 GB per day








I had heard about the discord issue but can you explain if possible what is the issue with cassandra scaling linearly? And do other LSM based DB's don't scale that way?

@Yubi, if we do CQRS, then the web socket (read) side can just update at a different rate, say 60hz to get a smooth animation

1000 messages per "pull" from kafka



200k TPS

200 TPS

1k TPS just fine, possibly



1 read from a sensor every 1000 seconds would probably work just fine off using kafka to batch up the reads













I had heard about the discord issue but can you explain if possible what is the issue with cassandra scaling linearly? And do other LSM based DB's don't scale that way?












this might be the blog link that briefly discusses compaction issues Discord ran into: https://discord.com/blog/how-discord-stores-trillions-of-messages

"Cluster maintenance tasks also frequently caused trouble. We were prone to falling behind on compactions, where Cassandra would compact SSTables on disk for more performant reads. Not only were our reads then more expensive, but we’d also see cascading latency as a node tried to compact."






I don't know if ScyllaDB uses LSM, but it is implemented in C++ so garbage collection is more predictable (as opposed to the Java implementation of Cassandra, which is using one of Java's garbage collection algorithms). The GC can be tuned for Java too, but I'm assuming it's still tricky/Discord wasn't able to do so for their workload




so ScyllaDB does use LSM based on the short snippet here https://www.scylladb.com/presentations/scaling-scylladb-storage-engine-with-state-of-art-compaction/ but can't see the whole thing right now

it* = ScyllaDB is implemented in C++






there might be more stuff to it though: "Log Structured Merge (LSM) tree storage engines are known for very fast writes. This LSM tree structure is used by ScyllaDB to immutable Sorted Strings Tables (SSTables) on disk. These fast writes come with a tradeoff in terms of read and space amplification. While compaction processes can help mitigate this, the RUM conjecture states that only two amplification factors can be optimized at the extent of a third. Learn how ScyllaDB leverages RUM conjecture and controller theory, to deliver a state-of-art LSM-tree compaction for its users."








if we make the update service Kafka consumers then we don't need the Redis mapping 





ohhh I see what you mean by "update service". it's actually the gateway. in that case no, we wouldn't want to put Kafka consumer on the gateway container









Qq: why do we need sockets if only there's one way communication? We can rely on webhooks
\----
I think webhooks are intended more for servers




web socket is far more efficient since it avoids all the http handshakes


I see. For bi-directional communication websocket looks great but for one way data flow there could be other options as well




can we use something grpc and batch the data before pulling?
\---
gRPC doesn't do batching, it's more of a code-generation thing in static typed languages (e.g. Amazon's Coral tool)


sorry HTTP2.0
\---
I'm not familiar with that





























































