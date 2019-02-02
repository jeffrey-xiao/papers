---
geometry: margin=3cm
title: Notes for Designing Data Intensive Applications
---

# Foundation of Database Systems

1. Reliability

- Hardware Faults
  - redundancy of hardware components
- Software Faults
  - think about assumptions and interactions
  - process isolation
  - measuring monitoring, monitoring, and analyzing system behavioro
- Human Errors
  - Minimize the opportunity for failure; easy to do the right thing and discourage the wrong thing
  - Decouple places where people make mistakes from places where they can cause failures
  - Test thoroughly: unit tests, integration tests, manual tests
  - Minimize impact in the case of failure

2. Scalability

- Throughput vs. response time
- Have to consider higher percentiles (E.G. 95th, 99th, 99.9th)
- Service level objectives (SLOs) and service level agreements (SLAs)
- _Head-of-line blocking_: small number of slow requests can hold up processing of subsequent
  requests
- _Tail latency amplification_: request is as slow as the slowest dependent request
- `HdrHistogram` to measure percentiles
- Scaling up vs. scaling out

3. Maintainability

- Three principles
  1. Operability
  - Visiblity into runtime behavior and internals of system
  - Automation and integration with standard tools
  - Good default behavior
  - Good documentation and clear operational model
  - Predictable behavior
  2. Simplicity
  - Clean abstractions
  3. Evolvability

## Data Models and Query Languages

### Data Models

#### Relational Model

- Data organized into relations
- Disconnect between objects in application code and database tables called _impedance mismatch_
- How to represent multi-to-one relationships?
  - Perform multiple tables or perform messy multi join between tables
  - Everything has an associated id
    - Consistent style and spelling
    - Avoiding ambiguity
    - Ease of updating (E.G. name change of a city)
    - Localization support
    - Never needs to change
- _Schema-on-write_: schema is explicit and database ensures all written data conforms to it
- _Column-family_ concept in BigTable allows for storage locality

#### Document Model

- Need for greater scalability; large datasets or high write throughput
- Specialized query operations that are hard to do in the relational model
- More dynamic and expressive data model
- Less impedance mismatch
- Better _storage locality_ -- one query is sufficient
- How to represent multi-to-multi relationships?
- _Schema-on-read_: structure of data is implicit and only interpreted when the data is read
  - Better when data is heterogeneous

#### Network Model

- Initial large following but faded into obscurity
- Generalization of the hierarchical model where a record could have multiple parents
- The way to access a record was to follow path from root record using links called an _access
  path_
- Code for querying and updating was complicated and inflexible as there were more than one way to
  reach a node
- Hard to change access path if you didn't have a path you wanted

#### Graph-Like Model

- Consists of two objects: _vertices_ and _edges_
- Well-known algorithms can operate on these graphs (E.G. shortest path, PageRank)
- Not limited to homogeneous data
- _Property graph_
  - Each vertex has
    1. A unique identifier
    2. A set of outgoing edges
    3. A set of incoming edges
    4. A collection of properties (key-value pairs)
  - Each edge has
    1. A unique idenitifer
    2. The vertex at which the edge starts (_tail vertex_)
    3. The vertex at which the edge ends (\_head vertex)
    4. A label to describe the relationship
    5. A collection of properties (key-value pairs)
  - Lots of flexibility for data modeling
  - Good for evolvability
  - _Cypher_ is a declarative language for property graphs
    - Difficult but possible to express in SQL
- _Triple-store model_
  - Mostly equivalent to the property graph but uses different words to describe the same ideas
  - All information stored in form _(subject, predicate, object)_ which is equivalent to vertex in
    graph
- _Datalog's data model_
  - Similar to triple-store model but _predicate(subject, object)_
  - Is a subset of Prolog
  - Define rules about new predicates and complex queries can be built up a small piece at a time

### Query Languages for Data

- Declarative
  - Specify the pattern of data you want, not how to achieve that goal
  - Relies on database system's query optimizer to figure how to do that
  - Hides implementation details of database engine (E.G. automatic operations)
  - Lends themselves to parallel execution
- Imperative
  - Tells computer to perform certain operations in a certain order
- MapReduce
  - Neither a declarative query language nor a fully imperative query API
  - Based on `map` and `reduce` functions that exist in functional languages
  - Must be pure functions
  - Easy to distribute and parallelize
  - Have to write two carefully coordinate functions, perhaps harder than writing a single query

## Storage

### Indexes

- Additional structure derived from primary data that can make some read paths faster
- Usually slows down write since index needs to updated every time data is written
- Simple index example: Hash index for key-value pairs (Bitcask) -- effective for keys that don't
  change often and values that do
  - Must fit in memory
  - Range queries are not efficient
- _Secondary indexes_ are crucial for performing joins efficiently
- Values in an index could be:
  1. The actual row
  - Known as a _clustered index_
  2. A reference to where the row is stored
  - The place where rows are stored is known as a _heap file_
  - When as value is updated, the heap file must be updated as well
  - If the value cannot find in the current location, it must be moved and either all indexes must
    be updated, or a forwarding pointer must be left behind
  3. A mix of the two approaches
  - Known as a _covering index_ or _index with included columns_
- Extra care is need when there are duplication to enforce transactional guarantees
- _Concatenated index_ contains several fields that are combined into one key
- Two-dimensional range queries often employ the use of specialized spatial indexes like R-trees
- Full-text search and fuzzy indexes
  - Lucene is able to search text for words within a certain edit distance
  - The in-memory index in Lucene is a finite state automaton over the characters in the keys
    which can be transformed into a _Levenshtein automaton_

### Log-Based Storage (LSM Trees)

- Break log into segments of a certain size
- Have to compact and merge segments to save space
- Deletion of keys involves writing a tombstone to overwritten previous values
- In-memory hash maps are lost in the event of a crash, so we have to restore each segment's hash
  map
  - Bitcask speeds up recovery by storing a snapshot of each segment's hash map on disk
- Partially written records need to detected and ignored
  - Bitcask files includes checksums
