# Designing-Data-Intensive-Applications-Notes

## Table of Contents
- [Chapter 1: Reliable, Scalable, and Maintainable Applications](#chapter-1-reliable-scalable-and-maintainable-applications)
- [Chapter 2: Data Models and Query Languages](#chapter-2-data-models-and-query-languages)
- [Chapter 3: Storage and Retrieval](#chapter-3-storage-and-retrieval)
- [Chapter 4: Encoding and Evolution](#chapter-4-encoding-and-evolution)
- [Chapter 5: Replication](#chapter-5-replication)
- [Chapter 6: Partitioning](#chapter-6-partitioning)
- [Chapter 7: Transactions](#chapter-7-transactions)
- [Chapter 8: The Trouble with Distributed Systems](#chapter-8-the-trouble-with-distributed-systems)
- [Chapter 9: Consistency and Consensus](#chapter-9-consistency-and-consensus)
- [Chapter 10: Batch Processing](#chapter-10-batch-processing)
- [Chapter 11: Stream Processing](#chapter-11-stream-processing)
- [Chapter 12: The Future of Data Systems](#chapter-12-the-future-of-data-systems)

## Chapter 1: Reliable, Scalable, and Maintainable Applications

### Data-Intensive vs. Compute-Intensive
- Modern applications are increasingly **data-intensive** rather than compute-intensive
- Data-intensive applications typically need to:
  - Store data (databases)
  - Remember results of expensive operations (caches)
  - Allow users to search data (search indexes)
  - Send messages to other processes (stream processing)
  - Process large amounts of data (batch processing)

### Three Key Concerns
1. **Reliability**
   - System should work correctly even in the face of adversity
   - Common faults: hardware faults, software errors, human errors
   - Techniques: redundancy, thorough testing, monitoring, easy recovery

2. **Scalability**
   - As system grows (data, traffic, complexity), there should be reasonable ways to deal with that growth
   - **Load parameters**: requests per second, read/write ratio, active users, cache hit rate
   - **Performance metrics**: throughput, response time, latency, percentiles (p50, p95, p99)
   - Approaches: vertical scaling (more powerful machines) vs. horizontal scaling (distributing load)

3. **Maintainability**
   - Over time, many people will work on the system - we need to make it easy for them
   - Key aspects:
     - **Operability**: Make it easy for operations teams to keep the system running
     - **Simplicity**: Make it easy for new engineers to understand the system
     - **Evolvability**: Make it easy to make changes to the system

### Implementation Abstractions
- Good abstractions hide implementation details, reducing complexity
- Data systems are a crucial abstraction for modern software

## Chapter 2: Data Models and Query Languages

### Relational Model vs. NoSQL
- **Relational databases**
  - Dominant since the 1970s
  - Data organized into relations (tables), with rows and columns
  - Schema enforced, normalization promoted
  - Joins are common operations

- **NoSQL motivations**
  - Need for greater scalability
  - Preference for free and open source software
  - Specialized query operations
  - Frustration with restrictive relational schemas
  - Desire for more dynamic and expressive data models

### NoSQL Categories
- **Document databases** (MongoDB, CouchDB)
  - Data stored in self-contained documents, often JSON-like
  - Schema flexibility, better locality

- **Graph databases** (Neo4j, Titan)
  - Optimized for data with complex many-to-many relationships
  - Vertices (nodes/entities) and edges (relationships/arcs)
  - Examples: social networks, web graphs, road networks

- **Column-family** (Cassandra, HBase)
  - Store data in column families, offering high write throughput
  - Inspired by Google's Bigtable paper

### Query Languages
- **Declarative vs. Imperative**
  - SQL is declarative: specify what results you want, not how to compute them
  - Imperative code: specify the exact sequence of operations

- **MapReduce**
  - Programming model for processing large datasets
  - Splits computation into map and reduce functions

- **Graph Query Languages**
  - Cypher (Neo4j), SPARQL (RDF), Gremlin
  - Property graphs vs. triple-stores

## Chapter 3: Storage and Retrieval

### Data Structures
- **Log-structured storage**
  - Append-only sequence of records
  - Fast writes, but reads require scanning the entire dataset
  - Examples: Bitcask, SSTables

- **B-trees**
  - Standard index implementation in most relational databases
  - Keeps key-value pairs sorted by key
  - O(log n) operations for lookups, inserts, deletes
  - Well-suited for both large reads and random access

- **LSM-Trees (Log-Structured Merge Trees)**
  - Key-value storage engine optimized for high write throughput
  - Used in Cassandra, LevelDB, RocksDB
  - Combines in-memory storage and on-disk SSTables
  - Compaction merges and cleans up data

### Transaction Processing vs. Analytics
- **OLTP (Online Transaction Processing)**
  - User-facing, high volume of small reads/writes
  - Low latency is critical
  - Typically accesses small amount of records per query

- **OLAP (Online Analytical Processing)**
  - Used for business intelligence, analytics
  - Low volume of complex queries analyzing large portions of data
  - Throughput (not latency) is the priority

- **Data Warehousing**
  - Separate database optimized for analytics
  - ETL (Extract-Transform-Load) processes copy data from OLTP systems
  - Star and snowflake schemas common for dimensional modeling

### Column-Oriented Storage
- Store each column separately rather than each row
- Better compression, as column data is often similar
- Queries can read only relevant columns, improving efficiency
- Examples: Redshift, BigQuery, Vertica, ClickHouse

## Chapter 4: Encoding and Evolution

### Formats for Encoding Data
- **Language-specific formats**
  - Java serialization, Python pickle, etc.
  - Major drawbacks: language coupling, security/versioning issues

- **JSON, XML, and Binary Variants**
  - Text-based formats are human readable but inefficient
  - Binary encodings (MessagePack, BSON, BJSON, UBJSON)
  - Schema-less but often ambiguous types

- **Thrift and Protocol Buffers**
  - Schema-based binary encoding formats
  - Compact, efficient, strongly typed
  - Forward and backward compatibility
  - Field tags critical for schema evolution

- **Avro**
  - Schema-based like Thrift/ProtoBuf but no field tags
  - Schema resolution using field names
  - Writer's schema and reader's schema concept

### Modes of Dataflow
- **Databases**
  - Writer encodes, reader decodes later
  - Schema evolution needs to maintain compatibility

- **RPC and REST APIs**
  - Client-server communication
  - Service compatibility across versions
  - REST typically uses JSON over HTTP
  - gRPC uses Protocol Buffers

- **Message Passing**
  - Asynchronous message brokers (RabbitMQ, Kafka)
  - One-way dataflow through intermediary
  - Useful for cross-service communication
  - Must handle backward/forward compatibility

## Chapter 5: Replication

### Reasons for Replication
- Keep data geographically close to users (reduce latency)
- Allow the system to continue working during outages (increase availability)
- Scale out read capacity (improve throughput)

### Replication Methods
- **Single-leader replication**
  - One node handles writes, others are read-only replicas
  - Common in relational databases (PostgreSQL, MySQL)
  - Asynchronous vs. synchronous replication tradeoffs
  - Challenges: failover, replication lag, consistency

- **Multi-leader replication**
  - Multiple nodes accept writes
  - Used for multi-datacenter operation
  - Conflict resolution becomes critical
  - Examples: Cassandra, CouchDB

- **Leaderless replication**
  - Any replica may directly accept writes from clients
  - Client reads from multiple replicas and resolves conflicts
  - Examples: Amazon Dynamo, Riak, Cassandra (when configured)
  - Quorum reads and writes (W + R > N)

### Handling Replication Lag
- **Read-your-writes consistency**
  - Users should see their own updates
  - Solutions: read from leader after writing, track timestamps

- **Monotonic reads**
  - Avoid going backward in time when making multiple reads
  - Read from same replica or use timestamps

- **Consistent prefix reads**
  - Causally related writes appear in correct order
  - Partitioning issues make this challenging

## Chapter 6: Partitioning

### Partitioning Strategies
- **Key-range partitioning**
  - Sort keys and assign ranges to partitions
  - Simplifies range queries
  - Risk of hot spots

- **Hash partitioning**
  - Apply hash function to keys
  - Even data distribution
  - Loses ordering, making range queries inefficient
  - Consistent hashing reduces rebalancing

### Secondary Indexes
- **Document-based partitioning**
  - Local indexes per partition
  - Scatter/gather for queries
  - Read amplification

- **Term-based partitioning**
  - Global indexes across partitions
  - More efficient reads, more complex writes

### Rebalancing
- **Avoid hash mod n** (causes massive reshuffling)
- **Fixed number of partitions** (more than nodes)
- **Dynamic partitioning** (split when size threshold reached)
- **Proportional to nodes** (fixed partitions per node)

### Request Routing
- **Service discovery**
  - ZooKeeper often used to track partition assignment
  - Options: node-direct vs. routing tier vs. client-aware

## Chapter 7: Transactions

### ACID Properties
- **Atomicity**: All operations complete or none do
- **Consistency**: Database remains in valid state
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed data won't be lost

### Isolation Levels
- **Read Committed**
  - No dirty reads
  - Non-repeatable reads possible
  - Default in PostgreSQL, SQL Server

- **Snapshot Isolation**
  - Each transaction sees consistent snapshot
  - Prevents non-repeatable reads
  - Write skew still possible

- **Serializable**
  - Strongest isolation level
  - Transactions behave as if executed serially
  - Implementation techniques:
    - Actual serial execution
    - Two-phase locking (2PL)
    - Serializable snapshot isolation (SSI)

### Weak Isolation Problems
- **Dirty reads**: Reading uncommitted data
- **Dirty writes**: Overwriting uncommitted data
- **Read skew**: Non-repeatable reads in transaction
- **Lost updates**: Concurrent writes overwriting each other
- **Write skew**: Write decisions based on earlier reads

### Implementation Techniques
- **Optimistic concurrency control**
  - Allow transactions to proceed, abort if conflict
  - Good when conflicts are rare

- **Pessimistic concurrency control**
  - Prevent conflicts via locking
  - Better when conflicts are common

## Chapter 8: The Trouble with Distributed Systems

### Types of Network Faults
- Packets can be:
  - Lost (dropped)
  - Delayed
  - Duplicated
  - Reordered

### System Model Assumptions
- **Synchronous model**
  - Bounded network delay, bounded process pauses
  - Unrealistic in practice

- **Asynchronous model**
  - No timing assumptions
  - Hard to distinguish slow vs. failed nodes

- **Partial synchronous model**
  - System behaves synchronously most of the time
  - Most practical distributed systems use this model

### Unreliable Clocks
- **Clock drift** between machines
- **NTP** not precise enough for many uses
- Time-of-day clocks vs. monotonic clocks
- Problems with synchronized clocks:
  - Limited precision
  - Leap seconds
  - Clock drift and skew

### Detecting Failures
- Timeout-based detection is error-prone
- Failure detector challenges:
  - Network delays
  - Process pauses (GC, virtual machine issues)
  - No guarantee of accuracy

### Byzantine Faults
- Nodes may lie or send contradictory messages
- Byzantine fault tolerance protocols complex
- Primarily needed in hostile environments

## Chapter 9: Consistency and Consensus

### Consistency Models
- **Linearizability**
  - Makes a distributed system appear as if there's only one copy of the data
  - All operations appear to execute in a single, totally ordered sequence
  - Cost: reduced performance, especially with network delays

- **Causal consistency**
  - Weaker than linearizability
  - Tracks causal relationships between operations
  - Allows concurrent, causally independent operations

### Consensus Problems
- **Leader election**
- **Atomic commit**
- **Total order broadcast**

### Consensus Algorithms
- **Two-Phase Commit (2PC)**
  - Used for atomic transaction commit across nodes
  - Coordinator and participants
  - Blocking on coordinator failure

- **Paxos**
  - Classic algorithm for fault-tolerant consensus
  - Complex to understand and implement
  - Uses quorum voting

- **Raft**
  - Designed to be more understandable than Paxos
  - Leader-based with terms
  - Log replication and safety guarantees

- **ZooKeeper (ZAB protocol)**
  - Total order broadcast
  - Linearizable operations
  - Used for coordination in distributed systems

## Chapter 10: Batch Processing

### Origins of Batch Processing
- UNIX tools as inspiration (grep, sort, uniq, etc.)
- Functional programming principles

### MapReduce
- **Map**: Extract data from input records
- **Reduce**: Aggregate data with the same key
- **Benefits**:
  - Handles large datasets
  - Fault tolerance through re-execution
  - Locality optimization

### Beyond MapReduce
- **Limitations of MapReduce**:
  - Verbose code
  - Limited operation types
  - Performance issues for iterative algorithms

- **Dataflow engines** (Spark, Tez, Flink)
  - DAG of operators
  - In-memory processing
  - More flexible than rigid map-reduce

### Data Flow from Storage to Output
- **Distributed file systems** (HDFS)
- **Column-oriented formats** (Parquet, ORC)
- **Join algorithms**:
  - Sort-merge joins
  - Broadcast joins
  - Partitioned joins

## Chapter 11: Stream Processing

### Stream Processing vs. Batch Processing
- **Batch**: Fixed-size input, produce output
- **Stream**: Unbounded input, produce continuous output

### Event Time vs. Processing Time
- **Event time**: When the event actually occurred
- **Processing time**: When the system processes the event
- Handling late events and time skew

### Message Delivery Semantics
- **At-most-once**: Messages may be lost
- **At-least-once**: Messages won't be lost but may be duplicated
- **Exactly-once**: Messages delivered exactly once

### Stream Processing Patterns
- **Stream-stream joins**
  - Joining data from different streams
  - Window-based approaches

- **Stream-table joins**
  - Enriching streams with data from tables
  - Table as a changelog

- **Table-table joins**
  - Materialized views
  - Incrementally updated results

### Stream Processing Systems
- **Direct messaging**: RabbitMQ, Kafka, Kinesis
- **Complex event processing (CEP)**: Pattern matching on streams
- **Stream analytics**: Aggregations, windowing operations

## Chapter 12: The Future of Data Systems

### Data Integration
- **Combining specialized tools**
  - Databases, caches, search indexes, batch/stream processors
  - Integration challenges

- **Batch and stream processing**
  - Lambda architecture vs. Kappa architecture
  - Unified solutions (Beam, Flink)

### Derived Data vs. System of Record
- **Source of truth**
  - Original data, immutable
  - Basis for derived datasets

- **Derived data**
  - Result of transformations
  - Can be regenerated from source

### Organizational Challenges
- **Data ownership**
  - Who can access what data?
  - Who is responsible for data quality?

- **Data governance**
  - Compliance (GDPR, etc.)
  - Audit trails
  - Ethics of data collection and use

### The Future
- **Unbundling databases**
  - Specialized systems for specific tasks
  - Composable data systems

- **Designing for correctness**
  - End-to-end thinking
  - Exactly-once semantics
  - Idempotence

- **Designing for humans**
  - Tools should empower people
  - Understanding and maintaining complex systems
  - Transparent and predictable behavior
