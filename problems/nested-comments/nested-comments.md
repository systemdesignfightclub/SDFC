


NESTED COMMENTS



RESOURCES
- Taken from here: https://www.teamblind.com/post/System-design-questions-4CnNOVgF






PROMPT
```
Design a feature for nested comments
```







REQUIREMENTS
- Users can add a comment
- Users can view the comments of a given thread
- 100k+ TPS reads and writes




VARIATIONS
1. "Tree" of comments
2. DAG of comments











CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)













--------------------
LIVE DISCUSSION QUESTIONS:









{
    thread_id: "defabc-345-abcabc"
    comment_id: "defabc-123-defabc"
    username: "yubidesu"
    comment: "text goes here"
}







naive approach 1 (1 thread = 1 record):
{
    thread_id: 34565000
    username: "yubidesu"
    comment: "text goes here"
    replies: [{
        username: "SDFC"
        comment: "more text from the reply"
        replies: [{
            comment: ""
        }]
    }, {
        username: "SDFC"
        comment: "more text from the reply"
    }]
}

naive approach 2 (track child IDs):
(you have to update the parent ID)
{
    thread_id: 34565000
    comment_id: "defabc-123-defabc"
    children: ["", "", ""]
    username: "yubidesu"
    comment: "text goes here"
}


naive approach 3 (graph DBs)

node
{
    comment: ""
}

vertex
{
    parent: ""
    child: ""
}

node
{
    comment: ""
}







preferred approach (track the parent ID):
{
    thread_id: 34565000                  -- partitioning key
    comment_id: "defabc-123-defabc"
    parent_id: "abcabc-123-abcabc"
    username: "yubidesu"
    comment: "text goes here"
}






{
    thread_id: 34565000                  -- partitioning key
    comment_id: "comm1_comm2_comm3_comm4"    -- inverted index
    parent_id: "abcabc-123-abcabc"
    username: "yubidesu"
    comment: "text goes here"
}


/thread/34565000/comment/comm1_comm2_comm3_

SELECT *
FROM table
WHERE thread_id = 34565000
AND comment_id LIKE 'comm1_comm2_comm3_%'

^ can alleviate I/O bottlenecks






DAG:
- list of parent IDs
- make join table

{
    thread_id: 34565000                  -- partitioning key
    comment_id: "defabc-123-defabc"
    parent_id: ["abcabc-123-abcabc", "", ""]
    username: "yubidesu"
    comment: "text goes here"
}




comments table
{
    thread_id: 34565000                  -- partitioning key
    comment_id: "defabc-123-defabc"      -- sort key
    username: "yubidesu"
    comment: "text goes here"
}

join table:
{
    thread_id: 34565000
    parent_id: "defabc-123-defabc"
    child_id: "abcabc-123-abcabc"
}











I never knew that graph dbs didn't scale and are reserved for olap per that one comment. not sure why though. TIL



sir how to clear machine coding round
\---
Machine Learning System Design books:
- Chip Huyen's
- Alex Xu's ML system design
- mmds.org
- Khang Pham's book





LLD interviews
\---
Effective Java by Joshua Bloch







Sir, I am back end java developer, is it enough Alex Xu's vol-1 and vol-2 to clear interviews?
\---
that's great for FAANG mid-level interviews and non-FAANG senior

for FAANG senior roles,
you will at least need DDIA as well






will graph DBs scale well with low latency?
\---
it *will* have high latency if you're using it right





what about L6/L7 SDMs for FAANG? DDIA needed?





I've heard that there are parts of DDIA that are already dated, such as it being content heavy on the direct use of map reduce instead of abstractions on top of it. if you agree, do you have any
\----
I think it's actually aged fairly well



suggestions for supplemental reading?




I solved more than 700 leetcode problems but can’t crack FAANG





is thread_id specific post ID? If yes, we'd be doing scatter gather while fetching all comments

1000s of threads, minimum
100 machines max DB
10 threads per machine




list of parent ids will be O(N) for read
K log N




I think inverted index for every comment with parent ids probably will be best for READ as reads will be higher, so I would optimize for READ than WRITE




























































































