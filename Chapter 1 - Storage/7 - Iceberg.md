# Apache Iceberg

Now that you are familiar with the concepts of catalogs and the metastore,
and understand the critical separation between how data is stored versus how it is logically organized,
it is time to move from theory to the practice of the modern data world.

Meet Apache Iceberg – the table format we use as the storage layer in our lakehouse.
designed to solve the consistency and performance "pains" of legacy directory-based systems.

###⏳ Timeline
Estimated Duration: 2 Days
- Day 1: Independent research and deep dive into the foundations of Iceberg.
- Day 2:
  - (Morning): First Q&A.
  - (End of Day): Question Answering & Final Q&A.

### 📚 Resources
Use the resources listed below and practice searching the internet for questions not answered by the provided documentation.
- [Apache Iceberg Official Docs](https://iceberg.apache.org/docs/latest/#documentation)
- [Apache Iceberg Definitive Guide](http://103.203.175.90:81/fdScript/RootOfEBooks/E%20Book%20collection%20-%202024%20-%20F/CSE%20%20IT%20AIDS%20ML/Apache%20Iceberg%20(2024).pdf)
- [Meap Book - Apache Iceberg](./asstes/Architecting_an_Apache_Iceberg_Lakehouse_v3_MEAP.pdf)

### Guide Questions❓
Please use these questions as a guide for your research, dive in, and deepen your understanding of all concepts.
1. What is Apache Iceberg? 
   Explain the problems it solves compared to Hive tables (schema evolution, partitioning, consistency, performance).


2. Describe the Apache Iceberg table architecture. 
   Explain metadata files, manifest files, data files, and snapshots and how they relate to each other.


3. What is an Iceberg catalog, and what is its role? 
   Explain what a catalog manages (table namespace, metadata pointers, commits), why it’s required, and how it differs from a metastore. 
   Mention common catalog implementations.


4. How does Iceberg handle concurrent reads and writes? 
  Explain snapshot isolation, atomic commits, optimistic concurrency control, and conflict detection.


5. What maintenance operations does Iceberg require, and why? 
   Discuss compaction, snapshot expiration, orphan file cleanup, and metadata cleanup.

### Guide Questions - Answers
 1. What is Apache Iceberg? 
   Explain the problems it solves compared to Hive tables (schema evolution, partitioning, consistency, performance).\
   Apache Icberg is a modern table format Apache solution which came out to solve hive metastore challenges:
   - consistency - updates acrross multuiple partitions are done atomically, so data is consistent to end users. They see the data before the update or after the update but not in between.
   - performance - icberg avoids excessive file/directory listing bottleneck while query planning because of its smart matadata layer architecture which improves peformance.
   - schema evolution - update schema in hive could be inconsistent or unsafe and update table's partitions resualts to rewrite the table. Icberg create new metadata files which allow to change the schema and how the table is partitioned atomically without the need of rebuilt the data. You can also see how the data was look like in the historical scheme or pratitions.
   - partitioning - can be changes as I said, not defined in the storage as directories. Only by the metadata.

2. Describe the Apache Iceberg table architecture. 
   Explain metadata files, manifest files, data files, and snapshots and how they relate to each other.\
Apache Iceberg table architecture:\
Consistes of data layer, metadata layer and Icberg catalog.\
- Data Layer - consists of the actual data, the data files (parquet, ORC, Avro), deleted files (track which records in the dataset have been deleted) and puffin files (statistecs and indexing information).
- Metadata Layer - 
   - manifest files - consists of metadata on the data layer. Written in Avro file format.
   - manifest lists - a snapshot of an Iceberg table, contains a list of all the manifest files. (their locations, partitions it belongs, upper and lower bounds for partition).
   - metadata files - consists metadata at a cetain point in time (table's schema, partitions, snapshots, and which is the current one). In each update of the table a new snapshot is added to the new metadata which holds the snapshots locations.
- Catalog - the first step which any engine will go to understanf where to read/write. It holds the location of the relevant metadata file.

3. What is an Iceberg catalog, and what is its role? 
   Explain what a catalog manages (table namespace, metadata pointers, commits), why it’s required, and how it differs from a metastore. 
   Mention common catalog implementations.\
   As I said, an Iceberg catalog is the first step in any operation. It points or references to the requested table by pointing the table's current metadata files. Every operation is done atomically after the pointer of the metadata file is updated in the catalog (commits, creating a table). In contrast to HMS which consist the path to the table and other metadata. Its only hold a pointer to the first step in the metadata layer which consist of files (against paths). The metadata are immutable files which created in every new updates providing flexbility by not being coupled to how the files are all organized and partitioned.

4. How does Iceberg handle concurrent reads and writes? 
  Explain snapshot isolation, atomic commits, optimistic concurrency control, and conflict detection.\
  - Optimistic concurrency control - handled by the catalog. Enable ACID guarantees. Checks for conflicts will be only when necessary, aiming to minimize locking and improve performance. Transactions can either commit or fail.
  - Snapshot isolation - Every update that is being done to the data creating a new snapshot. The catalog's pointer to the new snapshot will synchronized only by the when the transaction has succeed. Otherwise, other readers/writes get the catalog current metadata file location. 
  - atomic commits - transactions can either commit or fail.
  - conflict detection - The engine runs a check at the end of the write transaction to ensure there are no write conflicts and then updates the catalog with the value of the latest metadata file.

5. What maintenance operations does Iceberg require, and why? 
   Discuss compaction, snapshot expiration, orphan file cleanup, and metadata cleanup.
   - compaction - automating compaction (can be with airflow). There are three types of compactions. bin packing - combine files to bigger files with no global sorting (fast but not fully ordered). sorting - by one or more fields (efficient to query, longer compaction). zorder - sort by multipule fields equally and weighted (can improve even more read time, but longer compaction).
   - snapshot expiration - to optimize storage but prevent from time travel to an expired snapshot. Snapshots will get deleted together during the expiration transaction. It can be by a particular timestamp and by a specific snapshot id
   - orphan files - remove orphan files. Not in periodically way because it can be an intensive process. Orphan files are untracked files which where written by failed jobs. This action of removing orphan files is also optimizes storage. A special procedure will look at every file in your table’s default location an assess whether it relates to active snapshots. 
   - metadata cleanup - Each change to a table produces a new metadata file to provide atomicity. Old metadata files are kept for history by default. To automatically delete older metadata files, set write.metadata.`delete-after-commit.enabled=true` in table properties. his will only delete metadata files that are tracked in the metadata log and will not delete orphaned metadata files.

### Additional Questions

1. Do the metadata and the actual data have to be in the same place? I didnt understand the questions. But the thing is that the metadata exists under the metadata folder seperated from the data itself. For example, there is /orders/data and orders/metadata for the orders table. When the engine want to do a request over the table order, it goes to /orders/metadata/version-hint.txt. The only relevent content in this file will be an integer of the metadata file the engine needs to read. So I assume it has to be in the same directory of /orders/metadata.

2. Does the metadata has to be stored in HDFS/S3? No.

3. What are the types of metadata in Iceberg? manifest files, manifest lists, metadata files as I described earlier.

4. What file formats are the metadata files?
- manifest files - Avro file format. <hash>.avro
- manifest list - Avro file format. snap-<hash>.avro
- metadata files -  Json format. (v1.metadata.json, for example)

5. What is snapshot? A manifest list in.

6. Where the catalog exist? 
- In s3/hdfs there's a file called version-hint.text in the table's metadata folder.
- It can also be the hive metastore as the catalog.
- JDBC catalog - an external database which has all the pointers in the DB as a catalog.

7. What is JDBC catalog? Java Database Connectivity. Data stores for Iceberg's catalog interface. Such as MySQL or PostgreSQL (must support atomic transactions).

8. What information is stored in the DB catalog?\
There are two types of catalogs: file-based catalogs and service-based catalogs. File-based catalogs are less good for production environments in scenarios with multiple writers to tables which can cause inconsistency. In the file based catalog there's a file consist an integer of the metadata file version which known in the metadata path while in a db based catalog there is a map between the the table and the pointer of the current metadata file.

9. Describe the scheme of the catalog DB.\
A mapping between the table name to the path of the current pointer.


10. Pros of REST over JDBC/Thrift.
- requires fewer packages and dependencies compared to other catalogs. It just uses standard HTTP communication.
- flexibility - any service is capable to handle restful requests and responses. The data store of the service can be many different things.
- cloud agnostic - can use as a multicloud catalog.

11. What actions are done to maintain the catalog?
Expire snapshots, remove old metadata files, delete orphan files, compactions, rewrite manifests.

12. What are common catalogs?\
HMS, AWS glue catalog, JDBC catalog, Rest catalog

13. Iceberg v2 vs v3.
- positional deletes in v2 are deprecated in v3 and instead its binary deletion vectors (Encoding deleted positions in a bitmap. A set bit at position P indicates that the row at position P is deleted.)
- row lineage tracking - metadata fields that allow engines to detect row-level changes between commits (in v3)
- variant type - a flexible column type for semi-structured json data, allowing untyped data without strict schema enforcement.

### Q&A Answers
1. Metadata in manifest list contains an array of structs, each sruct keeping track of a single manifest file. Each struct contains:
   - location of data file
   - partitions it belongs
   - upper and lower bounds for partition columns of the data file
2. Metadata in the metadata.json contains an Iceberg table at a certain point in time:
   - table's schema
   - partition information
   - snapshots
   - which snapshot is the current one
3. Migrating HMS to create iceberg tables:
Set <iceberg.engine.hive.enabled=true> in its Hadoop configuration. (version-hint.xml). And add to the tables table a "metadata_location" column.
4. What are the pros of rest catalog from others? The REST catalog defines a standart HTTP API specification for catalog operations, eliminating the need for engine-specific catalog implementations. Becomes the recommended approach because:
- vendor neutrality - works consistently accross spark, trino, flink.
- simplified client configuration - single HTTP endpoint instead of engine specific settings.
- RESTful interface with JWT authentication support.
- Extensibility - can add features like caching, auditing, or access control.
5. Why S3 integrates better with iceberg rather then HDFS? Because both of them are doing snapshots which is a waste of data.
6. How to recreate the catalog if the DB failed? 
- find the latest metadata.json file (in hive its easy - the latest version of the metadata.json, in s3 its by the <LastModified> timestamp).
- re-register the table using <register_table> a producer of spark which required the values of the table name and the metadata_file.
7. What the name of handling confilcts in Iceberg? optimistic concurrency control - conflicts are detected at commit time. The transaction flow goes like this: 
- Reads the current table snapshot.
- Write new data files.
- Load the table’s latest metadata.
- Data check conflict.
- Generate new metadata files.
- Commit the metadata files to the catalog. If the commit failed retries as the number of configurations.
8. Puffin files contains:
- deletion vectors - Bitmap structures tracking deleted rows without rewriting files.
- Statistics and Indices - Additional metadata for performance enhancement.
9. What is orphan files? Files that are no longer associated with any application or process.
10. boring catalog - A lightweight file-based Iceberg catalog implementation using a single (boring) JSON file. The goal is to let people experiment easily with Iceberg’s commit mechanism and remove some of the usual friction around catalog setup.
11. Compaction strategies:
- binPacking - write small files to large files regardless of record order.
- sort - Run compaction, sort the records by one or more fields.
- zOrder - Equal weighting of fields to be sorted by in compaction.

### 🔄 Alternatives
Assignment: You are required to research and write a comparative analysis between Iceberg and an industry alternative.
- Deliverable: A written summary (minimum 1 or 2 sentences).
- Add real life usecase 
- Focus: Compare performance, architecture, and specific "pain points" this tool solves compared to legacy systems or competitors.
- Goal: You must be able to justify why the department uses this tool for our specific environment.

### 🎯 User Story & Scenario
Assignment: Based on your research and understanding of the department's pipeline, define a concrete Use Case for this technology.
- Deliverable: A written summary example/story (two paragraphs approx.).
- Requirement: Describe a real-world scenario (e.g., a specific client requirement) where this technology is the optimal solution.
- Data Flow: Map out the data flow and explain how this tool integrates with other components in the Data Pipeline.
