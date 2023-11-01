


SCHEDULED DIGITAL TRANSACTION



RESOURCES
- distributed cron has some overlap
    - https://github.com/systemdesignfightclub/SDFC/tree/main/problems/distributed-cron
- digital wallet has some overlap
    - https://github.com/systemdesignfightclub/SDFC/tree/main/problems/digital-wallet






PROMPT
```
Design a system to pay <robux, account_id, timestamp> at timestamp
```







REQUIREMENTS
- We have unlimited robux
- we want to deposit robux at timestamp to account_id
- no PCI compliance requirements, or anything similar that's required for handling real money
- assume account info is available
- transactions with strong consistency
- availability = the request will get an immediate response with the transaction in a status of PENDING
- 100 TPS for transactions
- 1kB per transaction, 2000 days of data = 20TB





VARIATIONS
1. 100 TPS scale
2. let's definitely crank up the volume on the TPS of those transactions
3. cross account transaction, let's maybe remove that assumption of "unlimited robux"




APPROACHES:
- "scheduled" aspect:
    - distributed cron
        - actual cron
        - golang threads
    - polling







CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)













--------------------
LIVE DISCUSSION QUESTIONS:






whats a DB trigger and how does it work?
\---
WAL - "write ahead log"




https://github.com/systemdesignfightclub/SDFC





Can you please explain in some more details what is the job of cron replicas?
\---
for redundancy if any of the nodes dies


is this design for a single system to assign resources for each job, or it is a design for distributed system.








in that case how cron replicas organize the tasks and allocate resources?







were you thinking about some kind of a distributed queue instead of send service?




not in place of send service, i was thinking in place of cron replicas we could use some distributed queues




Okay, so as per the timestamp in which the jobs are submitted, db triggers activate the cron replicas, which in turn will execute the jobs via, wallet service.hope I got the understaning right






Hi I have a question. I noticed you use database trigger for change data capture a lot.
\---
Yes I do :)

This requires polling?
\---
it actually doesn't

DDIA talks about using WAL. Would you mind sharing your thoughts? Sorry if repeat.






where is the initial robux,/timestamp/account id stored and where the scheduler service is reading from ?





1. Is 1 poll per 60 s enough?
\---
you would need to clarify with your interviewer, and depending on their answer, polling might not actually be a viable option for this problem



2. I see that you have a select * query which means you are locking on to an sql db for event timestamp?





I saw a couple of methods to dedup, idempotency keys vs 2phase commit.
\---
I'm focusing on the idempotency keys approach

Does Kafka do native deduping?
\---
I really need to look that up :)






How in the first design it ends up in multiple calls for the same transaction to the wallet service?Sorry if it is a dumb question.
\---
the cron service has "redundancy" meaning multiple machines will be activated for the same event_uuid



How many tps can postgres realistically scale to ?
\---
10,000 TPS for one machine


Would we have to consider things like iops, shared mem size etc 


{
  r-tree
    lat:   B-trees
    long:   B-trees
}


what db we can use for event timestamp series ?










Unrelated to this video, but wanted to understand how to we choose which DB to use for a design use case...
\---
go to #read-me in my discord group
DDIA chapter 3 (LSM-trees, B-trees, r-trees)
Database Internals by Alex Petrov
- consistency level support
- read-heavy vs write-heavy


and how do we set indices. 
\---
https://www.w3schools.com/sql/sql_create_index.asp







why do we need cron replicas , db trigger can fire at the desired timestamp and call a service which in turn call sendservice ?
\---
"db trigger can fire at the desired timestamp" -- I'm not sure about that

redis has TTL with triggers activated by expiration

this would possibly be an alternative to golang threads / cron / polling




some scheduler like quartz can run at specific time recorded at db and call send service
\---
1. is this a "cop out" solution that an interviewer wouldn't be happy with?
2. how does redundancy work?










Would manually sharded pg involve hosting multiple master pg dbs which have identical schema

and data is routed by application or pg can do this internally using it’s partitioning feature ? 
\---
"Vitess" -- used by youtube
MySQL




Im guessing the second one will not scale horizontally




I think scylla is also supposed to make this easier, but not sure about other limitations









what happens call to sendservice or send service calling wallet service fails.? 
\---
1. it should retry
2. if wallet service or send service is simply "down", that's an outage event -- you'd need to do a "backfill"




Negative balances will be handled by db constraints ?



TRANSACTION BEGIN

SELECT 

UPDATE

TRANSACTION END




My bad, instead of scylla we should probably be looking at cockroach instead. Scylla is more of a replacement of cassandra



would it be good to record failure to sendservice or wallet service in a db and then retry later ?





















































