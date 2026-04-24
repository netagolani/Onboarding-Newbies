# Catalogs & Table Formats :

## Overview
This session dives into the metadata layer that sits above raw files in a data
lake or warehouse.  Before we talk about individual systems, start by thinking
about the big picture: what is a *data warehouse* versus a *data lake* versus a
*lakehouse*, and why do teams care about catalogs and table formats in each
case?  (Hint: consistency, governance, and performance are the common threads.)

We’re not going to install or run Spark/Trino/etc. (If you don’t know what those are, no worries you will soon.);
the material stays at themetadata level.  That said, good formats enable optimizations such as
partition pruning, predicate push‑down, and efficient file compaction, all of
which have a dramatic impact on execution even though we won’t be executing
anything here.

**We’ll examine why catalogs exist, how they differ from Hive’s metastore
(which is itself just one implementation of a catalog), and the design goals of
modern table formats such as Iceberg, Delta Lake, and Hudi.  Examples of
catalog implementations include Hive Metastore, AWS Glue, Databricks Unity
Catalog, and even simple relational databases;

## Goals
- Clarify what catalogs and table formats actually manage and why teams put
  them on top of object storage.
- Sketch the difference between a warehouse, a lake, and the newer lakehouse
  idea so you have context for why metadata matters.
- Learn about emerging formats that implement ACID, schema evolution, time
  travel and other optimizations, and why those features make life easier for
  query engines.
- Build on previous lessons by focusing on interoperability and metadata
  management rather than specific execution engines.

:warning: **Note:**
- Keep the focus on metadata and format, and on optimizations derived from those rather than on query engines or execution.
- Ask your mentor if you're unsure about scope.

### ⏳ Timeline
Estimated Duration: 1 Day
- Day 1: Learn the concepts of catalogs and table formats; spend the day.
    - Have a Q&A session the same day

## Core Concepts

1. **Data warehouse / lake / lakehouse:**  What are the defining
   characteristics of each?  Why do architects care about a separate metadata
   layer in a lakehouse versus a traditional warehouse?

2. **The Concept Of Catalog**  Describe the purpose of a metadata catalog.  How
   does it compare to Hive Metastore (hint: the metastore *is* a catalog)
   and why might systems introduce separate catalog layers (e.g. AWS Glue,
   Databricks Unity Catalog, in‑house catalog backed by PostgreSQL)?

3. **Catalog Architecture:**  Explain typical components of a catalog service
   (namespace management, table and partition metadata, permissions).  What
   backend storage is used?  Is the catalog itself just a database, or does it
   also manage pointers to objects in a blob store?

4. **Table Formats Overview:**  Define what a table format is in the context of
   a data lake.  How do formats like Iceberg, Delta, and Hudi differ from
   simple Hive/Parquet tables?  What features do they add ?

5. **Metadata & Transaction Log:**  How do modern formats store their own
   metadata?  Discuss the concept of a transaction log or manifest file, and
   the distinction between file level metadata (e.g. Iceberg data file footers)
   and catalog entries.  When would you even need to think about files if the
   catalog abstracts them away?  

6. **Interoperability & Ecosystem:**  Describe how catalogs and formats enable
   multiple compute engines to work on the same data (Spark, Trino, Flink).
   Why is standardization important?  What role do open specifications
   (e.g. Apache Iceberg spec) play?


## Core Concepts - Answers

1. **Data warehouse / lake / lakehouse:**  
- What are the defining characteristics of each?\
Data Warehouse:
   - Considered of being OLAP database.
   - Mainly stores structured data, schema on write.
   - expensive to scale out in storage and computation.
   - Coupled to the warehouse's compute engine.
   - high performance

Data Lake:
   - Lower cost
   - Handle unstructured data
   - Decoupled to compute engine
   - lack of performance and ACID guarantees

Data Lakehouse:
   - Stores data as data lake
   - compute engines as data lake
   - But has a table format
   
- Why do architects care about a separate metadata layer in a lakehouse versus a traditional warehouse?
So it won't be coupled to the compution engine as traditional warehouse.

