---           
layout: post
title: Notes from Architecture of Open Source Applications - Scalable Web Architecture & Distributed Systems
date: 2014-06-12
updated: 2014-06-12
comments: false
categories: architecture notes, reading notes
readmore: true
---

In this post, I capture some notes from reading the Scalable Web Architecture & Distributed Systems article on the [Architecture Of Open Source Applications - Volume 2](http://aosabook.org/en/distsys.html). The goal of this particular chapter was to cover the key issues to consider when designing large websites, as well as some of the building blocks used for this purpose.

####Important points
- Taking the time to plan ahead when building a web service will help in the long run.
- Key principles that influence the design of large scale websites
	- Availability (solved by redundancy, rapid recovery from failures)
	- Performance (fast responses, low latency : Performance is a feature)
	- Reliability
	- Scalability 
	- Manageability (scalability of operations)
	- Cost (hardware, software costs)
- Achieving one of the above may be at the cost of the other - they are at odds
- Acknowledge in your design which principles you are willing to sacrifice
- System architecture considerations
	- What are the pieces?
	- How do they fit together?
	- What are the tradeoffs?
	- Scalability concerns as a forethought - just consider it during the design
- SOA like services will help in scalabilty
- Plan for some sort of benchmark for reads & writes. May be splitting them will be required
- Splitting reads & writes will allow us to scale each indepedently, differently (reminds me of CQRS) & provides isolation as well
- Flickr does db sharding where each [shard services a set of users](http://www.scribd.com/doc/2592098/DVPmysqlucFederation-at-Flickr-Doing-Billions-of-Queries-Per-Day). Has its pros & cons
- No right answer but 
	- Consider the key principles
	- Identify the system needs
	- benchmark various alternatives
	- understand how the system may fail
	- Have a failure plan
- Redundancy is required for gracefully handling failures
- Redundancy applies to services as well
- Redundancy removes single points of failure & provides a backup plan
- The key to service redundancy is shared-nothing architecture - each should be able to run on its own
- Copies are stored in various geolocations as a natural catastrophy can wipe åll the copies at once.
- Load balancers are great tools to allow all the instances service requests simultaneously
- Scaling vertically - adding more resources (CPU, Memory, etc) to the same server
- Scaling horizontally - adding more nodes & splitting the load across the nodes
- Recommended to have Horizontal Scalability should be an intrinsic design principle of system architecture
- One of the techniques in hor. scaling is breaking your services into partitions/shards. For example: paying vs non-paying users
- Distributing data across services has its own challenges as well - data locality
- Another problem is inconsistency (race conditions when r/w from same source)
- Paritioning data by data/load/usage-patterns, etc 
- A [blog post] (http://katemats.com/distributed-systems-basics-handling-failure-fault-tolerance-and-monitoring/)  on failures by the author
- Another suggested read : [Pathologies of big data](http://queue.acm.org/detail.cfm?id=1563874)
- Caches : locality of reference (recently used data will be accessed agaiGlbn)
- Caches are used at all layers in architecture
- [frontend] -- [req node] -- [database]
- if cache : [frontend] -- [req node [cache]] -- [database]
	- When you have load balancers, the same request can go to different req nodes
	- Solution is to use Global Cache & Distributed Caches
- Global cache : a new server that acts as cache & that which is faster than your original database - this is used by all req nodes
- Global cache can manage the retrievel & eviction policy itself or can leave the responsibility of retrieval to place into the cache to req nodes
- Distributed cache - each node owns part of the cache
	- typically cache is divided up using consistent hashing schemes
	- request will first go to other node before going to the source
	- one problem : missing nodes (remedy : redundancy)
- Memcached is a popular cache (loca/distributed)
- Another read: Facebook article on [scaling memcached](https://www.facebook.com/note.php?note_id=39391378919)
- Proxies are also helpful to coordinate requests from multiple servers
- Proxies can collapse same/similar requests into one, make a single call and return the result back to the callers - Collapsed Forwarding
- Proxies are very logical place for caching
- Generally better to push cache infront of the proxies
- Suggested proxies: Squid & Varnish
- Index : more storage, slower writes but results in faster reads
- Berkeley Dbs, tree like data stsructures are commonly used to store indexes
- Often, there are layers of index
- Load balancers are principle part of any architecture
- Main purpose is to handle requests and route to one of the nodes
- Load balancer has different strategies (random, round robin, etc) to pick the node that serves the request
- Load balancers can be in Hardware or Software (like HAProxy)
- typically in-front of the distributed system
- reverse proxies : load balancers can route req different based on the type of request
- challenge: managing user-specific session data
- sol#1 : route the user always to the same node
	 - but this prevents us from benefitting from redundancy/fail-over
- Effective management of writes is important for scaling
- Queues are fundamental techniques that allows one to maintain QoS
- Queues allow clients to working in asychronous fashion

My personal opinion : this is definitely a good article that covers some simple but practical concepts of distributed systems with image service as an example. But as the author mentions in the end note, this barely scratched the surface and to be honest, most of the concepts listed is Distributed Systems 101. Nevertheless, it is a quick read, all accessible in a single place.
	
