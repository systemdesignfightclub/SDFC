


STOCK EXCHANGE



RESOURCES
- Alex Xu Volume II
- A video by jane street
    - "How to Build an Exchange" -- https://www.youtube.com/watch?v=b1e4t2k2KJY





REQUIREMENTS
- Order placement, matching, and execution
- the price feed
- broker stuff is out of scope





VARIATIONS:
- sometimes the focus is on the price feed
- sometimes the focus is on the broker portion








Approaches:

- sharding by ticker, sidecar deployed redis for the wallet and gossiping w/ CRDT
- one very, very beefy mainframe









CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)









--------------------
LIVE DISCUSSION QUESTIONS:






i think Alex Xu's book actually has a very interesting take on this problem. i thought it was legit
YES.




So are we going to deal with many concurrent buying and selling transactions taking place at scale!!
Correct :)






yeah. robinhood isn't a market maker. they actually just pipe the order to a real MM that perform the trades





Little surprised that it is not using websocket for live feed
\---
we can't build on top of TCP because we care about speed, and handshakes for TCP add a lot of latency






machine sizing might be interesting to discuss here since we are sticking everything into a single node. also LLD might be interesting to discuss disk IO in regards to latency 




I read there's Active-Shadow replicator. Both of them do same computation in matching engine. If active goes down, shadow takes over. Sharding by ticker is what SEBI follows in India









problem with sharding is that we'll introduce a network hop. that adds millisecond latency which might not be acceptable






stack overflow has machines with TB ram. I wonder if that'll be able to host everything in a single machine. I didn't do the math so not sure




agree with fact, network hop is going to add 2 digit ms latency. Need to check what kind of beafy machine would able to host everything on 1 host






my earlier comment around sharding is that the client needs to first connect to a router that knows for a stock, which machine the request needs to go to?
\---
the client could cache which ticker is on which IP address




we could hard-code the stock to machine map into the client. but that makes client updates tricky. an older client would not be able to trade any newly added stocks 
\---
you could just load it up at the start of day and cache it on the client



clients like robinhood/binance share socket connection between them and end user (app/webpage). Any idea why stock exchange does UDP multicast?
\---
"retail investors" (college kids on robinhood) vs "institutional investors" (citadel / two sigma / goldman sachs)


- college kids on robinhood (don't necessarily require low latency)
- HFTs (require low latency)
- market makers (require low latency)
- investment banks / hedge funds (don't necessarily require low latency)




we could do forced updates for the client. but that makes the UX pretty bad. and also what happens if the forced update fails?
\---
the cache has a TTL of one day









don't quote me on this. from first principle perspective, the order placement needs to happen over TCP since we need to ensure the request goes through. pricing updates can go through

(continued) can go through UDP since TCP requires ping/pong [round trip] that adds a lot of latency. and there's no point in trying to deliver an old price value when a new one has become available. so UDP wins

Got it. That helps! Don't have much background on UDP multicast. Will definitely check. Any resource you have on this? Thanks

you might not like this answer. wikipedia







Why is there an arrow between the redis exchange gateways for different machines?
\---
gossiping to make the leaderless replicas of sidecar deployed redis consistent with each other






How is the sequencer designed?
\---
Database Internals - Alex Petrov

Percolator Transactions -> leaderless databases use a "sequencer" component to outsource the challenge of total order broadcast when they want to implement serializable transactions







what kind of machines serve 100k TPS here?
\---
"bare metal" in EC2 instances
4XL instance only (the AMD ones that have \~100 cores on them)

5-6 "xlarge" -- we were only getting "slices" of a machine












