2. **The Concept Of Catalog**  
- Describe the purpose of a metadata catalog.  How does it compare to Hive Metastore? (hint: the metastore *is* a catalog)\
Metadata catalog purpose is to track table location. The catalog is the central location to find existence of a table and additional information about each table (table name, schema, where the data stores)
In hive, the metastore is the catalog. It contains a mapping of table name -> set of directories, while in modern catalog table name -> location of the table's most recent metadata file.
- Why might systems introduce separate catalog layers (e.g. AWS Glue,
Databricks Unity Catalog, in‑house catalog backed by PostgreSQL)?\
To provide flexibility and decouple the actual data. To discover the same data in different ways, manage multipule schemas to the same data for several users or uscases without stores it in different ways. To allow a catalog which could integrate with different compute engines.


3. **Catalog Architecture:**  Explain typical components of a catalog service
(namespace management, table and partition metadata, permissions).\
Namespace - logical entity in a catalog which contains tables and views. Similaer to schemas or databases. Can be structures in a nested hierarchy - a.b.c.d\
table metadata - the catalog can have a set of system tables that stores metadata of each table. Tables such as snapshots, partitions, etc.
What backend storage is used?  Is the catalog itself just a database, or does it also manage pointers to objects in a blob store?
Its can be either a database which tracks and maintains pointers to the latest versions of the table metadata layer. Support ACID transactions. Or can be a file catalog which used to call version-hint.txt stored in the system under the metadata folder.

4. **Table Formats Overview:**  Define what a table format is in the context of
a data lake.  How do formats like Iceberg, Delta, and Hudi differ from
simple Hive/Parquet tables?  What features do they add ?\
Older table formats like hive table format are based on the contents of directories and path of the files design. Incontrast to modern file formats which are based on individual data files. Defining tables as list of files. Providing metadata for engines information on which files make up a table and not directories.

5. **Metadata & Transaction Log:**  How do modern formats store their own metadata?  Discuss the concept of a transaction log or manifest file, and the distinction between file level metadata (e.g. Iceberg data file footers) and catalog entries.\
Modern formats store their own metadata with a metadta layer which combines several components for example in icberg the metadata layer includes manifest files, which keep track of the data files (delete files, statistics and indexes about the data), manifest lists presents Iceberg table that containes a list of all the manifest files and metadata files which store metadata about an Icberg table at a certain point in time (schema, partition information, snapshots). Each time a change is made to an Iceberg table, a new metadata file is created and is registered as the latest version of the metadata file atomically via the catalog. The immutable metadata files considered as transaction logs which provides atomicy.
When would you even need to think about files if the catalog abstracts them away?
The catalog collects all the last pointers to the files of the metadata layer
and keep tracking them like a phone book. All the files of the metadata layer are files we're getting the metadata or when modifying adding new metadata files.

6. **Interoperability & Ecosystem:**  Describe how catalogs and formats enable
multiple compute engines to work on the same data (Spark, Trino, Flink).
Catalogs are seperated the physical storage from the metadata layer providing the single source of trouth with an access layer to multiple compute engines.
Why is standardization important?  What role do open specifications (e.g. Apache Iceberg spec) play? When there is a standatization, a component which all the compute engines first connect to find their data its became agnostic and flexible to multipule systems one way to interact and use for several systems which allows to combine all and with different compute engines.


## Wrapping Up :trophy:
Review your answers with your mentor, focusing on how catalogs and formats enable a consistent data platform across tools.

## Action Items
- Identify catalogs or formats you’d like to try in practice.
- Prepare questions for the mentor Q&A session.
- Link these ideas back to the [intro chapter](../Chapter%200%20-%20Intro/1%20-%20Big%20Data%20Core%20Concepts.md).
### 📚 Resources
Use the resources listed below and practice searching the internet for questions not answered by the provided documentation.
- [Apache Iceberg Definitive Guide](http://103.203.175.90:81/fdScript/RootOfEBooks/E%20Book%20collection%20-%202024%20-%20F/CSE%20%20IT%20AIDS%20ML/Apache%20Iceberg%20(2024).pdf) Use this resource only for warehouse vs lake vs lakehouse (Iceberg will be learned on a different day)

