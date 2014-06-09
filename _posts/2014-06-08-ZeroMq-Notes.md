---           
layout: post
title: Notes from Architecture of Open Source Applications - ZeroMq
date: 2014-06-08 
updated: 2014-06-08
comments: false
categories: Architecture notes
readmore: true
---

This morning on Reddit, I came across a discussion about RabbitMQ [post written by Aphyr](http://aphyr.com/posts/315-call-me-maybe-rabbitmq). So, I thought I would first get to know AMQP better before trying to understand what the post is about. So here is my notes on AMQP, some from the Architecture of Open Source Applications, Vol 2 [last chapter on zeromq](http://aosabook.org/en/zeromq.html) and some the documentation itself.

#### Important points
- ZeroMQ was originally conceived as ultra-fast messaging system for stock trading
- Concerns during design
	- Server in the middle will induce latencies and at somepoint it becomes a bottleneck
	- Cross-organization deployments cannot really have a central authority managing the whole messaging system
		- So usually, there shall be two messaging systems with bridges in between making the whole scenario fragmented.
	- *How to make messaging work without a central server?*
	- Smart endpoint - dumb network => ZeroMQ is a library
- Not having a central server to install & maintain is very practical
- A library approach over a server/application is preferred - consider it when building a new project
- Having global state in libraries is bad design as problems will arise when the library is instaiated twice for different purposes
- ZeroMQ doesnt have any global state, it is upto the consumer to create a global state if he needs one
- Performance criteria for messaging systems 
	- Throughput : no. of messages/sec
		- is for a set of messages, not one message
		- measured at a single point (sender or receiver, both arent the same)
	- Latency : how long it takes for a message to go from A to B
		- measured between two points in a system
		- no such thing as a latency of a stream of messages
	- Literature on Queuing theory will explain the relationship between latency and throughput
- So what affects performance?
	- Number of memory allocations
		- Best performance is achieved when you balance the cost of message allocation & cost of message copying
		- Copying is efficient for small messages
		- Zero Copying (allocate the whole message & return a pointer) is better for large messages
		- 0MQ messages are represented by opaque handle (for small message, handle has the message, for large mesg, it has pointer to message buffer)
		- Lesson: Dont assume there is a single solution for performance, it usually may have subclass of problems.	
	- Number of system calls
		- batching messages will help reduce the systems calls => better performance
		- but batching introduces latency (eg: Nagle's algorithm on TCP)
		- 0MQ strategy:
			- batching is turned off when message flow is sparse
			- if flow is high, then batch => increased latency. So might as well batch aggressively to atleast improve overall throughput
			- batch only at the top level
		- Lesson: for optimal throughput, turn-off batching at all lower levels of the stack & batch at the topmost level. Batch only when data arrives faster than it can be processed.
	- Concurrency Model
		- Locks are avoided and threads are run at full-speed
		- communication between threads is via messages
		- Classic Actor Model :)
		- One worker thread per core (having multiple threads will result in context switching)
		- each internal 0mq object is tied to a worker thread
			- hence no locks
			- no migration of objects to different cores
		- Theres a need to share worker threads among many objects - hence a need for scheduler along with other things such as
			- objects cannot block => system should be fully asynchronous as a blocking object will block itself as well as others
			- all objects have to become state machines
			- interactions between these state machines should be taken care of
			- handling shut-down of a fully asynchronous system is very complex (prone to race conditions, leaks)
		- Lesson: consider actor model for extreme performance & scalability
		- Lesson: think from the beginning on shutdown mechanics
- Code that executes all the time is critical path & optimization should focus on that
- Good design : each object is owned by exactly a single parent
- Two kinds of async objects in 0MQ
	- connection management objects that are not involved in message passing such as TCP Connection Listener which creates engine/session for each incoming connection or a TCP Connector which creates engine/session on a successful connection to a TCP peer. (also takes care of reconnections)
	- Data Transfer handling objects - Session & Engine
		- Session is responsible for interacting with a 0MQ object
		- Engine is responsible for communication with the network
		(diff engine based on protocol supported)
		- Sessions are exchanging messages with sockets in two directions and each direction is handled by a pipe object
	- There is also a context object shared by all sockets & async objects.
- User interacts with sockets.
- Lock-free algorithms
	 - these does not use kernel synchronization primitives for coordination but uses atomic CPU operations such as Compare-and-Swap (CAS)
	 - 0MQ uses lock-free queues
	 - each queue has exactly one writer thread & one reader thread. For 1-to-N, multiple queues are created
	 - CAS operation for each read/write is still not efficient enough
	 - Solution is to batch again
	 	- accumulate messages in a pre-write portion of the queue; as it is accessed by only the writer thread
		- similarly for reading, preload into a pre-read buffer
	 - Lesson: use existing lock-free algorithms instead of trying to invent one as they are extremely hard to debug. For extreme performance, lock-free algorithms aren't enough, you may have to do something smart on top of them such as batching
- API
	- Was originally based on AMQP
	- Later rewritten to conform to BSD Sockets API which made the adoption soar
	- Lesson: have a clean api based on what you want your project to be. reuse existing & proven ideas to avoid technical pitfalls
- MQTT is a good protocol for "distributing messages" & is easy to use
- 0MQ splits the messaging landscape into messaging patterns such as pub/sub, request/reply, etc. Each pattern is orthogonol to others.
- How to provide a solution that is as specific as possible, yet with wide range of functionality?
	- Define a layer of stack to deal with a particular problem area
	- provide multiple implementations of the layer
- having orthogonol implementations (eg: tcp endpoint cannot talk to udp endpoint) will allow future extensibility

In short, this is yet another excellent article. I have looked into the ZeroMQ guide and it looks excellent study for the students of distributed systems. I am looking forward to read through the guide and get to know messaging systems better.

Sidenote: we use in-house built messaging systems extensively at work & i can relate to a lot of problems that are mentioned in this article (technical pitfalls) that we find ourselves in.
