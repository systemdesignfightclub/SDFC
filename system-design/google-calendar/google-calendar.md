


GOOGLE CALENDAR



RESOURCES
- Alex Xu Volume I
    - Notification Service
- My own coverage (problems with similar content)
    - price drop tracker
    - distributed cron job






REQUIREMENTS
- create an event
    (- it could be a recurrence)
- send notification ahead of event
- event invites (multiple people can have a reference to the same event)
- see your current calendar for the current week





CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)






APPROACHES
1) pre-generate all the eventInstances for the series
Pros:
- avoid the daily/weekly batch job (might avoid some complexity)
Cons:
- it doesn't series updates very well, particularly if a lot of people are subscribed to the event (requires fan out with every update)

2) regularly check and generate the eventInstances weekly/daily (some big daily/weekly batch job)








--------------------
LIVE DISCUSSION QUESTIONS:



yeah. haven't seen this one before. good pick








If we do not pregenerate, how calendar UI can show all events when check in future?





we can store recurrence event in RRULE format i think goole also use the same



its a standrd rfc for storing recurring events




why would we generate "instances" of events rather than storing the frequency as RRULE and let the client deal with showing them in the UI? that doesn't seem very scalable 

sorry. let me clarify, neither pre-generate nor cronjob is scalable due to duplicate instances of events. why not have a single event definition that just has the frequency?

so what im suggesting is not to repeatedly inserting new records into the "event attendees" table for a recurring event. the client will see an event is recurring and just shows that accordingly



























































