# Hive Metastore & Table Format :

## Overview
Today’s session zeroes in on two foundational pieces of Hive: the metastore that holds metadata and the table formats that define how data is structured on disk. We will avoid any discussion of Hive’s execution engines (MapReduce, Tez, etc.) or query processing. The goal is to understand the storage and metadata layers that other tools in the ecosystem rely on.

**Focus only on metadata management and table/format semantics.**

## Goals
- Understand what the Hive Metastore is and why it exists.
- Learn how Hive tables are defined and how formats describe their physical layout.
- Practice self-directed study and time management.

:warning: **Note:**
- Independence is essential; plan your study day accordingly.
- If you can’t explain a concept clearly, revisit the documentation.
- Review the [Exercise](#exercise) before diving into research.
- Ask your mentor for clarification on scope if needed.

### ⏳ Timeline
Estimated Duration: 1 Day
- Day 1: Learn the concepts of Hive both metastore and table format; spend the day.
    - Have a Q&A session the same day

## Hive Metastore

Answer the following questions to explore the metastore:

1. **Purpose & Function:**  What is the Hive Metastore and what types of metadata does it store (databases, tables, columns, partitions, locations, statistics)? Why is a centralized metadata service necessary in a distributed data platform?

2. **Architecture & Backend:**  Describe how the metastore is implemented as a standalone service backed by a relational database. What are common backend databases, and how does the service scale and handle concurrent clients?

3. **Schema & Tables:**  What are the key tables in the metastore schema (e.g. DBS, TBLS, SDS, PARTITIONS)? How do they relate to Hive objects?

4. **Extensibility & Clients:**  How do external engines such as Apache Spark, Trino, and other tools interact with the metastore? What APIs and protocols are used?

5. **Administration:**  What are common administrative tasks (backup, schema upgrades, migration, repair)? What happens if the metastore becomes unavailable, and why is it considered a critical dependency in data platforms?

## Hive Metastore

1. **Purpose & Function:**
- What is Hive? A framework for data warehouse system that enables analytics at massive scale and factilitates reading, writing in distributed storage using SQL. Hive was created to make it possible for analysts with SQL skills.
- What is Hive Metastore? HMS is database where metadata is stored. Metadata repository. Responsible for the virtualization of data collections in HDFS as tables.
- What types of metadata does it store?
    - databases
    - Table's metadata - list of columns, owner, storage (location). SerDe metadata (implementation calass of serializer and deserializer).
    - Partition - Each partition can have its own columns and SerDe and storage information. This facilitates schema changes without affecting older partitions.
- Why is a centralized metadata service necessary in a distributed data platform?
    - common data access pattern - HMS is the bridge between the storage layer and the compute layer.
    - Single Source of Truth - of the metadta
    - Decoupling -  enables the whole system to scale independently by decoupling  the metadta, computing and storage which are decouple to one technology or platform.

2. **Architecture & Backend:**
- Describe how the metastore is implemented as a standalone service backed by a relational database? By default, the metastore is run in the same process as the Hive service. It can run in a standalone (remote) process.\
Hive metastore consists of two units:
    - service - metastore access to other Hive services.
    - Disk storage - Hive metadata storage

In the remote meta store mode we have three separated components:
    - Hive Service JVM - java virtual machine of the drive, the component which accepts queries (JDBC/ODBC)
    - Metastore Server JVM - Its an individual JVM which not in the Hive service which can scale up (provides availability) and handle concurrent clients of other processes using Thrify Network API.
    - remote database
- What are common backend databases? Derby (embedded metastore), MySQL, Oracle, Postgres, Databricks.

3. **Schema & Tables:**\
What are the key tables in the metastore schema?
- SDS - Storage Descriptor. Join with DBS and TBLS Tables.\
SD_ID , CD_ID , INPUT_FORMAT  , IS_COMPRESSED , IS_STOREDASSUBDIRECTORIES , LOCATION , NUM_BUCKETS , OUTPUT_FORMAT , SERDE_ID.
- TBLS - Tables. Join with DBMS, SDS, and Partitions table.\
TBL_ID , CREATE_TIME , DB_ID  , LAST_ACCESS_TIME , OWNER , RETENTION  , SD_ID , TBL_NAME  , TBL_TYPE ,VIEW_EXPANDED_TEXT , VIEW_ORIGINAL_TEXT , LINK_TARGET_ID.
- DBS - Databases. Join with SDS and TBLS tables.\
DB_ID , DESC , DB_LOCATION_URI , NAME , OWNER_NAME , OWNER_TYPE.
- PARTITIONS - partitions. Join with DBS and TBLS tables.\
PART_ID , CREATE_TIME , LAST_ACCESS_TIME , PART_NAME , SD_ID , TBL_ID , LINK_TARGET_ID.

4. **Extensibility & Clients:**\
How do external engines such as Apache Spark, Trino, and other tools interact with the metastore? Metastore server and client communicate using Thrift Protocol. The client configurations parameters are: hive.metastore.uris=<thrift://<host_name>:<port>>, user, authentication type (kerberos), hive.metastore.warehouse.dir=<base hdfs path>
What APIs and protocols are used?
Thrift API (RPC), JDBC ,Rest API, Java client API.

5. **Administration:**
- What are common administrative tasks (backup, schema upgrades, migration, repair)?
    - backup - backup periodiclly the metastore database.
    - schema upgrades- The metastore schema version needs to be compatible with hive binaries (hive cli, api) that are going to access the metastore. Upgrade schema is done by Hive schema tool to much hive version schema to the metastore schema.
    - migrations - move customers from hive services to trino.
    - repair - REPIAR TABLE is used to update the metadata in the Hive metastore to reflect the current state of the partitions in the file system. This is particularly necessary for external tables where partitions might be added directly to the file system (such as HDFS or Amazon S3) without using Hive commands.
- What happens if the metastore becomes unavailable, and why is it considered a critical dependency in data platforms?
If hive metastore is unavailable:
    - queries that need metadata wil fail.
    - New sessions that require table schemas cannot start properly.
    - Spark, trino and impala wont be able to read schema/catalog information, again quries will fail.
    - operations that consult the metastore will fail.
    - Jobs that already have fully-resolved plans: running MapReduce/Spark tasks that were compiled before the outage and do not need metadata at execution time may continue to run to completion.
    - Table creation, alteration, drop, partition operations: blocked — cannot persist or retrieve metadata.
    - ACID operation will fail - require metastore for transactional state.

## Hive Table Formats

Answer the following questions to understand table formats:

1. **Definition & Role:**  What does a “table format” mean in Hive? How does it differ from table metadata stored in the metastore? Explain the relationship between logical schema and physical file layout.

2. **Common Formats:**  Describe popular formats such as Text/CSV, Parquet, ORC, Avro. How do they differ in encoding, compression, columnar storage, and query performance?

3. **Schema & Tables:**  Explain the difference between managed and external tables, including ownership, lifecycle, and storage location semantics. How does the metastore map logical tables to physical data in storage systems like HDFS or object storage?

4. **Integration with Storage:**  How do table formats map to physical storage (directories, files)? What conventions does Hive use for partitions, buckets, and file naming?

## Hive Table Formats - Answers

1. **Definition & Role:**
- What does a “table format” mean in Hive?\
Table format allows querying tables in a specific format (a path format for example). 
- How does it differ from table metadata stored in the metastore? The table metadata stored metadata in a table, their are databases, tables rows and columns. While in table format their  is a path which lead to several files which described together a table.
- Explain the relationship between logical schema and physical file layout.\
The logical schema needs to be with the same order columns of the file layout. If not, when you query the table it won't succeed to present the misordered columns.

2. **Common Formats:**  Describe popular formats such as Text/CSV, Parquet, ORC, Avro. How do they differ in encoding, compression, columnar storage, and query performance?
- Text/CSV - the default file format. ESCAPED BY `<delimiter>`, Has a custom NULL format (default is '\N'). All binary columns assumed to be base64 encoded.
- Parquet - wide columnar format of flatted nested data structures. Supports compression and encoding schemes specified per column level.
- ORC - supports ACID transaction. Columnar format arranges columns adjacent within the file for compression. It was designed to overcome limitations of the other Hive file formats. An ORC file contains group of row data called stripes. Eacg stripe holds index data, row data and stripe footer which contains a directory of stream locations.
- Avro - apache avro is a row based storage format. Has a metadata header with json scheme, compression codec and sync maker which tells how to split the data.

3. **Schema & Tables:**
- Explain the difference between managed and external tables, including ownership, lifecycle, and storage location semantics?\
Managed tables hive owns the data. The data, properties and layout can only be changes via hive. When youre drop a table it deletes also it's files. Stores under the path /user/hive/warehouse/databse/tablename. In contrast to external table which can managed by processes outside of Hive. Fille would remain even if the table is dropped.

4. **Integration with Storage:**
- How do table formats map to physical storage (directories, files)? With SerDe which defins in the metastore for each table. For example, for a table of parquets will be in the metastore a SerDe of parquet which defines the OutputFormat that needs to be wrriten as parquet files in directories from java objects. By doing a serilize method.
- What conventions does Hive use for partitions, buckets, and file naming?
partitions needs to be by a partition key - a column in the table. If it is partitioned by multipule columns it will defined as subdirectories. Buckets are presented as numeric files - 0000001_0, 0000002_0.

## Hive Table Formats - Q&A Answers
1. What is partitions? How hive supports it?
Partitioning is a design technique of dividing a table into smaller and managable pieces called partitions. partition key is the determinator of how the data will distributed. You can run queries via hive which creates tables with partitions, the partition metadata will be stored in the HMS.
2. bucketing in hive vs partitions:\
Paritions in hive - 
    - stores each partition in a seperate directory in HDFS. 
    - A partition is typically based on the value of a column.
    - reduce the amount of data which needs to be scanned that filter on the partition column.

Bucketing in hive - 
    - divides data into a fixed number of equal-sized files which called "buckets".
    - The buckets are based on the hash value of a specific column.
    - Used for distribute data evenly across a set number of files for better query performance (for example, when joining large datasets).
    - Defined by the `CLUSTERED BY (column_name) INTO XX BUCKETS`.
    - The result of  a hash function to the bucketed column determines which bucket a record will go into.

In conclution, in bucketing the number of buckets - files, is fixed so its not effected by the data. In contrast to partition which determines in which bucket to put the data.

3. what happened when the metastore data base is down and the service is be accessed only up? Can we still connect to the data?\
The service has a cache store so the metastore's data will be available only if the data exists in the cache.
The size of the metastore cache can be restricted by a combination of cache white list and black list patterns. So only if the table considered as a white list it will be available.
4. In case the scheme is invalid? What will be the query output in parquet, csv, text.
Each file format acts different:
    - parquet - Saves its own schema in the file. column mistmatch: rename a column in hive but not in the parquet file returns NULL. Type mistmatc - the query will fail. Extra columns - ignored.
    - Text/CSV - text based formats. do not save column names or datatypes. column mistmatch - return NULL for the column. type mistmatch - return NULL. Extra columns - also NULL. 
5. What is SerDe?
- SerDe is short for Serializer/Deserializer.
- A SerDe allows Hive to read in data from a table, and write it back out to HDFS in any custom format.
- Anyone can write their own SerDe for their own data formats.
- Builtin SerDes can be Avro, ORC, Parquet, CSV, etc.
- Serde.deserialize() perform deserialization according the InputFormat to read. By the Inputformat it knows how to show files as tables (java objects which combined to rows)
- Serde.serialize() perform on the deserialized object according the OutputFormat to write. Knows how from java objects to write it as files in hdfs under paths.

### 🔄 Alternatives
Assignment: You are required to research and write a comparative analysis between Hive table format and HMS and an industry alternative.
- Deliverable: A written summary (minimum 1 or 2 sentences).
- Focus: Compare performance, architecture, and specific "pain points" this tool solves compared to legacy systems or competitors.
- Goal: You must be able to justify why the department uses this tool for our specific environment.

### 🎯 User Story & Scenario
Assignment: Based on your research and understanding of the department's pipeline, define a concrete Use Case for this technology.
- Deliverable: A written summary example/story (two paragraphs approx.).
- Requirement: Describe a real-world scenario (e.g., a specific client requirement) where this technology is the optimal solution.
- Data Flow: Map out the data flow and explain how this tool integrates with other components in the Data Pipeline.


## Wrapping Up :trophy:
Review your answers with your mentor and make sure you can articulate how the metastore and formats enable interoperability across Hadoop tools.

## Action Items
- Identify areas of metadata or format behavior you want to explore further.
- Prepare questions for the mentor Q&A session.
- Continue linking these ideas to other chapters as part of the Day 01 challenge.

## Recommended Resources
- [Hive Metastore Documentation](https://cwiki.apache.org/confluence/display/Hive/Metastore+Overview)
- [Hive Language Manual – Table Formats](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)

