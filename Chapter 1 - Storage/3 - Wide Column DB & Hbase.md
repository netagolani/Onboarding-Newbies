# Introduction to HBase :elephant:

> **Note:** this document was renamed earlier to `Wide Column DB & Hbase` to reflect the broader category; the title remains centered on HBase for now.

## Overview
Today’s session dives deeper into column‑oriented databases with a focus on Apache HBase, the Hadoop ecosystem’s wide‑column store. (The filename has been updated to “Wide Column DB & Hbase” per reviewer suggestion.) Understanding HBase will help you see how low‑latency random access is provided over massive data sets.

**The emphasis is on HBase’s architecture, core components, and operational model.**

## Goals
- Grasp the columnar database model and why HBase exists.
- Learn the responsibilities of key HBase components (RegionServer, ZooKeeper, HFile, etc.).
- Improve your ability to plan and self‑direct learning.

:warning: **Note:**
- This is a self‑study day; independence and time management are crucial.
- If you can’t explain a concept clearly, you probably need to revaisit it.
- Read the [Exercise](#exercise) before starting so you know what to emphasize.
- Ask your mentor if you’re unsure what to research.

### ⏳ Timeline
Estimated Duration: 3 Days
- Day 1: Learn the concepts of wide column DB and HBASE spesficly; spend the day.
- Day 2-3: Get deep into HBASE spesficly
    - Have a Q&A session at the third day and in between sessions each day

## Core Concepts

## Part 1: Wide Column Databases (General Concepts)

Answer these questions to understand the fundamentals of wide-column databases before focusing on HBase:

1. **Data Model & Structure:**  
   What is a wide-column database, and how does its data model work? Explain the concepts of rows, column families, and flexible schemas. How does this model differ from traditional relational databases and key-value stores?

2. **Use Cases & Motivation:**  
   Why do wide-column databases exist? In what scenarios are they most useful (for example: large-scale datasets, time-series data, sparse data, or systems requiring high write throughput)?

3. **Distributed Design:**  
   How do wide-column databases distribute data across clusters? Explain concepts such as partitioning, replication, and horizontal scalability.

## Part 1: Wide Column Databases (General Concepts) - Answers

1. **Data Model & Structure:**  
- What is a wide-column database? A wide-column database is a NoSQL database that organizes data storage into flexible columns that can be spread across multiple servers or database nodes, using multi-dimensional mapping to reference data by column, row, and timestamp. It automatically timestamps column values, enabling retrieval of historical data states effortlessly.
- how does its data model work? Data is organized by rows and columns, but the focus is on storing columns together rather than rows, which makes wide column databases more efficient for read-heavy workloads. A column family might look like a table, but each row can have its own unique set of columns, which gives the “wide” nature.
- rows - A row in a wide column database has a unique row key.
- columns family - Each row is composed of multiple columns grouped into column families. The column families serve as containers for the actual columns of data, and each row can store different sets of columns (flexibility).
- flexibility - Wide column databases allow a flexible schema, which means you can easily add new columns without affecting existing rows. This is ideal for applications that require evolving structures or need to accommodate different data formats. Wide-column databases don’t have a defined table schema, which leaves them flexible to have certain columns only apply to certain records.
- How does this model differ from traditional relational databases and key-value stores? A relational database management system (RDBMS) stores data in a table with rows that all span a number of columns. If one row needs an additional column, that column must be added to the entire table, with null or default values provided for all the other rows. If you need to query that RDBMS table for a value that isn’t indexed, the table scan to locate those values will be very slow.
Wide-column NoSQL databases still have the concept of rows, but reading or writing a row of data consists of reading or writing the individual columns. A column is only written if there’s a data element for it. Each data element can be referenced by the row key, but querying for a value is optimized like querying an index in a RDBMS, rather than a slow table scan. Key-Value databases are the simplest model and can be thought of as a configuration file or a two-column table of keys with an associated value. Wide-column databases expand that key-value store concept across multiple columns, but only the columns that are needed for that record.

2. **Use Cases & Motivation:**
- Why do wide-column databases exist? Powerful solutions for massive amount of data  that can be easily added new data attributes without complex schema migrations. In cases you required to add column in a flexible way.
- In what scenarios are they most useful? In scentios you have a lot of queries of columns, OLAP usecases, scaling horizontally.
   - Business intelligence and analytics - Their ability to quickly scan and aggregate specific columns makes them perfect for tasks like sales analysis, financial forecasting, and trend identification.
   - Data warehousing - to store and process massive amounts of historical data. Columnar storage allows for efficient querying across vast datasets, enabling organizations to perform complex analyses and support decision-making.
   - Big data processing 
   - Log and event data analysis - Columnar databases are ideal for analyzing log files, telemetry data, and event streams. Their compression and query performance make them suitable for monitoring systems, troubleshooting, and identifying patterns in high-volume data.

 3. **Distributed Design:**\
 How do wide-column databases distribute data across clusters?
 - horizontal scalability - by storing the data in a wide column way, it can be across many servers.
 - partitioning - Allows parallel processing of large datasets across multiple nodes. Data is partitioned using a partition key (usually a hash key)
 - replication - replicates the partition in different servers.

---

### Part 2: Apache HBase (Implementation & Operations)

Answer these five questions to cover HBase’s major areas:

1. **Architecture & Data Model:**  
   Describe the overall architecture of Apache HBase, including tables, rows keyed by row key, column families, regions, and the storage format (HFile). How do these elements differ from a traditional relational database, and why is schema design driven by access patterns?

2. **Components & Storage Flow:**  
   Explain the roles of RegionServers, MemStore, HFiles, block cache, and the Write-Ahead Log (WAL). How does data flow from a client write to durable storage, and how are reads served from memory and disk structures?

3. **Performance & Maintenance:**  
   What are minor and major compactions, MOB storage, Bloom filters, and caching? How do they affect read/write latency, storage efficiency, and amplification? Discuss the importance of row-key design and hotspot avoidance.

4. **Fault Tolerance & Coordination:**  
   How does HBase use WAL replay, region reassignment, and coordination via ZooKeeper to handle failures and maintain availability? What happens when a RegionServer crashes?

5. **Scalability & Operations:**  
   Discuss how HBase scales horizontally through region splitting and balancing, how it relies on HDFS for durability, and what administrative actions (snapshots, backups, schema changes, recovery) operators perform in production environments.

### Part 2: Apache HBase (Implementation & Operations) - Answers

1. **Architecture & Data Model:** 
- overall architecture of Apache HBase -
   - HBase stores data in tables, similar to traditional relational databases. Each table consists of rows and columns.
   - Table - An HBase table consists of multiple rows. Table names are Strings and composed of characters that are safe for use in a file system path.
   - Row - A row in HBase consists of a row key and one or more columns with values associated with them. Rows are sorted alphabetically by the row key as they are stored.
   - Row key - unique ID of row in relational DB, but in Hbase Its the key which represents the row. Row keys do not have a data type and are always treated as a byte[ ] (byte array).
   - column - column-family:column-qualifier
   - column family - Column families physically colocate a set of columns and their values, often for performance reasons. Each column family has a set of storage properties, such as whether its values should be cached in memory, how its data is compressed or its row keys are encoded, and others. Each row in a table has the same column families, though a given row might not store anything in a given column family.
   - column qualifier - A column qualifier is added to a column family to provide the index for a given piece of data. Given a column family content, a column qualifier might be content:html, and another might be content:pdf. Though column families are fixed at table creation, column qualifiers are mutable and may differ greatly between rows.
   - Cell - A cell is a combination of row, column family, and column qualifier, and contains a value and a timestamp, which represents the value's version.
   - Timestamp - Values within a cell are versioned Versions are identified by their version number, which by default is the timestamp of when the cell was written If a timestamp is not specified during a write, the current timestamp is used If the timestamp is not specified for a read, the latest one is returned The number of cell value versions retained by HBase is configured for each column family The default number of cell versions is three.
   - To make data easier to manage, HBase splits its tables into regions. Each region is a subset of the data, and the data within each region is determined by the row key range. When a region gets too large, it is automatically split into smaller regions to maintain performance and scalability. Each Region Server stores data for its regions in HDFS data files.
   - storage format (Hfile) - File format for hbase in hdfs. A file of sorted key/value pairs. Both keys and values are byte arrays.
- Physical View - Although at a conceptual level tables may be viewed as a sparse set of rows, they are physically stored by column family. A new column qualifier (column_family:column_qualifier) can be added to an existing column family at any time. The empty cells shown in the conceptual view are not stored at all. However, if no timestamp is supplied, the most recent value for a particular column would be returned.
- How do these elements differ from a traditional relational database - 
- Why is schema design driven by access patterns?

2. **Components & Storage Flow:**
- HMaster - The HMaster is a central component in an HBase cluster and is responsible for managing metadata and coordinating cluster operations. It keeps track of regions, assigns regions to Region Servers, and handles region splits and merges.\ 
The Hmaster exposed methods on Tables, ColumnFamily, Regions.\
- RegionServers - Region Servers are responsible for serving data in HBase. They do the real work. They host a set of regions. Each Region Server can serve multiple regions and is responsible for reading, writing, and managing data within those regions. In a distributed cluster, a RegionServer runs on a DataNode.
- Store - One column familiy inside one region.
- MemStore - in-memory storage. A fast, in-memory storage for writes. It temporarily holds the latest data until it is written to disk. After the data is written to the Write-Ahead Log, it is placed into the MemStore.
- HFiles - Once the MemStore becomes full (after many writes), the data is flushed to disk as HFiles in HDFS. An HFile is the file format that HBase uses to store data in HDFS. It contains a multi-layered index which allows HBase to seek the data without having to read the whole file. The size of those indexes is a factor of the block size (64KB by default), the size of your keys and the amount of data you are storing.
- block cache - is the read cache. It stores frequently read data in  memory. Least Recently Used data is evicted when full.
- Write-Ahead Log (WAL) - The basic idea behind WAL is to record changes in a log before they are applied to the actual storage. contains a sequential record of all changes made to the database. Transactions are not considered complete until the corresponding changes are safely recorded in the write-ahead log.
- How does data flow from a client write to durable storage - 
   1. WAL - The data is first written to the Write-Ahead Log (WAL) to ensure durability and recovery in case of failure.
   2. MemStore - The data is placed into the MemStore (in-memory storage) of the region servers..
   3. Disk - Once the MemStore becomes full (after many writes), the data is flushed to disk (in the region server) as HFiles in HDFS.
- how are reads served from memory and disk structures?
   1. MemStore - HBase first checks MemStore to get the freshest data.
   2. BlockCache - If not found, it will check the BlockCache (a fast cache of recently read data).
   3. Disk - If still not found, HBase will retrieve the data from HFiles on disk.

3. **Performance & Maintenance:**\
How do they affect read/write latency, storage efficiency, and amplification?
- Minor and major compactions -\
   - Minor Compaction - Minor compactions usually select a small number of small, adjacent StoreFiles and rewrite them as a single StoreFile. Minor compactions do not drop (filter out) deletes or expired versions, because of potential side effects. The end result of a minor compaction is fewer, larger StoreFiles for a given Store.
   - Major Compaction - is a single StoreFile per Store. Major compactions also process delete markers and max versions. During a major compaction, the data is actually deleted, and the tombstone marker is removed from the StoreFile. Instead, the expired data is filtered out and is not written back to the compacted StoreFile. When you create a Column Family, you can specify the maximum number of versions to keep. The default value is 1. If more versions than the specified maximum exist, the excess versions are filtered out and not written back to the compacted StoreFile. Reduces amplification.
- MOB storage - The MOB feature reduces the overall IO load for configured column families by storing values that are larger than the configured threshold outside of the normal regions to avoid splits, merges, and most importantly normal compactions. The default is 100 Kb.
- Bloom filters - Just like the HFile indexes, those data structures (when enabled) are stored in the LRU. Bloom filters are stored at the HFile level and evaluated before scanning the disk.
- Caching - improve read performance. Block cache is configurable at table’s column family level. Different column families can have different cache priorities or even disable the block cache. When performing a scan, if block cache is enabled and there is room remaining, data blocks read from StoreFiles on HDFS are cached in region server’s Java heap space, so that next time, accessing data in the same block can be served by the cached block. Block cache helps in reducing disk I/O for retrieving data.
- Importance of row-key design - improve performance. Records in Hbase are stored as a sorted list of row keys according to the lexicographic order and allow fast access to an individual record by its key or fast fetching of a range of data between a given start and end row keys.
- Hotspot avoidance - If a Region reaches that maximal size, it is split into two smaller regions, becoming a hotspot victim because one of these new Regions takes all new records (Limits the write throughput to the capacity of a single server instead of making use of multiple/all nodes in the HBase cluster). There are several solutions: Add salt to the Row Key, use of Hashed Row Key, reverse the row key.

4. **Fault Tolerance & Coordination:**
- How does HBase use those features via ZooKeeper to handle failures and maintain availability:
   - WAL replay - In case of a failure or server crash, when HBase restarts, it replays the WAL entries that were not yet persisted to the HFiles. This replay mechanism ensures that all the writes that were acknowledged but not yet written to the HFiles are restored, maintaining consistency.
   - region reassignment - 
   - coordination - 
- What happens when a RegionServer crashes?

5. **Scalability & Operations:**
- how HBase scales horizontally through region splitting and balancing?
- how it relies on HDFS for durability?
- what administrative actions (snapshots, backups, schema changes, recovery) operators perform in production environments?

### 🔄 Alternatives
Assignment: You are required to research and write a comparative analysis between Hbase and an industry alternative.
- Deliverable: A written summary (minimum 1 or 2 sentences).
- Focus: Compare performance, architecture, and specific "pain points" this tool solves compared to legacy systems or competitors.
- Goal: You must be able to justify why the department uses this tool for our specific environment.

### 🎯 User Story & Scenario
Assignment: Based on your research and understanding of the department's pipeline, define a concrete Use Case for this technology.
- Deliverable: A written summary example/story (two paragraphs approx.).
- Requirement: Describe a real-world scenario (e.g., a specific client requirement) where this technology is the optimal solution.
- Data Flow: Map out the data flow and explain how this tool integrates with other components in the Data Pipeline.

## Wrapping Up :trophy:
Go over your answers with your mentor and clarify any uncertainties. Relate HBase concepts back to the broader data platform.

## Action Items
- Identify HBase topics you want to delve into further.
- Collect a list of real‑world HBase deployments or related technologies.
- Prepare questions for the next mentor Q&A session.

## Recommended Resources
- [Official HBase Reference Guide](https://hbase.apache.org/book.html) – the definitive documentation.
- *Hadoop: The Definitive Guide* (O'Reilly) – chapters on HBase.