- Only one writer to logs, but they can be read concurrently by multiple threads
- Can use _Bloom filters_ to save unnecessary disk reads
- Can sustain higher write throughput than B-trees due to _write amplification_
- Can be compressed better and often produce smaller files on disk
- Compaction process needs to be able to keep up with write throughput

### B-Trees

- Keep key-value pairs sorted by key
  - Allows for efficient key value lookups and range queries
- Breaks database down into fixed-size _blocks_ or _pages_
- Generalized version of binary search tree where instead of having 2 children, the tree could
  have $N$ children
- Aims to maximizing the branch factor to minimize the pages read: A four-level tree of 4KB pages
  with a branching factor of 500 can store up to 256 TB
  - Can store an abbreviation of the key to differentiate it from other keys -- saves space and
    increases fanout
- Common for B-tree implementations to include an additional data structure on disk called a
  _write-ahead log_ (WAL)
  - Append-only file that is used to restore the B-tree back to a consistent state after a crash
- Careful concurrency control is needed when multiple threads access the B-tree
  - _Latches_ (lightweight locks) can protect the tree
- Lay out the tree out so that leaf pages appear in sequential order so there's locality during
  range queries
- Each key exists in exactly one place -- offer strong transactional semantics

### In-Memory Database

- Needs to reload its state when database is restarted
- Disk can be used as append-only log for durability
- Reads are served entirely from memory while files on disk can be backed up, inspected and
  analyzed
- Avoid overhead of encoding in-memory data structures to disk
- _Anti-caching approach_: evicting the least recently used data when there is not enough memory
  and restored when it is accessed

## Retrieval

### Transactional Processing

- Interactive access pattern called _online transaction processing_ (OLTP)
- OLTP systems are expected to be highly available and to process transactions with low latency
- Storage is typically laid out in _row-oriented_ fashion

### Analytics Processing

- Access pattern called _online analytics processing_ (OLAP)
- Scans over a large number of records and calculates aggregates
- Run analytics over a separate database called a _data warehouse_
  - Read-only copy of the data
  - Data is extracted from OLTP databases using periodic data dump or continuous stream of updates
    in a process known as _Extract-Transform-Load_ (ETL)
  - Optimized for analytic access patterns
  - Two main schemas
    1. Star schema: Events table have attributes and foreign key references to other tables called
       _dimension tables_
    2. Snowflake schema: Dimensions are further broken down into sub-dimensions
  - Storing data in _column-oriented_ fashion can speed up aggregations since they don't read
    irrelevant fields
    - Lends itself well to compression since the number of distinct values is often small compared
      to the number of rows
    - We can use _bitmap encoding_ for encode the positions that contains a particular id and then
      use run-length encoding to compress the values
    - Can use _vectorized processing_ to operate on column-oriented data
    - Maintaining a sorted column storage is advantageous as well for compression and range
      searching
    - We can also store the data sorted in different ways for redundancy and also to speed up
      certain queries
    - In order to update-in-place (B-tree approach) with compressed columns, we would have to
      rewrite all the column files, so we use the LSM approach
  - Can cache expensive operations using _materialized views_
    - Defined like a standard view, but result of query is written to disk
    - When underlying data changes, a materialized view needs to be updated
    - Makes writes more expensive, so not often used in OLTP databases, but they make sense in
      read-heavy data warehouses that bulk loads data

## Encoding and Evolution

### Formats for Encoding Data

1. In memory, data is kept in data structures optimized for access and manipulation.
2. To write to disk or send it over a network, we have to encode the data as some self-contained
   sequences of bytes.

#### Language-Specific Formats

- Java: `java.io.Serializable`
- Ruby: `Marshal`
- Python: `pickle`

There are problems related to language-specific formats:

- Tied to particular programming language, so may have difficult time integrating different
  programming languages
- Decoding process needs to be able instantiate arbitrary classes -- security vulnerability
- Versioning data is often ignored; how do we maintain forward and backward compatibility?
- Bad idea to use language's built-in encoding for anything besides transient purposes

#### Textual Formats (JSON, XML, CSV)

- Somewhat human-readable syntax
- Ambiguity around encoding of numbers
  - Cannot distinguish between number and string that consists only of numbers in XML and CSV
  - Cannot distinguish between integers and floating-point in JSON
  - Numbers larger than 2^53 cannot be exactly represented in IEEE 754
- No support for binary strings
- Optional schema support for XML, JSON; no schema for CSV

#### Binary Encoding

- Reduction in space usage
- MessagePack is a binary encoding for JSON
- Apache Thrift and Protocol Buffers can be used to compactly encode data using a schema
  - Does not encode field name, but includes field tags that appear in schema definition
  - Uses variable-length integers to improve space usage
  - You can add new fields to the schema by providing a new tag number
    - Old code will ignore the field with the new tag number (forward compatibility)
  - New code can always read old data because the tag numbers are unique and have the same meaning
    (backward compatibility)
  - To maintain backward compatibility, every field you add after the initial deployment must be
    optional or have a default value
  - Removing a field is like adding a field but with backward and forward compatibility concerns
    reversed
  - Third and Protocol Buffers rely on code generation for decoded data, type checking and
    auto-completion
- Apache Avro
  - Uses schema to specify the structure of data being encoded
  - No field tags, so to parse the binary data, you have to go through the fields in exactly the
    same order that they appear in the schema
  - Application encodes data with _writer's schema_
  - Client read data with _reader's schema_
  - An Avro reader can resolve the differences between writer's schema and reader's schema in the
    case that they are different -- the schemas only have to be compatible, but not identical
  - Client has to know the writer's schema to resolve differences
    1. Large file with lots of records: Send schema with records
    2. Database with individually written records: Keep version of schema with record and have table
       of schemas
    3. Sending records over network connection: Negotiate schema version on connection setup
  - Avro is friendlier to _dynamically generated_ schemas because you don't need to assign tag ids
  - Avro can be used without code generation

### Dataflow Through Databases

