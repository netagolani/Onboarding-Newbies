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
- What are common backend databases? Derby (embedded metastore), MySQL, Oracle, Postgres.

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

4. **Extensibility & Clients:**

## Hive Table Formats

Answer the following questions to understand table formats:

1. **Definition & Role:**  What does a “table format” mean in Hive? How does it differ from table metadata stored in the metastore? Explain the relationship between logical schema and physical file layout.

2. **Common Formats:**  Describe popular formats such as Text/CSV, Parquet, ORC, Avro. How do they differ in encoding, compression, columnar storage, and query performance?

3. **Schema & Tables:**  Explain the difference between managed and external tables, including ownership, lifecycle, and storage location semantics. How does the metastore map logical tables to physical data in storage systems like HDFS or object storage?

4. **Integration with Storage:**  How do table formats map to physical storage (directories, files)? What conventions does Hive use for partitions, buckets, and file naming?

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

