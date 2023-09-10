


LINK SHORTENER



RESOURCES
- Alex Xu Volume I
- Most books probably cover this





REQUIREMENTS
- User can provide a link and receive a shortened link back
- Users visiting the shortened link can get redirected to the full link website





VARIATIONS:
- bulk upload
- maybe link click statistics (visitor count) -- let's maybe leave this out of scope, it's basically the ad click counter/aggregator problem




















CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)









--------------------
LIVE DISCUSSION QUESTIONS:






This is my fav classic problem, usually I have seen hashing being used to give the shorten url but would love to see how you deal with when there is huge traffic.






I don't know if hashing indeed is the only scalable approach for this problem but yeah just wanted to mention this




we can use cqrs also here..separate read and write path







do we need authenticated user here what should be accessible only to logged in users or what not






can multiple URL generate Same UUID as that is created based in hashing?





Are there any hosted UUID service?
\---
I think zookeeper might be able to be used as a sequencer
(machine ID, timestamp/monotonic counter)








Do we need to compare UUID/hashing algorithms and dive into details during interview? Like Base64 in Alex Xu's book
\---
it's usually not what's covered in interviews





is a partition key the same as a composite key/clustered index?
\----

Primary Key = Partitioning Key + (Optional) Sort Key
Partitioning Key = what machine
Sort Key = make it uniquely identifiable, sorts within a specific machine

composite key = key composed of multiple attributed
= concatenated key?

clustered index = include an extra attribute in an index?

For more info,
"Storage Retrieval" chapter in DDIA
Database Internals by Petrov









For "part of the partitioning key", do we need to add a separate field with concatnated string in the database? 
\---
You do not have to explicitly make a new field with the other attributed concatenated together


{
    username: YubiDesu    --- primary key
    b: 1234
    c: 'Hi, I'm YubiDesu'
    d: 'https://wikipedia.com/system_design'
    role: STANDARD/PREMIUM/MODERATOR/ADMIN    --- secondary index
}






For the naive approach, what if writing to link DB fails but write to link cache succeeds?
\----
you'd return a 50x




how to handle conflicts in uuid ?
\-----
(machine ID, timestamp/monotonic counter) shouldn't result in a write conflict







Is it also possible to just write to DB? Then rely on cache-aside
\----
I'm fairly certain approach 3 is "cache aside"





do we need user in db schema ? and also don't we need timestamp also
\---

{
    UUID: 'defabc',          --     partitioning key, primary key
    long_URL: https://wikipedia.com/system_design
    short_URL: https://tinyurl.com/defabc
    user_ID: yubidesu.           (-- possibly part of partitioning key)
    timestamp: 1670005000
    redirect_counter: 230
}







mentioned how do we handle conflicts in case of millions or more of user trying to shorten url's (is it just hashing that would resolve this issue?)
\----
(machine ID, timestamp/monotonic counter) shouldn't result in a write conflict




just to get details of created and updated time and for expiring the shortURL
\---
you definitely need a timestamp for calculating TTL




How to handle duplicate URL request from user? User has submitted url and resubmits after few weeks later for short?
\----
I think it's  valid use case to allow users to create multiple unique short links

https://wikipedia.com/system_design

https://notion.com







how to make this service global ? what changes we need to make?
\---
copy-paste the whole thing to multiple regions, such as:
us-west-2
asia copy
europe copy
south america copy






write has to check cache and db the short url for the duplicate long exists and would return error






lazy loaded cache
LFU stuff




In approach 4, if write to DB fails, what will user see?
\---
you would retry the write with the task runner
if it keeps failing, your engineers should get paged

"exponential backoff with jittering"
"retry storms"
"thundering herd"






are we also having any limits in terms of size of the shortened url?
\---
URLs have a native limit for length




each region has different db?
\---
Yes :)

caching -> it might actually make sense to put a read replica of each region into all the other regions






I know you have mentioned that sequencer would handle things for write conflicts but if we are having our services in multiple regions how do we make sure that each of them is returning unique short urls, we would need some co-ordination right?
\----
The infrastructure provisioning would be linearizable

total order broadcast,
"global total ordering" is when you need linearizability

Lamport Timestamps
Vector Clocks
Logical Clocks


All we need here is a "total ordering that violates causality", and the thing you can use for causal consistency provides












in the global setup does it make sense one global db for single source of truth and have the rest of setup copied for multiple regions
\---
you have regional sources of truth









You can implement that in your service, right? if it doesnt exist in the cache, then the request goes to DB, loads into cache and return it.
\----
If you can avoid implementing something in a service, it'd cut down on hops








Can we download your excalidraw library?
\-------
it'll be in #read-me in discord
and in the description for this video








Lazy loading cache strategy is same as Cache Aside right?
\-------------

Approach 3 is cache aside with eager loading

Approach 6 is cache aside with lazy loading

I don't know if redis supports a lazy loading strategy without cache aside (such as what's in approach 4 -- pull-based)









what about application servers update the DB and push the updates into PubSub to populate cache?
\---
(drawn in approach 11)

I think this would make a lot of sense for the globally distributed read replicas






Haven't used Flink before, what is its responsibility here? Rank by frequency? 
\----
Flink is a "real-time analytics" solution
Min Count Sketch
OLAP is really expensive






For LFU approach why not just update the count directly from the service instead of using the queue
\------
Flink can do a rolling window counter






how do you handle CORS since you're redirecting the requests?
\----
Cross-Origin ____??
you can do programmatic manipulation of the HTTP headers by looking at the target URL







Are you planning to use JWT tokens to auth users?Also, when user enters the password for signing up into your service, do they Sha256 on client side and send the cryptographic cache to the server?
\---------
HTTPS -- with SSL you shouldn't really have a man-in-the-middle attack
I think with passwords, you usually send just the hash?
with storing the password, it should definitely be the hash with "salt" attached though








Can we say its an eventually consistent system
\-----

*Can* support "strong read consistency":
1, 2, 3, 5, 6, 7, 8, 9, 10


"Eventual consistency"-only approaches:
4


11 is strong read consistency for "cache", but eventual consistency only for "read replicas"







how can we request new problems to be discussed interested to discuss-> "Design a system for displaying adverts next to a search results, based on keyword and a list of different advertisement offer







































