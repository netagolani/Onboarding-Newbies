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