- Value written by new code can be read by old code which may clear fields written by new code since
  the old code will ignore new fields
- `data outlives code`: rewriting data into a new schema is expensive -- use simple schema changes
  to maintain forward and backward compatibility

### Dataflow Through Services: REST and RPC

- Clients make requests to web servers
- Servers can request another servers (_service-oriented architecture_ (SOA) or _microservices
  architecture_)
- When HTTP is used as the underlying protocol, it is called a web service
- Two popular approaches to web services
  1. _REST_: simple data formats, using URLs for identifying resources
  2. _SOAP_: XML-based protocol for making network API requests
- Making API requests over a network all stem from _remote procedure calls_ (RPC)
  - Tries to make a request to a remote network service the same as calling a function
  - Fundamentally flawed
    1. Network requests are unpredictable
    2. Network requests may fail or return without a result (_timeout_)
    3. Retrying a failed network request may have consequences if the action is not idempotent
    4. Network requests are much slower than function calls because of network latency
    5. Cannot use pointers or references in network requests and must encoded all parameters
    6. Client and service can be implemented in different programming languages
  - Various RPC frameworks have been built on top of previously mentioned encodings
    - Thrift and Avro have RPC support
    - gRPC uses Protocol Buffers
    - Finagle uses Thrift
    - Rest.li uses JSON over HTTP
  - New generation of RPC frameworks account for flaws
    - Use of _futures_ (_promises_) to encapsulate asynchronous actions that may fail
    - Use of _streams_ where a call returns a series of requests and responses over time
  - Custom RPC protocols with binary encoding format achieve better performance something generic
    (JSON over REST)
  - Backward and forward compatibility properties of an RPC scheme are inherited by whatever
    encoding it uses

### Message-Passing Dataflow

#### Asynchronous Message-Passing Systems

- a client's request is delivered to another process with low latency through a _message broker_
  which stores the message temporarily
- message broker can act as buffer if recipient is unavailable or overloaded
  - it can automatically redeliver messages
  - it avoids the sender needing to know the IP address and port number of the recipient
  - it allows one message to be sent to several recipients
  - it decouples the sender from the recipient
- messages are one way
- one process sends message to a named _queue_ or _topic_ and the broker ensures that the message is
  delivered to _consumers_ or _subscribers_

### Actor Model

- Logic is encapsulated in _actors_ which may have some local state
- Actors communicate with other actors by sending and receiving asynchronous messages
- Message delivery is not guaranteed and messages may be lost
- Can be used to scale an application across multiple nodes
- There is less of a fundamental mismatch between local and remote communication compared to RPCs

# Distributed Data

Reasons to distribute a database across multiple machines:

1. Scalability: can spread the load across multiple machines

- Vertical scaling in _shared-memory architecture_ is often not proportionate -- machine with
  twice the CPU, RAM, and disk capacity is probably more than twice expensive
- _Shared-disk architecture_ has a lot of contention and overhead of locking affects performance
- _Shared-nothing architecture_ is called _horizontal scaling_

2. Fault tolerance/high availability: Can use multiple machines to give you redundancy
3. Latency: Choose data centers that is geographically close to users to minimize network latency

## Replication

- Each node that stores a copy of the database is called a _replica_
- How is data synced among replicas?

### Single Leader-Based Replication

- One replica is designed the _leader_
- Clients send write requests to leader
- Leader sends data change to all _followers_ as part of a _replication log_
- Clients can send read requests to any follower
- _Synchronous replication_: wait until all followers acknowledge the write request from the leader
  before reporting to client
  - Write cannot be processed if any follower fails
- _Asynchronous replication_: leader does not wait for response before reporting to client
- _Semi-synchronous replication_: one follower is synchronous and if the follower fails, then an
  asynchronous follower is made synchronous
  - Guarantees that you have up-to-date data on at least two nodes
- Follower failure
  - Each follower keeps a log of the data changes
  - Can recover by requesting all transactions after the latest one recorded in its logs
- Leader failures
  - One of the followers need to be promoted to new leader and clients need to be reconfigured in a
    process called _failover_
    1. Determine that the leader as failed: most systems use a timeout to detect leader failure
    2. Choosing a new leader: Done through a election process or can be appointed by a _controller
       node_. The best candidate is the replica that has the most up-to-date data changes.
    3. Reconfiguring the system to use the new leader: clients have to send write request to new
       leader and old leader becomes a follower
  - A new leader's writes may not have replicated and most commonly discarded which may violate
    clients' durability
  - Discarding writes might be dangerous there is coordination between other storage systems
  - Two nodes might believe that they are the leader in the case of a fault (_split brain_)
  - Failover timeout is hard to configure; longer timeout means longer time to recover and a shorter
    timeout means unnecessary failovers

### Replication Logs

- Statement-based replication: leader logs every write request to followers for execution
  - Statements could be non-deterministic (E.G. `NOW()`, `RAND()`)
  - Statements could use auto-incrementing column or side effects that depends on requests being
    sent in a specific order
- Write-ahead log: closed coupled with how storage engine stores data
- Row-based log replication: use different log formats for replication vs. storage engine
  - For an inserted row, the log contains new values
  - For a deleted row, the log contains information to identify row that was deleted.
  - For updated row, log contains information to uniquely identify the updated row and the new value
    of the row
  - A transaction that modifies several rows contains multiple records and a record that indicates
    the transaction is finished
- Trigger-based replication: Move replication to application layer and have data changes trigger
  custom application code

### Replication Lag

- Application reads from an asynchronous follower may contain outdated data
- The followers still eventually caught up the leader (_eventual consistency_)

### Multi-Leader Replication

- Natural extension to single-leader replication
- Have a leader in each data center
- Better tolerance and performance than single-leader replication
- Writes can be concurrently modified and conflicts have to resolved
- Clients with offline operation and collaborative editing are examples of application of
  multi-leader replication
- _Topology_: Communication path along which writes are propagated
  - All-to-all: every leader sends its writes to every other leader
  - Circular: each node receives writes from one node and forwards those writes to another node
  - Star: designated root node forwards writes to all other nodes
