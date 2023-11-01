


TICKET MASTER



RESOURCES
- Grokking has some content, but I wouldn't recommend it
- from my previous coverage:
    - Flash sale problem
    - airline ticketing problem











One prompt for ticket master...
```
Functional Requirements:

    The system should be able to list down cities where its cinemas are located.
    Upon selecting the city, the system should display the movies released in that particular city to that user.
    Once the user makes his choice of the movie, the system should display the cinemas running that movie and its available shows.
    The user should be able to select the show from a cinema and book their tickets.
    The system should be able to show the user the seating arrangement of the cinema hall.
    The user should be able to select multiple seats according to their choice.
    The user should be able to distinguish between available seats from the booked ones.
    Users should be able to put a hold on the seats for 5/10 minutes before they make a payment to finalize the booking.
    The system should serve the tickets First In First Out manner

Non-Functional Requirements:

    The system would need to be highly concurrent as there will be multiple booking requests for the same seat at any particular point in time. The design should be such that system handles such ambiguity fairly.
    Secure and ACID compliant.
```





FLASH SALE PROMPTS
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







Variation 3:

Approach 1:
- Amazon's Patent for XOR trick

Approach 2:
- "pre-cut" groups of tickets












--------------------
LIVE DISCUSSION QUESTIONS:








When is it best to do replication v sharding for DB? Is it a matter of read-heavy v write-heavy?

replication handles high read traffic volume
sharding handles high write volume




Can we use ES as primary data store?
No. (surprisingly)
Documentation explicitly says this







The latest versions of ES attempts to solve split brain as they use simplified raft.





Can you share documentation? My company uses Elasticsearch at a scale of 100s of TB as primary db.




we would also need the exact seats I believe we might have to use it as some key (as bunch of requests can come for a particular seat)




so will that be similar to 2 phase commit the one done by task runners to get payment verification and other things






just a quick tip for excalidraw. you can actually use the number keys on your keyboard to select the shapes. a bit quicker than moving the mouse around



jamboard for google I think








I think I missed the beginning. event data is not super high volume right? so the ticket db is not sharded? if not, is a single node enough to handle all the pre-cut seats?

+1 did not get much from precut approach. is it single node, how data flow looks like?



And what is purpose of XOR trick? to know if seat is booked or not?




why store in local db , how to handle real time events









if we shard by user id, how do we handle unregistered (guest) users? I think we have to shard by hashes of request id right?
drop the user_id, require email address in their payment info submission form


but why would booking be allowed in the first place if we are not logged in?
(OR, we could require user is logged in, and then user_id is perfectly fine for partitioning key)





so is it like we are in between of our transaction and someone else books the same seat will be get some different seat inturn or will our transaction not succeed?
Yes.





should we not shard based on the UserID + seatID ?
Or any other combination but combining the SeatID too?






When would we use sharding_keys for database and how is it used for this in this scenario? (I assume something matching inventory with order?)

For more resources about understanding partitioning as a concept:
https://github.com/systemdesignfightclub/SDFC/




in india there is similar platform named bookmyshow. It uses website only and shows seating distribution. In that case I think for booking we would need transactions with some enum mentioning seat is 



wait. actually will user_id/request_id shards work? it is possible for 2 users of the same event to land on 2 separate shards even though they are trying to book the same event right?


I think we might have to shard by event id
Taylor Swift concert example -- definitely do not want to do that

but a particular event can become very hot thus would that be good?


it should be fine. for the largest events, they each can own a whole partition all to that particular event itself. that's how Shopify handles their peak shopping days

So, the issue is when multiple full machines need dedicated to one single "hot key"








so event+seat should work I believe?
seat+event should work, just make sure it finds partition by seat *before* event narrowing down the partition




Are we going to use Postgres (with multiple shards) or Cassandra (as this is very write heavy)?
cassandra actually definitely won't work, postgreSQL in a linearizable setting








https://www.postgresql.org/docs/current/transaction-iso.html

"Read Committed is the default isolation level in PostgreSQL. When a transaction uses this isolation level, a SELECT query (without a FOR UPDATE/SHARE clause) sees only data committed before the query began; it never sees either uncommitted data or changes committed during query execution by concurrent transactions"

















