


DIGITAL WALLET



RESOURCES
- Alex Xu Volume II
    - Digital Wallet
        - very confusing Raft coverage
    - Payment Service
- Stripe auth-capture flow documentation






REQUIREMENTS
- 1M TPS
- send transfers between accounts
- auditing
- variation: strong read consistency of sender's balance







CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)






Approaches:
1) DynamoDB, in-lined transactions to wallet/account records
2) manually sharded mostgreSQL, with the transaction table split out
3) remove the redundant transaction table, w/ strong read consistency, DB trigger off the end of the wallet DB
4) hot partitions for accounts with lots of transactions







--------------------
LIVE DISCUSSION QUESTIONS:







so here we should definitely use CDC between TransactionDB and Wallet DB and have kafka between them? pls correct me if am wrong here





so is DB triggers same as Transaction + OutBox pattern?


if no which is better?

Leo not really. db trigger is just one implementation of cdc. the transaction + outbox pattern is used to ensure exactly once message publication





can you briefly explain what an orchestrator is? Is it just a central service that coordinates a bunch of different services?
\---
similar to an AWS lambda function, but it does several steps, and it's really good at retries
it's like a bunch of AWS lambda functions -- it's a bunch of API calls in the form of tasks that form a DAG

- Netflix Conductor
- AWS Steps
- "Herd" (Amazon Proprietary)
- Luigi, Airflow
- Argos Workflows





it's not one vs the other. rather they are just different things


what db are you thinking for both services? I think you mentioned partition key + clustering key. so NoSQL? how would that work for db transaction?

- DynamoDB
- Spanner
- manually sharded PostgreSQL
- Cosmos DB







btw how about using DB as Cockroach DB? I feel even that is good choice for this (it has inbuilt feature for CDC)









from the explanation here: https://learn.microsoft.com/en-us/azure/architecture/best-practices/transactional-outbox-cosmos

It ensures events are saved in a datastore (typically in an Outbox table in your database) before they're ultimately pushed to a message broker. If the business object and the corresponding events are saved within the same database transaction, it's guaranteed that no data will be lost. Everything will be committed, or everything will roll back if there's an error. To eventually publish the event, a different service or worker process queries the Outbox table for unhandled entries, publishes the events, and marks them as processed. This pattern ensures events won't be lost after a business object is created or modified.




Yes I read the same on microservices.io



So Cockroach DB uses Outbox pattern for CDC


found it: https://learn.microsoft.com/en-us/azure/cosmos-db/consistency-levels



what db are you thinking for both services? I think you mentioned partition key + clustering key. so NoSQL? **how would that work for db transaction?**
\---
eventual consistency with sagas








Sorry maybe this is not exactly related to the problem you are covering today 
but just a extension on this flow you have made.

Let's say after we deduct money from Wallet we have say eg:- Order Placing service.
(So there would be 2 concurrent transactions taking place here both by user1) - 

Let's say user1 has 50$ in his wallet and starts a transaction1 and everything is 
successful and 50$ are deducted from user1's wallet (as part of transaction1) but now it fails in the
order placing service (let's say if the retries also fail due to servers being down) So it should do a 
rollback right?

Now if user1 does a concurrent transaction2 worth 30$ user1 should actually be able to do it as the earlier one failed the 50$ amount was not debited from user's account but due
to inconsistency transaction2 sees that there is no enough balance in user1's wallet.

How do we handle situation like this in case of Saga's pattern, incase of Distributed 
Transactions things work fine due to locking (which is definitely very bad)

I hope you get the overall gist of my question in a gist if due to inconsistencies 2 concurrent transactions
don't see correct state of Wallet DB how do we handle it?







PG transaction size seems to be around 2**32 (or 2**31 - 1):
https://stackoverflow.com/questions/709708/maximum-transaction-size-in-postgresql
https://stackoverflow.com/questions/60574381/postgresql-max-transactionid-4-billion
https://dba.stackexchange.com/questions/185320/postgres-cost-of-large-volume-of-inserts-in-many-tables-in-a-single-transaction
https://dba.stackexchange.com/questions/307005/postgres-autovacuum-keeps-transaction-ids-around-to-10-limit-causing-aggressi





is step 2 and 3 a part of transaction?
\---
it is considered a "distributed transaction"
but we're using sagas
which is "eventual consistency"
it's probably fair to say these transactions are not "ACID" compliant though





or are independent transactions
\---
the whole thing is considered a "distributed transaction"
which consists of individual ACID transactions


async maybe...
\---
async is an accurate word



They are AWS steps, so should be async.
\---
Correct :)




which is bad I believe coz what if 3rd step failed 
\---
- you've got retries
- in your DAG, you can roll back step 2 as part of the failure scenario handling


but how are we ensuring idempotency then


thanks for answering my questions maybe I'll pause for a while coz want to hear more approaches XD



saga requires compensating steps. so if the last step fails, we'll need to manually perform a "roll back" at the application level







usually am confused at say there are 2 concurrent transactions taking place, we deducted money from user1 (it failed in user2's step).




is the original idempotency key generated on client? 





and other transaction was deducting money from user1's wallet but as the earlier one did deduct money we cannot go with transaction2


Can you discuss more about indexes in Wallet DB as we need to search the txn_id . ( you have discussed but I missed)







txns: [{
    txn_id: abcdef-234-abcdef            -- this is for idempotency
    amount: '-50.00'
    status: RECEIVED/FLATTENED
}]







Doesn't we have hotspots issues one account does lot of transaction ( similar to celebrity problems), if we choose account_id as partitioning key ?
\---
we can treat the account balance similarly to a sharded counter
you'd need to use the shard_id as part of the paritioning key, for handling idempotency

approach 4)
hot partitions for accounts with lots of transactions



if we combine the transaction table and account table into the same db, then how do we partition? if we partition by transaction id, then a single transaction might land on 2 nodes -> continued

if we partition by account, then reading transactions becomes a scatter-gather. I think it makes more sense to have transaction and account be in two separate dbs and services 
\---
then how do we partition?
->
By account_id

then reading transactions becomes a scatter-gather
->
a key-range query for a specific transaction_id would definitely be scatter-gather and at high scale, it'd definitely be a design killer -- but, I can't think of where it'd occur outside of the auditing table -- plus, you can look up both account_id s off the transaction record



If we go with having account balance as a sharded counter, if we get a case where in amount to be debited > sharded balance, then we have to reach out to more than 1 shard. How will we handle txn id?
\---
somebody with low funds shouldn't be singlehandedly responsible for 1000s of TPS of transactions -- it's more likely to be Walmart / amazon.com





what is the user of sharded_counter_id ?
\---
it's applying the concept of a "sharded counter" to the account balance



1000
over 3 machines


400 -- machine A
300 -- machine B
300 -- machine C






**I'm fairly certain there's no forgetful bloom filter thing for an account balance -- only for counters**
