- Have to order events properly; can use technique called _version vectors_

### Handling Write Conflicts

- Try to avoid conflicts in the first place
- Converge towards a consistent state
  - If each replica applied writes in the order that it saw writes, database would be in an
    inconsistent state
  - _Last Write Wins_ (LWW)
    - Achieves goal of eventual convergence at the cost of durability
    - If there were several concurrent writes that were reported as successful, one of the writes
      will survive and the others will be silently discarded
  - Merge values together
  - Record conflict and let the user resolve the conflict
    - On write: as soon as database system detects conflict, it calls conflict handler to resolve
    - On read: when a conflict is detected, all conflicting writes are stored and the application
      handles resolution on read
  - Automatic conflict resolution
    - Conflict-Free Replicated Datatypes (CRDTs): data structures that automatically resolve
      conflicts in sensible ways
    - Mergeable Persistent Data Structures track history explicitly and use three-way merge
      function
- _Version Vector_:
  - Update rules
    - Initially all counters are 0
    - Each time a replica experiences a local update event, it increments its own counter in the
      vector by one
    - Each time two replicas synchronize, they both set the elements in their vector to the maximum
      of the element across both vectors
  - Used to establish a partial order among replicas in a distributed system
- _Vector Clock_:
  - Update rules
    - Initially all clocks are 0
    - Each time a process experiences an internal event, it increments its own logical clock by 1
    - Each time a process sends a message, it increments its own logical clock by 1 then sends a
      copy of its vector
    - Each time a process receives a message, it increments its own logical clock by 1 and updates
      each element by taking the max between the element in its vector clock and the element in the
      received vector clock
  - Used to establish a partial order among a set of events

### Leaderless Replication

- Allow any replica to directly accept writes from clients
- Clients directly send writes to several replicas or coordinator node does it on behalf of client
- _Quorum_ reads and writes
  - Possible for writes to a subset of replicas to fail, but it is okay
  - Read from multiple replicas and take the latest one
  - If there are $n$ replicas, each write must be confirmed by $w$ nodes and we must query at least
    $r$ nodes where $w + r > n$ for there to be _quorum_ reads and writes
  - How to handle concurrent writes?
  - How to handle failed writes and rollbacks?
  - Concurrent write and read could return either the new or old data.
- _Read repair_: clients writes newer value to replicas with stale value
- _Anti-entropy process_: background process to look for differences in data and copies any missing
  data from one replica to another

### Sloppy Quorums and Hinted Handoff

- Writes and reads still require $w$ and $r$ successful responses, but not necessarily at the
  designated $n$ home nodes for a value
- After network interruption is fixed, writes must be sent to appropriate home nodes (_hinted
  handoff_)

## Partitioning

- Main reason to partition data is for _scalability_
- Usually combined with replication so copies of each partition are stored on multiple nodes
- If partition is unfair, then some partitions will have more data (_skewed_)
- A partition with disproportionately high load is called _hot spot_

### Partitioning by Key Range

- Can keep keys in sorted order
- Range scans are easy
- Certain access patterns can lead to hot spots

### Partitioning by Hash of Key

- Partition boundaries can be evenly spaced or chosen pseudorandomly (_consistent hashing_)
- Lose the ability to do efficient range queries
  - Cassandra uses a _compound primary key_ consisting of several columns
  - First key is hashed and other columns are used as a concatenated index, so it can perform
    efficient range scan over the other columns

### Skewed Workloads and Relieving Hot Spots

- Most applications cannot compensate for skewed workloads or hot spots
- Responsibility of the application
- E.G. Adding two-digit random number to hot user ID to partition the data across 100 keys
  - Reads would have to query from all 100 keys

### Secondary Indexes

- _Document-partitioned secondary indexes_: Keep secondary indexes between different partitions
  completely separate
  - Known as a _local index_
  - Have to query all partitions (_scatter/gather_)
  - Reads are slower and suffer from tail latency amplification
- _Term-partitioned secondary indexes_: The term we are looking for determines the partition of the
  index
  - Known as a _global index_
  - Makes reads more efficient
  - Writes are slower and more complicated and may affect multiple partitions of the index

### Rebalancing Partitions

- Why we would want to rebalance partitions?
  1. Query throughput increases
  2. Dataset size increases
  3. Machine fails
- **Do not partition by hashing mod $N$**: Makes rebalancing excessively expensive
- Have a fixed number of partitions that is much greater than the number of nodes
  - Nodes can steal partitions until partitions are fairly distributed
  - Assignment of keys to partitions does not change
  - Can give more powerful nodes more partitions
  - Choosing the right number of partitions is difficult if total size of dataset is highly variable
- Dynamic partitioning
  - Can create partitions for key range-partitioned databases dynamically
  - When partition grows to exceed a certain size, split partition in two
  - When partition shrinks below a threshold, merge with adjacent partition
  - Empty database starts with a single partition, but can configure initial set of partitions on an
    empty database (_pre-splitting_)
- Partitioning proportionally to nodes
  - Fixed number of partitions per node
  - When new node joins cluster, randomly choose a fixed number of existing partitions to split and
    take ownership of one half

### Request Routing

- Instance of more general problem called _service discovery_
- Approaches
  1. Allow clients to contact any node and the node will forward request to appropriate node if it
     does not contain the necessary information
  2. Send all requests to routing tier first
  3. Require all clients to be aware of partitioning and assignment of partitions
- Many distributed data systems require separate coordination service (E.G. ZooKeeper) to keep track
  of cluster metadata
- When partition changes ownership, ZooKeeper notifies routing tier

## Transactions

- _Transactions_ have been mechanism of choice for simplifying faults
- Groups several reads and writes into a logical unit and either the entire transaction succeeds
  (_commit_) or fails (_abort_, _rollback_)
- Created to simplify the programming model for applications accessing a database
- Application can ignore certain error scenarios and concurrency issues because it is handled on
  database side

### ACID

