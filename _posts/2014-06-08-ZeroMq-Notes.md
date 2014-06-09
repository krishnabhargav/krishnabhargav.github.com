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
		- Theres a need to share worker threads among many objects
- Code that executes all the time is critical path & optimization should focus on that
