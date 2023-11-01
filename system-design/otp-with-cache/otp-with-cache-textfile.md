


One Time Password ("OTP") with Cache



RESOURCES
- https://en.wikipedia.org/wiki/One-time_password






PROMPTS
```
Design an OTP with a cache

Company: Uber
Role: Senior (L5)
```







REQUIREMENTS
- User tries to authenticate
- OTP is provided to user
- User completes OTP
- User successfully performs operation or action that required OTP





VARIATIONS:
1. OTP is sent to primary device and entered into secondary device (Google does this with the Youtube app)
2. OTP is sent to the secondary device and entered into primary device





- DB schema for cache
- sidecar deployed OTP generation
- draw out the two options a bit more








CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)













--------------------
LIVE DISCUSSION QUESTIONS:



why do you need to cache the OTP?





I think backend needs to assign auth cookie as well
\---
we'll draw that out when we're doing option one




sidecar deployment implies library import and run locally?






So, cache stores values for each userName. How it manages to hold such large volumes of data? Can you talk about it in brief.


16GB of RAM for cache of one machine

5 minute TTL
300 seconds of data

how many requests per second can we handle?
how big are the requests? up to 500 char.s (500 bytes)

500 bytes * 300 seconds
150,000 bytes at 1TPS
150kB


16,000MB
16,000,000 kB

100,000 TPS can be handled

100,000 TPS * 300s * 500 bytes per request

150000 00 000

15,000,000,000 bytes
15,000,000 kB
15,000 MB
15 GB






'yubidesu' -> {
     one_time_password: "4567"
     generated_timestamp: 1670005000
     action: {
         action_type: "BUDGET_CHANGE"
         action_metadata: "$500"
     }
}








How are we invalidating the OTP ?
\---
We are using TTL of 5 minutes, and the OTPs are only good for one minute



What if Cache crashes what is fallback mechanism ?
\---
they have to re-generate the OTP (we are not fault tolerant)








how would you structure the action table schema and permission on those action ,do you have and book |source code for reference







What if a user keeps entering some wrong OTPs, do we invalidate the OTP after a few failed attempts?
\---
use a counter, and if you cross a threshold, do NOT flip the authenticated flag even if the correct OTP is supplied




Do they decide when to trigger OTP flow for anomaly activity? Or keep some value like 15 days and ask for OTP again?
\---
Let's maybe follow-up with a separate design for the risk checking of a credit card for suspicious purchase activity





I think websocket would be overdo just for handling single flow. Short polling would be fine
\---
I definitely agree with that






What about rate-limiting in case of flood of OTP attacks?
\---
the wrong_attempt_counter is sufficient for handling brute force issues





I think the wrong_attempt_counter is fine as long as the cache doesn't crash.

last_5_incorrect_login_attempts is actually tracking "OTP generations", while the counter is for within the context of one specific instance of a generated OTP








The excalidraw links that you have provided in the github doesn t open.




