- Atomicity
  - If actions are grouped into a transaction, the actions as a whole succeeds or fails together
  - If transaction is aborted then nothing has changed in database
- Consistency
  - If a transaction is applied on a valid database, then the database will still be valid after the
    transaction is committed
  - Invariants of database (but not necessarily of application) are preserved
- Isolation
  - Concurrently executing transactions are isolated from each other
  - Ensures that when transactions have been committed, the result is the same if they had run
    serially
- Durability
  - Ensures that if transaction has committed successfully, the data it has written will not be
    forgotten even in the case of a crash

### Multi-Object Transactions

- In relational data model, a row in one table often has foreign key reference to a row in another
  table
- Have to update secondary indexes

### Handling Errors

- Problems with retrying an aborted transaction
  - If transaction succeeded but network failed while server tried to acknowledge the successful
    commit, then the transaction will be performed twice
  - If error is due to overload, then retrying will make the problem worse
    - Limit the number of retries
    - Use exponential backoff
  - If error is permanent (constraint violation), then retrying is pointless as it would fail again
  - If transaction has side effects, it might not be ideal to retry the transaction (sending email
    twice)

### Isolation Levels

#### Read Committed

- Properties
  1. When reading from database, you will only see data that has been committed (no _dirty reads_)
  - A transaction may see some updates, but not all updates of another transaction
  - A transaction may see data that is later rolled back
  2. When writing to the database, you will only overwrite data that has been committed (no _dirty
     writes_)
  - Conflicting writes between different transactions can be mixed up
- Databases can prevent dirty writes and reads by using row-level locks
- Long-running write transaction can stall many read-only transactions
- Databases can remember old committed value and new value set by transaction that currently holds
  the write lock
  - Reads return the old value
  - Once new value is committed, the old value will be overwritten

#### Snapshot Isolation and Repeatable Read

- Problems with _read committed_
  - _Non-repeatable reads_ or _read skews_ cause temporary inconsistencies (E.G. If object 1 and 2
    are related, read object 1 (transaction 1) then write object 2 (transaction 2) then read object
    2 (transaction 1))
  - Temporary inconsistencies can become permanent when taking backups or fail integrity checks
- _Snapshot Isolation_: Each transaction reads from a _consistent snapshot_ of the database
- Maintain several versions of an object side by side (_multi-version concurrency control_ (MVCC))
- Transaction is given a unique, always-increasing ID
  - Reads in transactions will access data from only committed transactions (ignore in progress
    transactions)
  - Creations and deletions will tag its ID
  - Rows are marked for deletion and then freed when no other transaction can access the undeleted
    row
- Updating indexes
  - Have index point to all versions of an object
  - Use persistent data structures (_append-only/copy-on-write_ B-trees)

#### Preventing Lost Updates

- Two concurrent _read-modify-write_ cycles will lose the update from one cycle (E.G. incrementing a
  counter)
- Solutions
  - Atomic write operations: take an exclusive lock on object when it is read
  - Explicit locking
  - Automatically detect lost updates: can perform efficiently with snapshot isolation
  - Compare-and-set: Only update if the value in the database is the value you think it is

#### Write Skew and Phantoms

- Concurrent writes to two disjoint but related objects can cause inconsistencies
- Common pattern to _phantoms_ (writes in one transaction affecting search query of another
  transaction) causing write skew
  1. A `SELECT` query to check some requirement
  2. Based on first query, either continue or abort
  3. Make write and commit transaction
- Solutions
  - Explicit locking
  - _Materializing conflicts_ takes a phantom and turns it into a lock conflict on a concrete set of
    rows

#### Serializability

- Guarantees that the end result of concurrently executed transactions is the same if they had
  executed serially
- Solutions
  - Actual Serial Execution: Remove concurrency and execute all transactions on a single thread
    - RAM is cheap enough to be feasible to keep the entire active dataset in memory
    - OLTP transactions are short and only make small number of reads and writes
    - OLAP queries are read-only so they can be run on a consistent snapshot
    - Use stored procedures to avoid interactive transactions
    - Can partition data, but cross-partition writes are slow
  - Two-Phase Locking (2PL)
    - Several transactions are allowed to concurrently read the same object, but writes have
      exclusive access
    - Locks on each object that has two modes: _shared mode_ or _exclusive mode_
    - If transaction first reads then writes an object, must upgrade shared lock to exclusive lock
    - Transaction holds lock until the end of the transaction
    - _Deadlocks_ are common so database should automatically detect deadlocks and abort one
      transaction
    - _Predicate lock_ locks similarly to shared/exclusive lock but it belongs to all objects that
      match a search condition
    - _Index-range locks_: checking for locks becomes time-consuming, so lock on an index to
      approximate predicate
  - Serializable Snapshot Isolation (SSI)
    - Optimistic concurrency control technique
    - Transactions continue in the hope that everything is fine and aborted when they are not
    - Performs badly when there is high contention
    - Built on top of snapshot isolation and adds an algorithm for detecting serialization conflicts
    - Transactions may take an action based on a _premise_ that is true at the beginning of the
      transaction, but false later
    - Have to handle 2 cases when a query result may have changed
      1. Detecting reads of a stale MVCC (uncommitted write occurred before read)
      - Keep track of any ignored writes that are uncommitted and check if any of these writes
        were committed when the transaction wants to commit
      2. Detecting writes that affect prior reads (the write occurs after the read)
      - Alert another transactions that their reads are out-of-date
      - Transaction should abort if another committed transaction alerted it

# The Trouble with Distributed Systems

- _Partial failure_: Some parts of system are broken in some unpredictable way, while other parts
  are working fine
- Most internet-related applications are _online_ and have to serve users with low latency at any
  time
- In a large system, it is reasonable to assume that there is always something that is broken
- It is possible to build a reliable system from unreliable components
  - Error-correcting codes tolerating some wrong bits
  - Transmission Control Protocol is built on top of the Internet Protocol (IP)

## Unreliable Networks

