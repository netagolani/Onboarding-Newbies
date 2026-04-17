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
When a failure of a region server happens, other region servers race to create a znode called lock inside the dead region server's znode that contains its queues. The region server that creates it successfully then transfers all the queues to its own znode, one at a time since ZooKeeper does not support renaming queues. After queues are all transferred, they are deleted from the old location. The znodes that were recovered are renamed with the ID of the slave cluster appended with the name of the dead server.

5. **Scalability & Operations:**
- how HBase scales horizontally through region splitting and balancing?
- how it relies on HDFS for durability? The WAL and the HFiles are wrriten in HDFS, so when there is a failover the durability of keeping the hsbase managing files in hdfs helps the recovery mechanism.
- what administrative actions (snapshots, backups, schema changes, recovery) operators perform in production environments?
- Backups
   - Full shutdown backup - the cluster is down but the backup is fully covered because there are no changes during the backup.
   - Live cluster backup - the copytable used for copying data from one table to another on the same cluster or to another cluster.
- Snapshots - HBase snapshots allow you to clone a table without making data copies. Without using snapshots, the only way to backup or clone a table was to use the provided CopyTable or ExportTable tools (it means no read or writes and degrade performance). Used for data migration and recovery from data corrupted. From a snapshot you can create new tabe `clone_snapshot`, or restore the original table `restore_snapshot`. To clone a table to another cluster, you export the snapshot to the other cluster and then run the clone operation.
- Schema changes - When changes are made to either Tables or ColumnFamilies, these changes take effect the next time there is a major compaction and the StoreFiles get re-written. Update are made with `put` command.

### Q&A - Answers
1. Big table and Hbase invention: Big table was invented for managing structured data at Google. Bigtable has achieved scalabilty, high performance, and high availability, it designed to reliably scale out to petabytes. Bigtable was used by a lot of google products. The first Hbase was created as a Hadoop contribution. It carries all the features of the original Google Big table paper like the Bloom filters, in-memory operations and compression. Apache HBase became its open-source implementation.
2. HBase used for massive amount of data which needs to scale horizontally with fast performance for accessing data. Usecases examples:
   - Application logs for diagnostic and analysis.
   - Genome sequences and the disease history of people in a particular demographic.
   - Head-to-head competition histories in sports for better analytics and outcome predictions
3. Joins in HBase: There are no joins in hbase. You have to do it on your own by either denormalizing the data before writing to HBase, or doing the join between tables in the application or MapReduce case.
4. Quering operations in HBase:
   - Get - returns attributes of a specified row.
   - Scan - iteration over multipule rows for specified attributes.

Instead of retireving all the data and filter it in the client side, HBase allow filters which applied logic on the server-side. Filtering reducing overhead. Most efficient features:
   - SingleColumnValueFilter - like where clause. Filtering by value in a specific column.
   - PrefixFilter - Filter rows based on rew key prefix.
   - ColumnPrefixFilter - Filtering columns with specific prefix.
5. The .META. is stored in a regionServer like any other table (it ever can be split into muiltiple regions). the META table structure is -\
Key: region start key, region id. Value: region server.\
It can be replicated by maintains read-only copies of the META table by configuring a set of properties in cloudera manager.
6. Hbase doesnt have to run over hdfs, it can also run over s3 or it can run in a standalone mode.
7. connction registery - Client internally works with a connection registry to fetch the metadata needed by connections. This connection registry implementation is responsible for fetching the following metadata: Active master address, Current meta regions locations, Cluster ID.\
There are three types of connection registry - 
   - MasterRegistry - deprecated.
   - RpcConnectionRegistry - hbase+rpc.
   - ZKConnectionRegistry - hbase+zk.
8. What is thrift? When and why to use the thrift method? Apache Thrift is an open-source RPC framework. It has several benefits -\
   - Thrift Filter Launguage - performs server-side filtering when accessing Hbase over Thrift.
   - Thrift IDL - the interface definition launuage defines both the data structure and the interfaces for the services that communicate across different systems.
   - Performance - Thrift provides compact and efficient binary serialization incontrast to rest which is more human readable making it more cpu intensive.
   - Versioning - Thrift has mechanism  for versioning data which is very helpful in distributed environment where your service interfaces may change, but you cannot atomically update all your client and server code.

In conclusion, you may use Thrift over Rest in disributed systems which transfers big amounts of data with more efficient mechanisms and interface level for multipule different services.
9. Hbase native API - to use hbase CLI we nned to SSH into an HBase node and use the HBase shell. Example of get help on a specific command: `hbase> help "create"`. More commands can be: `hbase hfile` to diagnose information about specific hfile, `scan <table_name>`, `get <table_name>, <row>`, `<drop <table_name>`, `<disable <table_name>` (for change settings and then enable).   
10. Where tha Write-ahead log file? It exists in a regionServer in the /hbase/WALs/ directory wrriten to HDFS which means its replicated (important for failovers), with subdirectories per RegionServer.
11. What triggers a flush from memStore to disk?
   - Reaching memStore size - hbase.hregion.memstore.flush.size
   - Reaching memStore usage limit - hbase.regionserver.global.memstore.upperLimit The flush order of region's memStore will be in descending order until it gets slightly below hbase.regionserver.global.memstore.lowerLimit.
   - Reaching the number of WAL log entries with the value hbase.regionserver.max.logs, MemStores from various regions will be flushed out to disk based on time (the oldest memStore) to reduce the number of logs in the WAL.
12. What is the components which groups tables? A namespace.
13. What are the data types which can be stored in Hbase? Everything in HBase tables is stored as a byte[ ] There are no types. 
14. Failover of region server - 
   - zookeeper noticed there are no heartbeats.
   -  The Hmaster splits the WAL into separate files and stores them in available region servers. 
   - Each region server replays the WAL, to rebuild the memstore for that region.
15. Tombstone Marker - Hbase files are immutable so it can't be modified the Hfile as deleted. Instead it adds another record of deleted keys - tombstone markers which marks it as dead. I imagine it as a new version which tells that the cell/column/etc is dead. When there is a Scan or Get method it knows to skip the deleted cells. The tombstone markers and the values themselves are deleted in major compaction.
16. Does in every major compaction the tombstone will be deleted? Yes and also the actual data itself, unless there is a major compaction on other HFiles because of the compaction policy.
17. triggers of compactions - can be automatically by number of HFiles, time and data size. It is set in the compaction policies. It can also be triggered manually for specific needs.
18. According to which logic object compactions are made? Regions.
19. Which Hfiles are stored together? Hfiles which belongs to the same Region.
20. Major compaction resposibilies - Merge all the HFiles of a region to on single Hfile, reduce seeks to disk, delete expired and deleted cells (according the tombstone markers).
21. When does bloom filter applied and where it stores? Bloom filters provided in get operations to reduce the number of disk reads (do not work with scans). The Bloom filters are stored in the metadata of each HFile and never need to be updated. When an HFile is opened because a region is deployed to a RegionServer, the Bloom filter is loaded into memory.

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
