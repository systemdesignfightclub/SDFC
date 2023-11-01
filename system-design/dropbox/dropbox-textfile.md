


DROPBOX



RESOURCES
- Grokking The System Design Interview -> Dropbox
- Alex Xu Volume II -> S3-like object storage
- Advanced Grokking -> GFS
- Advanced Grokking -> HDFS
- Google File System whitepaper
  - https://research.google/pubs/pub51/
  - https://stephenholiday.com/notes/gfs/
- GFS: Evolution on Fast-Forward
  - https://queue.acm.org/detail.cfm?id=1594206
  - https://stephenholiday.com/notes/gfs-case-study/
- Erasure Coding in Windows Azure Storage
  - https://research.microsoft.com/pubs/179583/LRC12-cheng%20webpage.pdf
  - https://stephenholiday.com/notes/azure-erasure/
- XORing Elephants: Novel Erasure Codes for Big Data
  - https://stephenholiday.com/notes/xorbas/
- Finding a needle in Haystack: Facebook’s photo storage
  - https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Beaver.pdf
  - https://stephenholiday.com/notes/haystack/
- The Hadoop Distributed File System
  - https://storageconference.us/2010/Papers/MSST/Shvachko.pdf
- Distributed File Systems: A Survey
  - https://ijcsit.com/docs/Volume%205/vol5issue03/ijcsit20140503234.pdf
- https://en.wikipedia.org/wiki/Erasure_code
- Dynamo whitepaper
  - https://stephenholiday.com/notes/dynamo/
  - https://notes.stephenholiday.com/Dynamo.pdf








REQUIREMENTS
- Users can upload files
- Users can download files


















CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)













--------------------
LIVE DISCUSSION QUESTIONS:








Looks like the taskrunner is more as the broker between the real consumer(backend service on the right) and provider on the left. Then task runner also needs to track the offset and handle retry ?

Yes, the task runner does track the offset :)







Can you post the link for the blog






The upload service may need to check the remaining storage size i think. And this can also be interesting for group account while multiple users uploading to the same drive, like read-modify pattern ,

- GFS does "lazy allocation" (?) (64MB)





What if download or upload stops after 99%
\---
you would still need to clean it up :(





dropbox chunks the files on uploads - 2048Bytes/chunk (may be that is what you call multi part upload ) ?




Db triggers will be a bottleneck. Sorry joined late





Who will do the chunking ? I guess there are client side components which are actively monitoring the folders for all the updates or changes happening in the files and uploading the delta changes
\---
In GFS, it's the "master node" that figures out where the chunks are





dropbox also does deduplication server side









Also bittorrent is another approach. Seed Metadata servers has mapping of a list of block could be in some of its p2p seed nodes.

Where the person downloading know how many blocks are still remaining to be downloaded
\---
GFS's name server would tell you up front








What about versioning?
\---
that's only for S3 (dynamo protocol whitepaper)

I think GFS & HDFS actually support append(), while S3 does not





They use a Non cryptographic hash for checksum



but the master in gfs is a bottleneck







A GFS Cluster is made up of a single master and multiple chunkservers





deduplication of chunks



Not more. It’s now Collusus right? Which is GFS 2.0?




What about synchronization across devices ?

What kinds of db we can use for Metadata DB?



dropbox uses mysql
for metadata db






If download and upload stops at 99%, restarting from last chunk will avoid complete restart but client needs to built the checkpointing logic i assume

- TCP has that logic built-in
- P2P might do something fancy for this




If two users edit the same file, in case of collaboration on same documents, how it will be handled here?
\---
With S3, each edit is a new version
Google Docs has a CRDT called "Operational Transformation"
GFS somehow supports BigTable being built on top of it







Dropbox actually does use 2PC internally _somewhere_
->

2PC is for updating records in its metadata mysql db I believe.

https://dropbox.tech/infrastructure/cross-shard-transactions-at-10-million-requests-per-second







Although we can simply use the file_name as the functional dedup indicator... 





What about synchronization across devices ?

One user may have several clients










that's a multi leader replication problem - usually solved with LWW conflict resolution (kinda like apple calender)











Few online resources say Dropbox has something called Synchronization Service

https://medium.com/@narengowda/system-design-dropbox-or-google-drive-8fd5da0ce55b

https://medium.com/@anuupadhyay1994/design-dropbox-a-system-design-interview-question-6b58b528214

https://stackoverflow.com/questions/11338032/architecture-used-by-dropbox




Probably this is the original source paper from where everyone copied
->
"Designing a Dropbox-like File Storage Service"
