- The internet and most internal networks are _asynchronous packet networks_ and many problems can
  arise with sending and receiving a message
  1. Your request may be lost (network cable unplugged)
  2. Your request may be in a queue and will be delivered later (overloaded network or recipient)
  3. The remote node may have failed (crashed or powered down)
  4. The remote node may have temporarily stopped (long garbage collection pause)
  5. The remote node may have processed request but response has been lost (misconfigured switch)
  6. The remote node may have processed request but response has been delayed (overloaded network)
- Impossible to tell why you didn't receive request
- Usually handle this issue with timeouts

### Network Partitions

- One part of network is cut off from the rest due to a network fault
- Systems need to detect faulty nodes
  - Load balancer needs to stop sending requests to dead node
  - If leader fails in distributed database with single-leader replication, then one of the
    followers needs to be promoted to be the new leader
- Hard to get feedback to explicitly tell you that something is not working

### Timeouts and Unbounded Delays

- Timeout is the only sure way of detecting a fault
- Long timeout means a long wait until a node is declared dead, short timeout has a risk of
  incorrectly handling a temporary slow down
  - If node is alive and in the middle of an action, but declared dead, the action has to be
    performed twice
- Asynchronous networks have _unbounded delays_ (no upper limit on the time it may take for a packet
  to arrive)
  - If multiple packets are arriving at a destination, the network switch queues them up to feed them
    into the destination link one by lone (_network congestion_)
  - If all CPU cores are currently busy, the incoming request is queued by the operating system
  - TCP performs _flow control_/_congestion avoidance_/_back pressure_ in which a node limits its own
    rate of sending
- Can choose timeouts experimentally
  - Systems can continually measure response times and their variability and automatically adjust
    timeouts
- _Synchronous_ networks have _bounded delays_
  - A _circuit_ is established that allocates a guaranteed amount of bandwidth along the entire
    route between the two callers so there is no queueing
- Datacenter networks and the internet are packet-switched networks because they are optimized for
  _bursty traffic_
  - Packets have to be queue but it maximizes utilization of network

## Unreliable Clocks

- Each machine has its own clock (usually quartz crystal oscillator) which are not perfectly accurate
- It is possible to synchronize clocks to some degree (E.G. Network Time Protocol (NTP))
- _Time-of-day clocks_ returns the current date and time according to some calendar
  - Usually synchronized with NTP
  - If local time is too far ahead of NTP server, it may jump back to previous point
  - Unsuitable for measuring elapsed time
- _Monotonic clocks_ are guaranteed to always move forward
  - The absolute value of the clock is meaningless
  - The difference between two values tells you how much time has elapsed
  - NTP may adjust the frequency at which the monotonic clock moves forward, but cannot make it
    jump forward or backward
  - Using a monotonic clock for measuring elapsed time is fine in a distributed system
- Clocks aren't reliable
  - Quartz clock in computer is not very accurate and it _drifts_
  - If computer's clock differs too much from an NTP, it may refuse to synchronise or clock will be
    forcibly reset
  - Node can be firewalled off from NTP servers
  - NTP synchronization is affected by network delay
  - Leap seconds can sometimes result in a minute that is 59 seconds or 61 seconds. NTP serves
    usually performs the leap second adjustment over a course of a day (_smearing_)
  - Cannot trust clock in device's that you don't fully control
- Should not use wall-clock time to order events -- instead use _logical clocks_ which are based on
  incrementing counters
- Google's _TrueTime_ API reports the confidence interval on the local clock
  - Spanner implement snapshot isolation using the interval from TrueTime
  - To ensure that transaction timestamps reflect causality, Spanner waits for length of confidence
    interval before committing a read-write transaction

## Process Pauses

- A node in a distributed system may be paused for an unbounded amount of time
  - JVM have a garbage collector that occasionally needs to stop all running threads
  - A virtual machine can be _suspended_ and then _resumed_
  - Operating system can context-switch to another thread or hypervisor switches to another virtual
    machine (_steal time_)
  - Operating system can be configured to allow _swapping to disk_ and could spend most of its time
    swapping pages (_thrashing_)
- Some systems have a specified deadline by which the software must respond (_hard real-time
  system_)
  - Prioritize timely responses above all else
- Can treat GC pauses like brief planned outages of a node
  - Let other nodes handle request from clients while GC pause happens
  - Some latency-sensitive financial systems use this approach

## Knowledge, Truth, and Lies

- Truth is defined by the majority
- Distributed algorithms rely on a _quorum_ (voting among the nodes)
- Clients use a lock or lease to have protected access to resource
- A _fencing token_ is a number that increases every time a lock is granted

## Byzantine Faults

- Nodes are assumed to be honest
- Distributed system problems are much harder if there is a risk that nodes lie
  - Data in a computer's memory can be corrupted by radiation in aerospace environments
  - P2P networks like Bitcoin
- A system is _Byzantine fault-tolerant_ is it continues to operate correctly even if some of the
  nodes are malfunctioning
- Most Byzantine fault-tolerant algorithms require super-majority of more than two-third of the nodes
  to be functioning correctly

## System Model and Reality

- Three system models
  1. Synchronous model
  - Bounded network delay
  - Bounded process pauses
  - Bounded block error
  2. Partially synchronous model
  - Behaves like a synchronous system most of the time, except it sometimes exceeds the bounds for
    network delay, process pauses, and clock drift
  3. Asynchronous model
  - No timing assumptions can be made
- Three common node failures
  1. Crash-stop fault
  - Nodes can only fail by crashing
  - Nodes are gone forever after failing
  2. Crash-recovery faults
  - Nodes may crash at any moment and start responding after some unknown time
  3. Byzantine (arbitrary) faults
  - Nodes may be absolutely anything
- Correctness of Algorithm
  - Liveness: good things do eventually happen during execution of a program
  - Safety: bad things do not happen during execution of a program
- Theoretical analysis and empirical testing are equally important
- Abstract models help distill down the complexity of real systems

# Consistency and Consensus

## Consistency Models

