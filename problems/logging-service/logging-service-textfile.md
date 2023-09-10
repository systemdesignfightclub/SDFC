


LOGGING SERVICE



RESOURCES
- problem requirements context:
    - https://leetcode.com/discuss/interview-question/system-design/622704/Design-a-system-to-store-and-retrieve-logs-for-all-of-eBay
    - https://leetcode.com/discuss/interview-question/system-design/292216/Design-Error-Logging-service/283697
    - https://leetcode.com/discuss/interview-question/system-design/1753420/design-a-log-aggregation-service-to-get-top-n-errors-within-a-given-time-range
- from the SDFC discord's thread about metrics: "applications like Prometheus do pull-based" (Thanks, @BOSS!)
    - https://prometheus.io/blog/2016/07/23/pull-does-not-scale-or-does-it/
- there's a trick to doing streaming logs of kubernetes (Thanks, @neakor!)





REQUIREMENTS
- `Design a system to store and retrieve logs for all of eBay` (quoted from one of the leetcode threads above)





VARIATIONS:
- "real-time" requirement for viewing the logs





APPROACHES:
1) push-based
2) pull-based
3) sync vs. async
4) continuous
5) ElasticSearch vs. Data Warehouse, such as dremel/presto/trino on iceberg/parquet files




Next:
- synchronous approach
- continuous, push-based, async w/ kubernetes
- data warehouse







push-based vs pull-based trade-offs:
- who gets the retry logic
- who has to hold the network connection open
- the collection backend service will have to be aware of all daemon agents

push-based:
daemon agent is responsible for recording offset
pull-based:
more ambiguous, possibly the collection service back-end
















CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)









--------------------
LIVE DISCUSSION QUESTIONS:





Have you found it useful to go through the references in Alex Xu - Vol. 1 & Vol. 2, DDIA and Understanding Distributed Systems books
\---
Alex Xu's, yes
DDIA, I haven't gotten that far yet




hello, how long do you think it should take a student to get to a good enough level in system design
\---
Define "good enough"?





He yubi,
How one can get practical hands on also along with this theoritical study, any action plan for that to begin with ?
\---
- Designing Distributed Systems by Brendan Burns
- https://fly.io/dist-sys/
(Thanks, @Loading)




If you're reading Brendan Burns, I recommend reading all his books. Coz they share similar concepts

Kubernetes Patterns, Kubernetes Up & Running






This is a meta SD question.







What protocol Daemon service will be use to communicate the collection service (HTTP ? ), it will be sync or async call ?


what is being transferred from the daemon to the collection service?





what is being transferred from the daemon to the collection service?



What happen when a log from a service is very large? if we have 100k character in a log, that would be 100kb/log?









grpc streaming for push?





local logs at node persist in disk like WAL(append to file sequentially)? What are the query patterns Logging Service supports apart from searching( doesn't it have to support key range based query)?
\---
yes, the .log file should basically be a WAL






Do we expect log entry to be very big structured data?
\---
So, the .log file is unstructured
ElasticSearch should be structured
.parquest is structured



do we really need log file ? can we not directly upload to Collection service ?
\---
"continuous" means more overhead spent on network traffic
batched makes one round trip for a grouping of log statements





SaaS product to hire freelance service and provide freelance service









In the pull based model , the collection backend service will have to be aware of all daemon agents, not sure if that's too efficient

especially if we have a prod service that frequently auto scales








should we not batch multiple log line to honour rate limits in collection service backend?
\---
batched definitely does cut down on the overall number of round trips occurring over your network







For synchronous, maybe add a log buffer in memory.
\---
I think this would immediately make it "async"





how do we maintain state ? who is Responsible for maintaining the checkpoints of which point a log file is read ?
\----
push-based:
daemon agent is responsible for recording offset
pull-based:
more ambiguous, possibly the collection service back-end





imo, push won't scale well. the logging service would need to scale with the sum of all services. pull would decouple producing from consuming. this is the same choice prometheus and Kafka made



how do we handle back pressure also ?


































