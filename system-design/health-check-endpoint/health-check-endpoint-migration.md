


HEALTH CHECKING ENDPOINT MIGRATION



RESOURCES
- somewhat similar to "Weather App" problem that I've previously covered





REQUIREMENTS
- "Number of health checks should not overload the service instances."
- short term and long term approach
- service mesh as original design to work with








APPROACHES:
1) rate limiter
2) centralized health check service
    - "watchdog service"
    - https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/monitor-app-health#use-watchdogs (thanks, @sabre!)
3) proper independently scalable services with load balancers













CHECKLIST:
- [ ] load balancers
- [ ] CDNs?
- [ ] secondary indices?
- [ ] DB schemas
- [ ] partitioning keys, secondary indices
- [ ] NALSD numbers (return to this at end)













--------------------
LIVE DISCUSSION QUESTIONS:









start the discussion buy telling us why python streamlit is the best framework on earth









How does the inclusion of proxies changes/modifies the problem statement? Will it not be the same for a simple microservice architecture?






The 'Health Checkers' on the top is a cron job? Will it pull health data for all the services?






I did not understand what is/was the role of proxies. If someone can point me to this question document, I can read it more carefully, or if you could address it towards the end, that would be great.






ITs like deploying side car on  each machine.




Will the read replicas have real time health data ?
\---
it is eventually consistent
meaning you will get stale reads





so will this be a push happening to Health Checkers or would be a pull based model?




It can't be eventual. Thats the whole point of health service
\---
you would still have stale reads in the original design







No it will effect availability.  So it cant be a solution




so will Service B not talk with "Health Checkers" will it only send the response to Service A and then it will indirectly send to Health Checkers?




We talked about k8 in problem statement at start, If health check fails, k8 won't even schedule the pods so that can act as a reliable source? Or we are assuming it's not configured in the cluster? 







Why 404? It's a client error code, LB will never return 404. I will say they will return 5xx



Load balancer  also generally calls the local services health check endpoints to keep its self updated with active services.

/basicCheck
/deepCheck (actually calls DBs)






We have micrometer/prometheus setup for health checks/application metrics? Any thoughts on that?






If you are just deploying your service it wont respond for that period of time.
But how will the load balancer will know that deployment is complete.
So in order to achieve the load balancer pings all the services based on configured time and maintains list of active nodes.








you write that in your Kubernetes script to tell the loadbalancer to start checking in "periodseconds" after the application is up after "deploymentseconds"




Loadbalancer does low level network pings not call health check..



"there is no K8s here ..this is legacy system"







Actually it depends, LB can relay response from backend services along with their configured code. Otherwise they generally follow 5xx 




Does this apply in microservice or monolith can also apply this concept ?





FYI, This whole switching which you were  talking about around deployment. Its generally done vai traffic manager. And each stamp generally ahs its own load balancer.





4XX means service is up.
\---
404 might mean the machine is dead






Nope, they won't translate 404 to 5xx

They only do if there no response from backend.

I pointed 404 because, you said backend pod/node is down and drew a cross (so no response from backend?) that's why I suggested it might be 5xx in this case

404 is route not found. Machine is dead is 502 gateway timeout when LB tries to reach backend











https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/monitor-app-health#use-watchdogs







I meant green and blue will have its own load balancer.




















