- Reading Your Own Writes
  - Users should be able to immediately see their writes (_read-after-write consistency_)
  - Techniques to implement read-after-write consistency
    - When reading something a user may have modified, read from leader; otherwise, read from
      follower
    - Make immediate read request go to leader (E.G. all read requests go to leader for one minute
      after the last update)
    - Client can remember the timestamp of most recent write and then system can ensure that replica
      reflects updates at least until the timestamp
  - Might want to provide _cross-device_ read-after-write consistency
    - Another device does not know the timestamp of update on the other device
- Monotonic Reads
  - If a process preforms read $r_1$, then $r_2$, then $r_2$ cannot observe a state prior to the
    state reflected in $r_1$
  - Ensure that user always makes their reads from the same replica
    - What happens if replica fails?
- Monotonic Writes
  - If a process performs write $w_1$ then $w_2$, then all processes observe $w_1$ before $w_2$.
- Consistent Prefix Reads
  - Reads should return some prefix of writes with no gaps
  - There is possibly no global ordering of writes
- Eventual consistency
  - If no update takes a very long time, all replicas will eventually become consistent
  - Is only a liveness guarantee, _strong eventual consistency_ adds the safety guarantee that if
    any two nodes that have received the same (unordered) set of updates will be the same state
    - CRDTs are a common way of ensuring strong eventual consistency
- PRAM consistency
  - Enforces that any pair of writes executed by a single process are observed everywhere in the
    same order
- Casual consistency
  - Only casually related operations must occur in order
  - Independent casual chains could execute in any relative order
- Sequential Consistency
  - Order is enforced on one node
  - Sequential consistency is essentially linearizability without real-time constraint - processes
    are allowed to skew in time
