


TWITTER



RESOURCES
- similar to "Tagging Service" design
- there's plenty of books that cover "newsfeed" or "twitter"





REQUIREMENTS
- Users can make a tweet
- Users can view their newsfeed
- Users can view tweets of a specific profile

- filtering / harmful content (approach 5)

OUT OF SCOPE:
- pictures attached to tweets
- extended services like retweet,like/comment , analytics



VARIATIONS
1. low volume -- 5,000 TPS of reads, 100 TPS of writes
2. high volume











CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)













--------------------
LIVE DISCUSSION QUESTIONS:







do you netflix sys design questions?
\---
https://www.youtube.com/playlist?list=PLlvnxKilk3aKQSAzomjJimVECgPEwSgT2



i got design twitter for my very first system design interview






should we mention on common services like user service or extended services like retweet,like/comment , analytics ?



Do we store tweet text in DB or S3 and store URL?





what db we are using for Tweet DB?




at what point do we store text blocks in something like s3? for example, you stored leetcode questions in block store irc
\----
the limit for postgreSQL records is 2MB, but you probably don't actually want to go that high





How's the Kafka partitioned on topics?





in timeline db schema, follower id is the person you are building the timeline for?
\---
Yes, that's correct :)




Timeline DB can be simply KV database like DDB?
\----
DynamoDB supports 20-30 secondary indices
at least 15 secondary local
at least 5 global secondary

KV database
memcached -- actually probably doesn't support secondary indices








but we cannot have Tweet DB as NoSQL right coz we are using dB triggers which would be applicable on SQL only
\---
DynamoDB does support DB triggers







timeline db should be write-heavy OLTP?
\---
totally possible that only 25% read rate on the tweets in timeline DB





I think we need reads on Tweet table if I want to look at profile of any user (all of user's post)




wouldnt be having only one broker be bad HA





In one of my interviews, they specifically asked to have some entry points for OLAP stuff like user recommendations. Can OLTP can do EOD batch stuff to some OLAP? 






do we need secondary indexes for Timeline Table?









so just one query is it okay to think or say that whenever we are using ORDER BY, we can say that we need secondary indexes on top of that field?
\---
usually secondary indexes will really help with speeding those up





We would need both negative and positive cases in the warehouse to train right, so shouldn’t the data be coming from the first kafka broker
\---
Yes, you would need both





I think for cold start problem (new user's tweet), we rely on Tweet DB 





also we have used cache here for hot key/partition issue, when should we prefer consistent hashing can you give any real life example please
\----
cache -- for key-range queries
consistent hashing -- for key-value lookups



twitter.com/tweet/347358723587



SQL key-value query:
SELECT 347358723587 FROM tweet_table


SQL key-range query:
SELECT * FROM timeline_table
WHERE follow_table.followerId = 'YubiDesu'
ORDER BY timestamp
LIMIT 200
OFFSET 1000














































































































































