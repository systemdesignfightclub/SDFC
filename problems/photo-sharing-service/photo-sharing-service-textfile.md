


PHOTO SHARING SERVICE



RESOURCES
- similar to dropbox, youtube, app store (I'm treating this as a "large file" problem)






PROMPTS
```
Problem Statement: We want to design a photo-sharing or photo-publishing website, something like flickr or Google Photos. Users can upload photos, download full-size photos, and also browse a web page/mobile UI with thumbnails of photos in a particular photo album. Design the software system that stores and retrieves the photos and thumbnails

Company: Google
Role: Senior (L5)
```






REQUIREMENTS
- Users can upload photos
- Users can download full-size photos
- Users can browse a web page/mobile UI with thumbnails of photos in a particular photo album
- Service handles storage and retrieval of the photos and thumbnails


















CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)













--------------------
LIVE DISCUSSION QUESTIONS:
















Hi one question in general, was reading some chat on discord, wanted to ask you here, can we use consistent hashing in SQL DB's? I have never seen it but just wanted to know







Perhaps we do not need to do Step 3. For whatever reason the upload fails, step 3 will be an orphan record.



please correct me if am wrong here, so user get's a pre-signed unique url for the image user is trying to upload.



and then it tries to store that in Metadata right? or before that is the step 3 doing anything?



will that s3_link which we are storing in metadata db be public ?



how about stream changes as events to metadatadb? and update the status accordingly?
\----
I don't know if S3 actually supports DB triggers




What does “pic” in “pic_uuid” field stand for?
pic = picture (photo)



S3 Event Notification can be helpful for stream changes in a bucket?







Sorry what is a presign url ?

A pre-signed URL (or pre-signed link) is a URL that provides temporary access to a specific resource or operation, usually on a cloud storage service like Amazon S3 or Google Cloud Storage. It allows you to grant temporary permissions to users or applications without them needing direct access to your credentials or authentication mechanisms. Pre-signed URLs are often used when you want to share private resources securely or enable controlled access to resources for a specific period.




S3 supprt triggers, i have invokes lambda based on put in s3 buckets






What will be schema for user specific time bound sharing?




So inside bucket, we will have directory for each user?
\----
I'd try to avoid that due to the potential for hot keys



I think S3 supports dynamic sizing of images. Client specifies the resolution of an image and S3 will resize the image on-demand and then stores it
\------
maybe don't rely on that in google interviews :)




Do we need to support multiple formats(jpeg, png)?
\----
it shouldn't really make any difference since the bucket treats each image as just a file





hi but pic_uuid is random right, so it would still not solve the hot partitioning issue incase any (due to randomly getting assigned to any node).
\----
"hot partition" vs "hot key"


yubidesu-1
yubidesu-2
yubidesu-3
jenner-1
jenner-2
jenner-3



partition_A
yubidesu-1
yubidesu-2
yubidesu-3

partition_B
jenner-1
jenner-2
jenner-3

partition_B
random-guy-1
random-guy-2
random-guy-3




partition_A
random-guy-1
jenner-2
yubidesu-3

partition_B
jenner-1
yubidesu-2
random-guy-3


partition_B
yubidesu-1
random-guy-2
jenner-3







This would work if we use ULID rather than UUID I believe
\----
what's a ULID?






S3 where original size pictures will be stored can scale automatically right. We need to take care of metadataDB by using consistent hashing. Am i correct?
\---
S3 has "buckets" of up to 10PB

pick a random bucket when getting a pre-signed URL









How about putting load balancer before upload service







The partition is baed on hashes of the UUID. So the randomness doesn't really matter here. It depends on the hashing function. ULID is like snowflake? Its goal is to support sorting?






How about having thumbnail creation job done asynchronously via queue using Kafka, In order to request thumbnail(s), the requester has to produce a message to the requested topic in Kafka.
To consume these thumbnail creation requests, we created a service “Thumbnail Consumer”, to process those requests, and spawn the Thumbnail maker





How about thumbnail maker populating CDN ?





ULID can be useful here as we can fetch the images in the album based on upload order





why doesn't album service not returning results from thumbnamil but let browser request to thumbnail store? (sorry if it's a dumb question
\----
the bandwidth would otherwise likely be a scaling bottleneck for the service



10kB / thumbnail

10 thumbnails

100kB of thumbnails images to a page load


































