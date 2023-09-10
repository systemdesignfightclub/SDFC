


ONLINE PRESENCE INDICATOR



RESOURCES
- Alex Xu has some messaging app coverage
- Stanley Chiang has some *very* good message app coverage
- not much coverage specifically on the online presence indicator aspect though





REQUIREMENTS
- indicate whether the other users of a messaging app are currently online or not





VARIATIONS:
- 1-1 / small group (WhatsApp)
- large group (discord)





APPROACHES:
1) Direct send of the heartbeats directly over the redis pub-sub (WhatsApp, etc. when there's no actual message store)
2) user-group mapping included
3) variation approach 1, but with message data store
4) polling the online status (no websocket broadcast)
5) DB trigger that only broadcasts _changes_ of the online status, using redis TTL








CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)









--------------------
LIVE DISCUSSION QUESTIONS:







I always have one confusion regarding web socket request redirection, how load balancer does it, does it has direct connect from client to server and bypassing the load balancer, or load balancer get the request and forward it to server who is handling, in later case load balancer has to open a web socket connection from client and one from server or it works in L4 layer where it doesn’t know about web socket connect and just forward the packets ?
\---
"Envoy" for service discovery
It does skip past the load balancer after the persistent web socket connection is set up (unless you use a stateless service for the receive back-end service)



Does we need the heartbeat separately, we can not use the existing message that a client are sending to other users ( might be different in discord )
\---

I think the infrastructure for heartbeats and messaging could be re-used -- the heartbeats themselves could even be treated as message to some extent





Qq: Is User Machine mapping store need to be managed by backend services independently or it is taken care by API G/W for websockets?
\---
Yes, I do think the user-machine mapping store is typically handled within the service discovery component



Without data store, how do you know the group members to send to?
\---
Your phone itself would handle the fan-out







since redis supports push based messages, can't we just send the messages directly to the "Send Backend Service"?
Or is the intermediate "Send Task Runners" service required because the we need to use the "User Machine Mapping Store" before sending it to a "Send Backend Service"?
\---
You could do that mapping on either end for the case of WhatsApp (approach 1)
Yes for the 2nd question :)



Does this design imply that sending messages do not use websockets, but receiving messages do? Why not use websockets for sending messages as well?
\---
Yes, I did do a stateless design on the send side
It can definitely be done either way
stateless services can scale up and down much more easily





how does fan-out directly from the phone help/improve encryption?
\---
The service doesn't have to store info on the group members
You don't know what groups they're a member of



Without data store, how do you know the group members to send to?
\---
the phone itself stores that information



XMPP (WhatsApp)
vs
Torrenting software





Can you compare the pros/cons of Approach 2 over Approach 1 ?
\---
approach 2 enables "discoverability" of groups
while approach 1 has more privacy focus




what does the user mapping look like
\---
which one?





In approach 3 online status service will only be responsible to return the status first time ?
\---
Yes :)

After that it will be responsibility of fanout
\---
Yes :)






Is the heartbeat data store of Approach 3 a redis data store with a TTL of 5 mins?
\---
That does work, but I wasn't originally thinking of that





when will we go for polling (Approach 4) vs push (approach 3)
\---


polling
- makes sense when it's something very active and every poll is guaranteed to have messages, but you don't care for real-time updates
- "silent updates"

push-based
- either very slow and inactive messaging, or where the user would not care for real-time updates
- "non-silent notifications"








TTL DB Triggers:
https://stackoverflow.com/questions/59073206/redis-any-way-to-trigger-an-event-when-a-value-is-no-longer-being-actively-wri
"redis keyspace notifications"
https://redis.io/docs/manual/keyspace-notifications/









Why FE need signal for turning offline, can it not always does it if didn’t get any notification for online say in 5 minute ?































