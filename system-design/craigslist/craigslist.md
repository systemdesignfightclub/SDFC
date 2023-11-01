


CRAIGSLIST



RESOURCES
- "Acing the System Design Interview" by Zhiyong Tan (not published -- DO NOT pre-order, they keep bumping the date!)




REQUIREMENTS
- Users (sellers) can post a listing
- Users (viewers) can view listings (multiple)
- Users can view a specific listing (singular) details







APPROACHES:
- load balancers and reverse proxies to compare:
    - nginx ("Engine X")
    - apache web server
    - HAProxy
    - Traefik
    - ELB (AWS)
- other serving components:
    - S3
    - CDN
- other approaches:
    - DynamoDB w/ partitioning key
    - read replicas




next:

how query looks for listing details call

S3 as your server

CDN




CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)













--------------------
LIVE DISCUSSION QUESTIONS:





Hi there, just one query is the problem different than let's say a blog post design problem or let's say a twitter post design problem?







https://seattle.craigslist.org/see/atq/d/seattle-limbert-craftsman-oak-writing/7670014475.html






Probably we can focus on SEO as well. This is very relevant to this problem




is that browset - user doing thing something that comes built in with excalidraw?
\----
https://github.com/systemdesignfightclub/SDFC






is it important to mention they're pure HTML?
\---
it is kinda important, yes :)




I think we should keep index at the category_id also right? coz initially we'll be viewing all the listings based on the category
\----
completely correct :)








can we use that library in interviews?
\---
I have no idea -- you should ask your interviewer

is it tied to your account?
\---
there are no excalidraw "accounts", I think

I'm 100% okay with your guys using my library though







why PostgreSQL? Why DynamoDB?
\---
you don't need to overscale if you don't have to




Maybe you can take this at the end, I know this is a read heavy problem thus we can put cache to deal with hot partition issue but how do you deal it when it is write heavy use case?




Here LB storage might become bottleneck. We definitely need purging strategy in order to keep LB resourceful


10kB


100 listings = 1MB

1TB of storage


100,000 listings = 1GB

100,000,000 listings = 1 TB

365 days for the above

300,000 listings per day

3 listings per second







can you emphasize a little bit more on the static files aspect of your approaches?





SEO makes this problem really interesting. You need to have cron jobs to push new/updated pages to Google Analytics in order to start looking new page in search results. Its very niche area though
\----
the static files are *very* genious for improving page load times for SEO




sorry why do we have synchronous read replica?
\---
redundancy, durability

99.99
.01

(.01)^2
is the new downtime










harvard.facebook.com

yale.facebook.com





I usually think of saying we can use something like Lucene for dealing with Search thing
\---
that's for string search

if you did support string search, yes
Lucene / ElasticSearch would be the way to go :)



could you quickly explain how the listings DB read replicas to solve the problem of high read volume?




cant one of the read replicas be promoted to master instead of a synch replica?
\---
works for asynchronous replicas and dynamoDB, but not for synchronous replicas




I think if we are using multiple DB for writes we should go with a learderless or multi-leader DB. DynamoDB is a single leader thus I don't think would make sense here but pls corrrect me
\---
leaderless DBs would probably work fine here





In one of the my prev company (similar problem) searching and filtering are powered through Elastic Search and actual page hosted through CDN. ES is only missing piece here






ohh thank you for answering the write heavy hot partition issue, thanks it is quite good idea of adding random id


















































