


AD CLICK ECOSYSTEM



RESOURCES
- Khang Pham's book
- Alex Xu's ML book
- Ad click aggregator from Alex Xu Volume II
- My previous coverage of ad click prediction and ad click aggregator





REQUIREMENTS
- ad click auction + impression placement
- ad click aggregation








APPROACHES:
1) impression serving / "placement" ID
  a. "impression ID" = Ad ID + User ID + de-dupe key
  b. Ad ID + de-dupe key
2) One big ML Model vs. model for each ad from the "Model Store"
  a. Model(Ad Features, User Features) -> click probability
  b. Ad ID -> Model(User Features) -> click probability













CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)













--------------------
LIVE DISCUSSION QUESTIONS:






Draft files:
https://imgur.com/a/TTq48vH







Curious about how to vary the data flow based on privacy regulation (EU user = GDPR vs Cali user VS rest of USA, for example)



and how to utilize browser fingerprinting for tracking in addition to (or in leueu of) cookies









Hi did we also discuss the QPS and other load for this problem? If not any estimate I assume this would be Ready heavy and are we talking about planet scale here? 1e7 something
\---
10M+ TPS of reads
100k+ TPS of clicks




Is it normal to need 2 network calls for getting Ad ID and the actual S3 URL?









Would this be stateful design? just would like to know how would the design change from higher level if we want to go with stateless design







Alex Xu, talked about event timestamp and processing timestamp. I think in this design we don't have to care Watermark if we store event timestamp as it is not real time and not using stream windowing









just wanted to ask in general, so here would we do the partitioning by userID or by Ad_ID coz if we do by AD_ID -> It would be a scatter_gather if we want to fetch which all type_of_users have clicked the AD

but if we want to know what all ADs did a user clicked on that would be a scatter gather right?



And if we partition by UserID, what if we want to get the data of what is the topMost ads/area_of_interest for all the users, this would be a scatter_gather again.






will adding secondary indexes (local or global) work? (PS :- I just added this point out of curiosity to know more if that would really work)
\----
data warehouses tend to not have secondary indices because you tend to do full table scans

"columnar storage"

Dremel whitepaper:
- "in-situ" ("in-place") compute -- you bring the compute to the storage
- columnar, and document-based







I missed the live session, how do u guys get notified when it is scheduled?
\---
there should be a little bell icon


it is usually the same timing everytime but you can join the discord for that
\---
it's at 10:30pm PST every weekend






can we use redis sorted set for top k metric here?


for top k we can use count min sketch


















































