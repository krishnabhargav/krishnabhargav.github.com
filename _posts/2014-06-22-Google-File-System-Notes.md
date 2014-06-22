---           
layout: post
title: Reading Notes - Google File System
date: 2014-06-22
updated: 2014-06-22
comments: false
categories: Architecture notes, Publication Notes
readmore: true
---

As part of my readings, I am going to make some important points here from the paper - [The Google File System](http://research.google.com/archive/gfs.html). Google File System is a scalable distributed file system for large distributed & data-intensive applications. This paper is the inspiration for HDFS, the file system that powers Hadoop.

#### Introduction
- Component Failures are the norm => failures are assumed to happen all the time & is part of the design. Therefore constant monitoring, error detection, fault tolerance & automatic recovery must be integral to the system.
- Files are huge by traditional standards.
- Files are mutated by appending new data.
- Co-designing applications and the file system API provides flexibiity.

#### Design : Assumptions
Lesson: Design should be guided by assumptions.

- Commodity components that often fail shall be used.
- System shall store modest number of large files.
- Workloads : Large Streaming Reads & Small Random Reads.
- Workloads : Large Sequential Writes that append data to files.
- Concurrency : Multiple clients can append to the same file concurrently.
- High sustained bandwidth is more important than low latency.

#### Design : Interface 
- Familiar file system API (but not standard API such as POSIX)
- Operations: Create, Delete, Open, Close, Read, Write, Snapshot, Record-Append
- Files are organized hierarchically in directories & identified by pathnames.

#### Design : Architecture
- GFS Cluster : Single Master & Multiple Chunk Servers
- Files are divided into fixed-size chunks (64-MB)
- Each chunk has a 64-bit globally unique handle assigned by the Master
- Chunk Servers stores chunks as linux files on local disks
- Chunk Servers can read/write a chunk specified by the handle & byte range
- For reliability, chunk is replicated on multiple chunk servers (3 by default)
- Master server 
	- Maintains all system metadata
		- Namespace information
		- Access Control information
		- Mappings from files to chunks
		- Current locations of the chunks
	- Controls system-wide activities
		- Chunk Lease Management
		- Garbage Collection of Orphaned Chunks
		- Chunk Migration between chunkservers
	- Periodically communicates with Chunk Servers in HeartBeat messages to give instructions & collect its state
- Clients interact with both master & chunk servers - master for metadata ops & chunk servers for data.
- Neither the client nor the chunk servers caches file data - not beneficial to cache huge files.
 
#### Design : Single Master
Question I first got when reading this - if you have a single master, does it not become a bottleneck? What if the master fails - how is reliability accomplished? 

- Single master simplifies design & provides flexibility
- Master can make sophisticated chunk placement & replication decisions using global knowledge
- Minimize reads & writes to it so that it does not become a bottleneck
	 - Clients never read & write data through the master
	 - Client asks the master which chunkservers it should contact 
	 - (input : filename & chunk index, output: chunk handle & location of replicas). Master can also include information about the following chunks as this extra information sidesteps several future interactions at practically no-cost
	 - Client caches this information (key:filename & chunkindex) for some duration
	 - Client interacts with chunk servers directly for many subsequent operations
- How is reliability accomplished? (covered later in Fault Tolerance section)
	- Master & Chunk server are designed to restore their state & start within seconds. 
	- Master State is replicated for reliability - operation log & checkpoints are replicated on multiple machines
	- Mutation of state is complete only if log record has been flushed to the disk locally and on all master replicas.
	- If master process fails, it can restart almost instantly
	- if the local disk fails, monitoring infrastructure outside GFS starts a master process elsewhere with replicated operation log.
	- clients uses only the canonical name of the master - a DNS alias
	- Shadow masters provide read-only access to file system even when the primary master is down (shadows != mirrors, information is lagged by a fraction of seconds behind)
	- Shadowing works because content is stored on chunk servers - no stale information is actually retrieved.
	- Shadow master reads a replica of growing opereation log, applies the same sequence of changes to its data structures exactly like the primary does.
	- It also polls the chunk servers at startup to locate chunk replicas

#### Design : Chunk Size
Chosen chunk size of 64 MB which is much larger than typical file system block size. The advantages of large chunks are:

- Reduces the clients need to talk to master because more reads & writes can be done to the chunk (since its big) with one intial request to master.
- Chunk location information for even large TB files can be conveniently cached (large chunks => less metadata, more space to cache)
- Network overhead can be reduced keeping  persistent connection to TCP over an extended period of time
- Reduces metadata stored on the master (which can allow storing metadata in memory)

The disadvantages with large chunks are:

- A small file consists of small number of chunks. So the chunk servers storing these files becomes hot spots if client are accessing the same file.

#### Design : Metadata
The master serve stores the metadata. There are three kinds of metadata stored:

1. File and Chunk Namespaces
	- Mutations to these are also logged in the **Operation Log**
	- Operation Log is stored locally and also replicated on remote machines
2. Mapping from Files to Chunks
3. Location of each chunk's replicas
	- Chunk location information is not persisted
	- Master asks each chunk server about its chunks at master startup & whenever a chunkserver joins the cluster
	- Master keeps the information up-to-date thereafter because it controls all chunk placement & monitors chunkserver status through HeartBeat messages
	- So no need to special sync between master-chunkserver when they join or leave the cluster

Metadata is stored in memory and the master server scans the state periodically in the background for

- Implementing chunk garbage collection
- Re-replication in the presence of chunkserver failures
- chunk migration to balance load & disk space

Since metadata is stored in memory, the capacity of the whole system is limited by how much memory the master server has. Also noted that master maintains less than 64 bytes of metadata per 64-MB chunk. (also for file namespace data is less than 64 bytes per file).

##### Design : Operation Log

- Operation Log contains historical record of critical metadata changes that is central to GFS. Serves as 
	- persistent record of metadata
	- logical timeline that defines the order of concurrent operations
- Files & chunks as well as their versions are all uniquely & eternally identified by the logical times at which they were created.
- Operation log is critical => hence information is to be stored reliable which means the changes are not made visible to clients until the metadata changes are made persistent. 
- Operation Log is replicated on multiple remote machines.
- Client operations are responded to only after the corresponding log records are flushed to the disk - locally & remotely.
- Master can be recovered by replaying the log
- To improve startup time, the log is kept small => done via checkpoints
- Checkpoints
	- When the log reaches a certain size, the entire state is stored
	- This forms a checkpoint
	- During recovery, the master "use" a checkpoint & then the log shall be replayed from that checkpoint onwards.
	- Checkpoint is in a B-tree like structure that can be directly mapped into memory and used for namespace lookup without extra parsing
	- This speeds up recovery and improves availability
	- Checkpoints take time so the internal state is structured in a way that a new checkpoint can be created without delaying incoming mutations
	- Master switches to a new log file and creates a new checkpoint in a separate thread.
	- The new checkpoint includes all mutations before the switch.
	- When completed, the checkpoint is written to disk both locally and remotely.
- Recovery needs only the latest complete checkpoint and subsequent log files. Older checkpoints & logs can freely be deleted.
- Failure during checkpointing does not affect correctness because the recovery code detects and skips incomplete checkpoints.

#### Design : Consistency 
GFS has relaxed consistency model that supports highly distributed applications well but remains relatively simple and efficient to implement. The guarantees that GFS provides are:

- File namespace mutations (file creation, etc) are atomic.
	- Handled completely by Master
	- Namespace locking provides atomicity and correctness
	- Operation log defines global total order of these operations
- State of a file region after data mutation depends on the 
	- type of mutation (writes or read appends)
	- whether it succeeds or fails
	- are there any concurrent mutations
- All mutations to a chunk are applied in the same order on all its replicas

This particular section from the paper is not so readable and is not easy to understand. So I took some [notes from here](http://pages.cs.wisc.edu/~thanhdo/qual-notes/fs/fs4-gfs.txt).

- A file region can be
	 - consistent: if all clients sees the same data regardless of which replica they read from
	 - defined: consistent & all writes intact
- A file region can also be consistent but undefined. Example: when a client writes big (> 64 MB) as it will be multiple write operations, writes may be interleaved and overwrittent
- Record-Append is idempotent => Append-at-least-once semantics. if it fails, try again and again

GFS applications can accomodate the relaxed consistency models by some simple techniques:

- Relying on appends rather than overwrites
	- Appending is far more efficient
	- More resilient to application failures
- Checkpointing
	- Checkpoints may also include application level checksums
	- Readers verify and process only the file region upto the last checkpoint
	- Allows writers to restart incrementally
	- Keeps readers from processing successfully written file data that is still incomplete from the application's perspective
- Writing self-validating, self-identifying records

Overall, the consistency model of GFS is not entirely clear to me and needs to be read again. 

#### System Interactions : Leases and Mutation Order
System is designed to minimize the master's involvement in all operations.

- Each mutation is performed at all the chunk's replicas
- **Leases** are used to maintain a consistent mutation order across replicas
- Master grants a chunk lease to one of the replicas - called the primary
- Primary picks a serial order for all mutations to the chunk - all replicas follow this order.
- Lease minimizes the management overhead on the master
- Lease has an initial timeout of 60 seconds. But primary can request and extend the lease from the master.
- Extension requests and grants are piggybacked on the HeartBeat messages.
- Master may sometimes try to remove a lease before it expires
- Master can grant a new lease to another replica after the old lease expires
- Data can be pushed by the client to all replicas in any order. Each chunkserver will store the data in an internal buffer until a write request is received.
- Client sends a write request to the primary which then assigns consecutive serial numbers to all mutations received, applies the mutation and then forwards the request to all replicas.
- Secondaries replies back to the primary indicating they completed the operation
- Primary replies to the client.
- In case of errors (write can succeed at primary & fail at subset of secondaries -> this still is an error), the client can retry the mutations.
- If the write is large or goes beyond a chunk boundary, the write is split into multiple write operations - which may be interleaved or overwritten by concurrent writes from other clients (consistent but undefined state).

#### System Interactions: Data Flow
- Flow of data is decoupled from the flow of control - improves network efficiency as expensive data flow can be scheduled based on the network topology regardless of which chunkserver is primary
- Control flows from primary to secondaries, data is pushed linearly in a pipelined fashion
- Goal is to fully utilize network bandwidth, 
	- DAta is pushed linearly along a chain of servers 
	- Each machine's full outbound bandwidth is used to transfer the data as fast as possible
- Goal is also to avoid network bottlenecks & high-latency links
	- For this, each machine forwards data to the closest machine (that hasnt received the data) in the network topology
	- Distance may be estimated from IP Addresses in simple network topologies
- Goal of minimizing the latency
	- Data transfer over TCP connections is pipelined
	- Once some data is received by a chunkserver, it starts forwarding immediately.
	
#### System Interactions: Atomic Record Appends
- Traditional Write : Client specifies data & offset. So concurrent writes to same region are not serializable. data may contain fragments from different write operations
- Record Append - Client only specifies the Data. GFS appends it to the file atleast once atomically at an offset of GFS's choosing & the offset is returned to the client
- If the record append exceeds the chunkboundary, the primary will pad to fill in the chunk & asks the replicas of the same. It will then ask the client to retry the operation.
- Record Append is restricted to be atmost 1/4th of max chunk size to keep worst-case fragmentation to minimum.
- If record append fails at any replica, the client retries the operation.
- GFS doesn't guarantee that replicas of same chunk are bytewise identical. The only guarantee is that data is written atleast once as an atomic unit.
- In terms of consistency guarantees
	- REgions in which successful record append operations have written their data is defined (and consistent)
	- Intervening regions are inconsistent (undefined)
	
#### System Interactions: Snapshot
- Snapshot operation makes a copy of a file or directory tree almost instantaneously while minimizing any interruptions of ongoing mutations
- Standard copy-on-write technique is used to implement snapshots. 