- Linearizability
  - Basic idea is make a system appear as if there were only one copy of the data and all operations
    on it are atomic
  - As soon as one client completes a write, all client reading from the database must be able to
    see the value just written
  - The invocations and responses of requests in a system can be ordered to yield a sequential
    history that is valid (E.G. consecutive reads can't return different results) and if a response
    preceded an invocation in the original history, it must also precede it in the sequential
    reordering
    - Concurrent operations can be reordered
  - Is a single-object property
  - The combination of both serializability and linearizability is called _strict serializability_
    - Is a multi-object property
  - Useful to guarantee that there is only one leader
  - Linearizable coordination services like Apache ZooKeeper and etcd are often used to implement
    distributed locks and leader election
  - Analysis of replication methods:
    - Single-leader replication: potential to be linearizable
    - Consensus algorithms: linearizable
    - Multi-leader replication: not linearizable because leaders concurrently process writes and
      asynchronously replicate them
    - Leaderless replication: probably not linearizable
  - Strict quorum can be made linearizable if readers perform read repair synchronously before
    returning results to application
  - CAP Theorem: when a network fault occurs, you have to choose between either linearizability or
    total availability
  - Linearizability is slow -- multi-core CPU is not linearizable for performance

## Ordering Guarantees

- Causality imposes an ordering on events: cause comes before effect
- Causality is a partial ordering (occurred before, after or concurrently)
- Linearizability is stronger than casual consistency
- Can use _sequence numbers_ or _timestamps_ from a _logical clock_ to provide a _total order_ for
  events in single-leader replication
- Can also use _Lamport timestamps_ to generate sequence numbers
  - Pair of $(counter, node ID)$
  - Each client keeps track of the maximum counter so far and attaches it with the request
  - When a node takes the maximum of its counter value and the received counter value and increments
    it by 1
  - Total ordering: the timestamp with the greater counter is the greater timestamp; if the counter
    values are the same, the one with the greater node ID is the greater timestamp
- Timestamp ordering is not sufficient to solve many common problems in distributed systems
  - Can only compare timestamps after the events have occurred
  - Cannot be used to answer immediate decision problems

## Total Order Broadcast

- Properties
  1. Validity: If a correct participant broadcasts a message, then all correct participants will
     eventually receive it
  2. Uniform Agreement: If a correct participant broadcasts a message, then all correct participants
     will eventually receive it
  3. Uniform Integrity: A message is received by each participant at most once and only if it was
     previously broadcast.
  4. Uniform Total Order: Messages are delivered to correct participants in the same order.
- Uses
  - Consensus services like ZooKeeper and etcd
  - _State machine replication_: if every message represents a write to the
    database and every replica processes the writes in the same order, then the database on each
    replica is consistent
  - Serializable transactions
  - Logs (replication log, transaction log, write-ahead log, etc.)
  - Lock service: every request to acquire the lock is appended as a message to the log with a
    sequential monotonically increasing number which becomes the fencing token

## Implementing Linearizable Storage Using Total Order Broadcast

- Linearizable writes (compare-and-set)
  - Append a message to the log with the old and new values
  - Read the log and wait for the message to be delivered back to you
  - If there are any conflicting log entries, then abort
  - After receiving your message, commit the operation
- Linearizable reads
  - Append a message and when you receive your message again, perform the read operation
  - Fetch the position of the latest log message in a linearizable way and wait for that message to
    be delivered to you, then perform the read operation
  - Read from a replica that is synchronously updated on writes

## Implementing Total Order Broadcast Using Linearizable Storage

- Assume you have a linearizable register that stores an integer and has an atomic increment-and-get
  operation
- For every message you want to send through total order broadcast, increment-and-get the
  linearizable register and deliver the message to all nodes
- Process the message in sequential order with no gaps

## Consensus

- FLP result: there is no algorithm that is always able to reach consensus if there is a risk that a
  node may crash
  - FLP result is proved in the asynchronous system that cannot use clocks or timeouts
  - In practice, we can use timeouts and other ways to identify suspected crashed nodes
- _Linearizable sotrage and total order broadcast are equivalent to consensus_
- Uses
  - Leader election: nodes must agree on a single leader or there might be a split brain situation
  - Atomic commit: nodes must agree on whether a transaction succeeded or failed so that they can
    all commit or rollback

## Atomic Commit

- Atomicity is commonly implemented by storage engine
- Not sufficient to send commit request to all nodes and independently commit the transaction on
  each one
  - Some noes may detect constraint violation and aborts while other nodes successfully commit
  - Some commit requests may be lost in the network
  - Some nodes may crash before the commit record is fully written
- Cannot revert committed transactions, because reads on committed data must be retroactively
  reverted as well

## Two-Phase Commit (2PC)

- 2PC uses a coordinator component
- Steps
  1. When application wants to begin a distributed transaction, it requests a transaction ID that is
     globally unique
  2. Application beings a single-node transaction on each of the participants ans attaches
     transaction ID
  3. When application is ready to commit, coordinator sends _prepare_ request
  - If all participants reply yes, then coordinator sends _commit_ request
  - If any of the participants reply no, then coordinate sends _abort_ request
  4. After commit request is sent, coordinate writes decision to transaction log (_commit point_)
  5. If participant crashes, transaction will be committed when it recovers and participant cannot
     refuse to commit when it recovers
- _Blocking_ atomic commit because 2PC can be stuck waiting for coordinator to recover

### Coordinator Failure

- If coordinator fails before sending prepare requests, then participants can abort transaction
- If participant replies yes to prepare request, it must wait to hear back from coordinator
  - If coordinator crashes or network fails, a participant goes into an _in doubt_ or _uncertain_
    state.
  - A transaction holds onto the locks through the time that the node is in doubt
  - Other transactions will be blocked
- The only way 2PC can complete is by waiting for the coordinator to recover
- Coordinator must write its commit or abort decision on transaction log before sending commit or
  abort requests
- Commit point is essentially a regular single-node atomic commit on the coordinator

## Three-Phase Commit

- Assumes a network with bounded delay and nodes with bounded response times
- In most practical systems with unbounded network delay and process pauses, no guarantee for
  atomicity
- In general, non-blocking atomic commit requires _perfect failure detector_ (timeouts are not
  perfect)

## Distributed Transactions in Practice

- Provide important safety guarantee but they are criticized for causing operational problems and
  killing performance
- Types of distributed transactions
  1. Database-internal distributed transactions: All participants are running the same database
     software
  2. Heterogeneous distributed transactions: Participants are two or more different technologies
  - E.G. message from message queue can be acknowledged if and only if database transaction
    processing the message is successfully committed
  - XA transactions is a standard for implementing 2PC for heterogeneous distributed transactions

## Limitations of Distributed Transactions

- If coordinator is not replicated, it is a single point of failure for the system
- Coordinator is a part of the application server, and it changes the nature of the deployment since
  most server-side applications are developed in a stateless model
- XA needs to compatible with a wide range of data systems -- lowest common denominator
- Distributed transactions have tendency to amplify failures as all participants need to respond to
  successfully commit a transaction

## Fault-Tolerant Consensus

- Properties
  1. Uniform agreement: No two nodes decide differently
  2. Integrity: No node decides twice
  3. Validity: If a node decides value $v$, then $v$ was proposed by some node
  4. Termination: Every node that does not crash eventually decides some value
  - Formalizes the idea of fault tolerance
  - Requires at least a majority of nodes to be functioning correctly in order to assure
    termination (assumes no Byzantine faults)
  - If there are Byzantine faults, there must be more than one-third of the notes are not
    Byzantine-faulty for there to be termination
- Total order broadcast is equivalent to repeated rounds of consensus

### Epoch Numbering and Quorums

- Every time current leader is thought dead, vote is started among nodes to elect new leader
- Election is given an incremented epoch number (totally ordered and monotonically increasing)
  - If there is conflict between two leaders in two different epochs, then leader with higher epoch
    number wins
- Leader must collect votes from _quorum_ of nodes
  - Node votes in favor of a proposal if it is not aware of any other leader with a higher epoch
- Two rounds of voting: one to choose leader and another to vote on leader's proposal
- Need a quorum intersection between round one and round two

### Limitations of Consensus

- Process which nodes vote on proposals is a kind of synchronous replications
  - Some committed data can be potentially be lost a failover
- Most consensus algorithms assume fixed set of nodes
  - _Dynamic membership_ extensions are less understood
- Reply on timeouts to detect failed nodes
- Designing algorithms that are most robust to unreliable networks is still an open research problem

## Membership and Coordination Services

- Coordination and configuration services like ZooKeeper or etcd are not well suited for
  general-purpose databases
- Features
  - Linearizable atomic operations
  - Distributed locks (_lease_)
  - Total ordering of operations
  - Failure detection
  - Change notifications
- Uses
  - Choosing leader
  - Partitioning resources
  - Service discovery: on startup, services register their network endpoints in a service registry
- ZooKeeper runs on a fixed small number of nodes and supports a potentially large number of clients
- Provides a way of outsourcing work of coordinating nodes to external service

# Derived Data

- Two broad categories of systems that store and process data
  1. _Systems of record_ hold authoritative version of your data
  2. _Derived data systems_ are the result of taking some existing data from another system and
  transforming or processing it in some way
- Distinction between the two depends not on the too, but on how you use it
- Three different types of systems
  1. _Services (online systems)_ tries to minimize response time
  2. _Batch processing systems (offline systems)_ take a large amount of input data, runs a job to process it and
     produces some output data
     - Tries to maximize throughput
     - Often scheduled periodically
  3. _Stream processing systems (near-real-time systems)_ are somewhere between online and offline
     systems
    - Stream jobs operate on events shortly after they happen and have lower latency than batch
      systems

## Batch Processing

- Can analyze a log file using a chain of Unix commands
- Unix philosophy
  - Make each program do one thing well
  - Expect the output of every program to become the input to another
  - Design and build software to be tried early
  - Use tools in preference to unskilled help to lighten a programming task
- Emphasizes automation, rapid prototyping, incremental iteration, experimentation, and breaking
  down complex projects to manageable tasks
- Unix tools have a uniform interface
- Biggest limitation of Unix tools is that they run only a single machine

## MapReduce and Distributed Filesystems
