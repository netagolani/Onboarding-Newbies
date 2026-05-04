# Hive on MapReduce and Tez 🐝

## Overview
Today’s session introduces how Hive queries are executed using distributed compute engines. While Hive defines the table format, schema, and metadata through the Hive Metastore (HMS), the actual query execution is performed by an underlying processing engine.

Historically, Hive executed queries using MapReduce. Later, Apache Tez was introduced to significantly improve performance and efficiency.

**The emphasis is on understanding how Hive queries are translated into distributed execution jobs and how MapReduce and Tez differ as execution engines.**

## Goals
- Understand how Hive queries are executed in a distributed environment.
- Learn the basic execution model of MapReduce.
- Understand how Apache Tez improves Hive query execution.
- Build intuition about how SQL queries translate into distributed jobs.

:warning: **Note:**
- This is a self-study day; independence and time management are crucial.
- Focus on understanding the concepts rather than memorizing implementation details.
- If you cannot explain how a Hive query becomes a distributed job, revisit the material.
- Ask your mentor if something is unclear.

### ⏳ Timeline
Estimated Duration: 1 Day

- Day 1: Learn the basics of Hive query execution and how MapReduce and Tez are used as execution engines.

---

### Guide Questions❓

Answer these five questions to understand how Hive queries are executed using MapReduce and Tez.

1. **Hive as a Query Platform:**  
   Hive provides tables, schemas, and SQL querying on top of distributed storage systems such as HDFS. Explain Hive’s role as a platform layer that sits above storage and relies on external compute engines to execute queries.

2. **Hive Query Stages and Task Execution:**  
When Hive translates a SQL query into a distributed job, how is the work divided into stages and tasks?  
Explain how Hive breaks a query into execution stages, how tasks are distributed across the cluster, and how intermediate results are passed between stages.

3. **Hive Query Execution Pipeline:**  
   What happens when a user runs a query in Hive? Describe the main stages of execution: SQL parsing, logical planning, physical planning, and submitting jobs to an execution engine such as MapReduce or Tez.

4. **MapReduce Fundamentals:**  
   What is the MapReduce programming model? Explain the roles of the `map phase`, `shuffle and sort`, and `reduce phase`. Why was MapReduce originally used as Hive’s execution engine?

5. **Introduction to Apache Tez:**  
   What is Apache Tez, and how does it improve Hive query execution? Explain how Tez replaces chains of MapReduce jobs with a Directed Acyclic Graph (DAG) of tasks, reducing unnecessary disk I/O and improving query performance.

---
### Guide Questions - Answers

1. **Hive as a Query Platform:**
   Hive provides tables, schemas, and SQL querying on top of distributed storage systems such as HDFS. Explain Hive’s role as a platform layer that sits above storage and relies on external compute engines to execute queries.\
   Hive provides an abstraction layer on top of HDFS that represents the data as tables with rows, columns, and data types to query and analyze using an SQL interface called HiveQL. It abstracts the complexity of writing MapReduce code by allowing users to interact with structured data through familiar query syntax. Converts HiveQL queries into MapReduce, Tez, or Spark jobs.

2. **Hive Query Stages and Task Execution:**  
   When Hive translates a SQL query into a distributed job, how is the work divided into stages and tasks?  
   Explain how Hive breaks a query into execution stages, how tasks are distributed across the cluster, and how intermediate results are passed between stages.\
   The work is divided into stages and tasks by the compiler according to the metadata it gets from the HMS. Every stage in the execution plan is either a map/reduce job on HDFS.
   Hive breaks a query into execution stages by the compiler which generates an execution plan with optimizations. Tasks are distributed across the cluster by Yarn. Intermediate results are passed between stages via shuffling while there written in temporary files in the HDFS.

3. **Hive Query Execution Pipeline:**  
   What happens when a user runs a query in Hive? Describe the main stages of execution: SQL parsing, logical planning, physical planning, and submitting jobs to an execution engine such as MapReduce or Tez.\
   - execute query - the user submits a query from the UI to the driver (JDBC/ODBC).
   - get plan - the driver creates a session with the compiler to generate an execution plan.
   - get metadata - the compiler sends metadata request to HMS.
   - send metadata - the HMS send the metadata to the compiler. The compiler performaing type-cheking and semantic analysis. Generates the execution plan. The plan is a DAG of stages
   - send plan - The compiler sends the execution plan to the driver.
   - execution plan - the driver sends the execution plan to the execution engine.
   - submit job - the execution engine then sends the stages for execution on HDFS. For each task (mapper or reducer) the deserializer read the rows from HDFS files and then written to the HDFS temporary file through the serializer. The final remporary file is then moved to the table's location.
   - send result - the execution engine reads the temporary files directly from HDFS as part of a fetch call from the driver. The driver sends results to the Hive interface.

   The driver includes internal stages: parser, planner, optimizer, executor.
   logical planning (what data) by the optimizer can be:
   - predicate pushdown - a technique that pushes down the filter conditions to the storage layer, so that the storage layer only returns the rows that satisfy the filter conditions. Improve performance.
   - Column/projection pruning - a technique that removes the unnecessary columns from the query plan, so that only the columns that are required for the query are processed.
   - join reordering - optimized join order for performance.

   physical planning (how to execute data) by the optimizer can be:
   - partition/bucketing pruning - skipping irelevent partitions or buckets.
   - perform join on the mapper.
   

4. **MapReduce Fundamentals:**  
   What is the MapReduce programming model? Explain the roles of the `map phase`, `shuffle and sort`, and `reduce phase`. Why was MapReduce originally used as Hive’s execution engine?\
   MapReduce programming model:
   - `map phase` - map phase accepts key value pairs (k,v) as input.
   - `shuffle and sort` - gets the output of the mapping phase. Different values are grouped together based on same keys. The output will be (k,v[]).
   - `reduce phase`- gets the output of the shffling and sorting phase. The reducer function is executed to all the values on each key and computes the final output.

5. **Introduction to Apache Tez:**  
   What is Apache Tez, and how does it improve Hive query execution? Explain how Tez replaces chains of MapReduce jobs with a Directed Acyclic Graph (DAG) of tasks, reducing unnecessary disk I/O and improving query performance.\
   Apache Tez is a framework for building very high performances data processingby creating complex DAGs. Tez sees the big picture and created the most efficient DAG In contrast to map reduce which creates mappers and reducers causing alot of overhead with many stages. Tez sits on top of Yarn.
   - Less I/O to disk - MapReduce access the disk multipule times during processing, resulting at least 5-6 disk access for a single mapreduce job.
   - vectorization - Tez gets data from disk, performs all the steps, stores the intermediate results in the memory performs vectorization (processes batch rows instead of one row at a time).
   - container reuse - tez enables to reuse containers while in mapreduce there is a new container for each task in the job.
   - sorting - mapreduce sort the ouput from each map while Tez doesn't due to is complex Dag which is planned before execution.

### Q&A - Answers

1. Hive aux - auxiliary jars are jars which enables additional capabilities that configured in the hive class path. Such as custom SerDe, dimention lookup and iceberg hive routine.

2. Where are the intermidiate results stored in MR and Tez?\
MR - intermidiate results are stored in the HDFS.\
Tez - intermidiate results are stored in the memory. Unless it gets to the threshholdsand of tez.runtime.unordered.output.buffer.size-mb and then spills to the disk.
3. What is stored in the memory and what is stored of disk in Tez?\
Intermidiate results are stored in memory (unless it gets to the maximum  buffer) while final results are wrriten to HDFS.
4. Why in MR there is a network overhead?
MR has a network overhead because of the shuffling method which transfers data from servers and because all of the intermidiate results are written to HDFS must be replicated and approved by the name node.
5. Why does Tex can reuse containers in contrast MR?\
Tez can reuse containers because after finishing a task, Tez doesnt release the container's resources back to Yarn. It reassing a new task to the same container. (if the local data is relevent to the new task its even more efficient).
6. Jobs, Tasks & Stages in MR.

7. Who is responsible of how many workers (containers) in MR and Tez?

8. Problems of big or small files in hive

9. Who is resposible of doing retries while quering hive?

10. Fault tolerance in Hive.

11. Usecases of Hive.

---

### Things I Want to Remember:
-  Yarn was created to decouple resource managment from job execution.
- Yarn is the resource management layer of Hadoop. 
- Yarn allows multipule engines to run on single cluster.
- HiveServer2 is a service that enables clients to execute queries against hive. Supports multi-client concurrency and authentication. Better support for clients like JDBC and ODBC.
- HS2 is includes the components of thrift server, compiler, driver.

### 🔄 Alternatives
Assignment: Briefly research another distributed processing framework used for large-scale data processing.

- Deliverable: A written summary (1–2 sentences).
- Add a simple real-life use case.
- Focus: What problem does this framework solve compared to MapReduce or Tez?

---

### 🎯 User Story & Scenario
Assignment: Based on your research and understanding of the department's pipeline, define a concrete Use Case for this technology.
- Deliverable: A written summary example/story (two paragraphs approx.).
- Requirement: Describe a real-world scenario (e.g., a specific client requirement) where this technology is the optimal solution.
- Data Flow: Map out the data flow and explain how this tool integrates with other components in the Data Pipelin

## Wrapping Up :trophy:
Review your answers with your mentor and make sure you can clearly explain how a simple Hive query becomes a distributed computation job. This knowledge will help you understand later query engines and compute systems used in the data platform.

---

## Action Items
- Identify parts of Hive query execution you want to explore further.
- Look at example Hive query plans to see how jobs are structured.
- Prepare questions for the next mentor Q&A session.

---

### 📚 Resources
Use the resources listed below and practice searching the internet for questions not answered by the provided documentation.
- [Hive Documentation](https://hive.apache.org/docs/latest/)
- [Hadoop: The Definitive Guide (O'Reilly)](https://piazza-resources.s3.amazonaws.com/ist3pwd6k8p5t/iu5gqbsh8re6mj/OReilly.Hadoop.The.Definitive.Guide.4th.Edition.2015.pdf)
- [Apache Tez Documentation](https://tez.apache.org/)
- [Apache MapReduce Docs](https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)
