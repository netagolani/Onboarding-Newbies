# Introduction to Trino 🐰

## Overview
Today’s session introduces Trino, a distributed SQL query engine designed for high-performance analytics across multiple data sources. Unlike traditional databases, Trino does not store data itself. Instead, it queries data where it already resides, enabling fast interactive analysis across large and diverse datasets.

**The emphasis is on Trino’s architecture, distributed execution model, and how it integrates with modern data platforms.**

## Goals
- Understand what Trino is and where it fits in a modern data architecture.
- Learn how Trino executes distributed queries across workers.
- Explore how Trino connects to multiple data sources and federates queries.
- Improve your ability to research distributed data systems independently.

:warning: **Note:**
- This is a self-study day; independence and time management are crucial.
- If you can’t explain a concept clearly, you probably need to revisit it.
- Ask your mentor if you’re unsure what to research.

### ⏳ Timeline
Estimated Duration: 3 Days

- Day 1: Learn the fundamentals of distributed SQL engines and Trino’s role in the data ecosystem.
- Day 2: Dive deeper into Trino architecture, connectors, and query execution.
- Day 3: Review performance optimization and operational concepts, followed by a Q&A session.

---

## Core Concepts

### Part 1: Distributed SQL Engines (General Concepts)

Answer these questions to understand the broader category of distributed query engines before focusing on Trino specifically.

1. **Role in the Data Architecture:**  
   What is a distributed SQL query engine, and where does it sit in a modern data architecture? How does it differ from storage systems such as data lakes or databases?

2. **Motivation & Use Cases:**  
   Why do organizations use distributed SQL engines? In what scenarios are they preferred (for example: interactive analytics on large datasets, federated queries across multiple systems, or querying data lakes)?

3. **Distributed Query Processing:**  
   How do distributed query engines typically execute queries across multiple nodes? Explain concepts such as parallel processing, data shuffling, and shared-nothing architectures.

---

### Part 2: Trino (Implementation & Operations)

Answer these questions to cover Trino’s major architectural and operational concepts.

1. **Purpose & Position in the Data Stack:**  
   What is Trino, and where does it sit in the data architecture? Explain its role as a distributed MPP SQL engine, how it differs from storage systems, and when it should be used instead of other query engines.\
2. **Architecture & Query Execution:**  
   How is Trino architected, and how does it execute queries in a distributed environment? Discuss the roles of the Coordinator and Workers, stages and tasks, data exchanges, and the execution model.

3. **Connectors & Catalogs:**  
   How does Trino integrate with external systems through connectors and catalogs? Explain what connectors are responsible for, how catalogs are configured, and how Trino can federate queries across multiple data sources.

4. **Governance & Workload Management:**  
   How does Trino handle governance and workload management? Discuss resource groups, concurrency control, memory limits, access control (RBAC), and multi-tenant isolation.

5. **Query Optimization & Performance:**  
   How does Trino optimize query performance? Explain cost-based optimization, statistics usage, join strategies, predicate pushdown, partition pruning, spilling, and caching behavior.

6. **High Availability & Multi-Cluster Routing:**  
   How does Trino Gateway enable high availability and workload isolation across multiple Trino clusters? Explain how it provides a single entry point, performs rule-based query routing and load balancing, monitors cluster health, and enables failover and multi-cluster isolation beyond what a single Trino coordinator can support.

---

## Q&A #1 Answers
1. SPI - service provider interface. Defines the functionality a connector has to implement. By implement SPI, Trino can use standart operations internally to connect to any data source and perform operations on it.
2. UDF - trino supports user-defined functions UDFs, which allow you to write your own function implementations and deploy them in Trino to execute within SQL queries. Return a single output value. Inline UDF - only valid within the context of the query. Catalog UDF - to be used in any future query.
3. The connection between connector and catalog - every catalog uses a specific connector. A catalog is a collection of configuration properties used to access a specific data source with a required connector by the property connector.name.
4. distributed plan vs logical plan -
   - logical plan - after getting a statement in a text format, the coordinator parses and analyzes it. Then it creates a query plan which represents the needed steps to process the data and return the results per SQL statement. It uses the Metadata SPI to get information about tables, columns and types to validate the query and the statistics SPI to perform cost-based query optimizations during planning.
   - distributed plan - extansion of the simple query plan consisting of one or more stages. The data location SPI facilitated in the creation of the distributed query plan.
5. Views Types -
   - simple view - ach time we access a regular view, it runs the relative query in the background.
   - materialized view - stores data persistently on the main storage. It serves as a snapshot view of the data. Materialized views can improve query performance resulting in faster access.
Implementation of views in Trino via hive connector - If using Hive views from Trino is required, you must compare results in Hive and Trino for each view definition to ensure identical results. There are three modes to handle hive views.
   - Disabled - the default behavior is to ignore Hive views.
   - Legacy - translates HiveQL query that defines a veiw as if it is written in SQL, without any translations. It can leads to problems and errors.
   - Experimental - Analyze, process and rewrite Hive views. Contained expressions and statements.
*****still need to understand the implementation of trino in context to the views types.
6. Trino optimizations -
   - predicate pushdown - optimizes row-based filtering. Filter unnecessary rows from a condition in a `WHERE` clause. The processing is pushed down to the data source by the connectorand then processed by the data source. This reduces the network traffic between Trino and the data source and improved overall query performance.
   - Projection pushdown - column-based filtering by the select clause.
   - join enumaration - Trino uses Table statistics provided by connectors to estimate the costs for different join orders and automatically picks the join order with the lowest computed costs.
7. Fault tolerance execution - feature that is turned of by defualt. Is a mechanism in Trino that enables a cluster to handle query failures by retrying queries or their components tasks in the event of worker fails or lack of resources. Intermediate exchange data is spooled and can reuse by another worker.
When configured, the Trino cluster buffers data used by the workers during query processing. In case of failure, the coordinator reschedules processing of the failed piece of work on another worker. This allows query processing to continue using buffered data. The coordinator node uses a configured exchange manager service that buffers data during query processing in an external location, such as an S3 object storage bucket. Worker nodes send data to the buffer as they execute their query tasks. You can configure it in a scope of cluster with different retries policies according to the cluster type.
8. Trino Gateway - The load balancer, proxy server and configurable routing gateway for multiple trino clusters. sers don’t need to worry about what catalog and data source is available in what Trino cluster. Trino Gateway exposes one URL for them all. Administrators can ensure routing is correct and use the REST API to configure the necessary rules.
9. What is Impersonation? User Impersonation allows Administrators to access and operate as if they were logged in as that User.
10. Explain & Analyze commands -
    - explain - Show the logical or distributed execution plan of a statement, or validate the statement. The distributed plan is shown by default. validate returns true or false checking if the statement is valid. TYPE { LOGICAL | DISTRIBUTED | VALIDATE }
    - analyze - Collects table and column statistics for a given table.
11. Caching - File system caching keeps copies of the retrieved files on a local cache storage, separate for each node. Over time the same files from object storage are cached on any nodes that require the data file for processing a specific task. Each cache on each node is managed separately, following the TTL and size configuration, and cached files are evicted from the cache.
12. What you have in Trino UI - you need to authenticate to the UI firstly.
    The main page has a list of queries along with information like unique query ID, query text, query state (Queued, Running, Blocked, Failed). I can click the query for more details with summery section, graphical representaion of various stages and list of tasks. It also has a buttom to kill the currently running query.
    
## Q&A #2 Answers

1. What is the other restart policy exept of `TASK`?
   - Query - automatically retry a query in the event of an error occuring on a worker node. Ideal for cluster of small queries.
   - Task - retry individual query tasks in the event of failure. Must configure an exchange manager (responsible for storing spooled data). Ideal for executing large batch queries.
The  retry policy configured in the cluster level.
2. How I decide the priority of a query?
According to the selector of the query.
3. Which parameters can be configured in the selectors? User, userGroup, source, queryText, queryType, clientTags, group.
4. Access Control - can handle and restrict actions. Enforce authorization from trino before the connection authorization. There are multipule types of access control and you can combine them but it can cause collisions.
   - File-based access control - json files which defines the acess control to role/user/group in levels of catalog, schema and tables. Kind of messy solution of handling access control with big jsons files.
   - Ranger access control - apache framework to manage access controls.
5. JMX - exposes large number of different metrics via Java Management Extensions. You can also use the JMX connector and query the metrics using SQL. For example heap size, thread counts, information about trino queries and tasks, etc.
6. gateway dependencies on set up: Required an SQL database like postgreSQL (usr, password, connection url, driver, disable or enable migrations. HTTP headers must be enabled `http-server.process-forwarded=true`. Set all needed in the config.yaml and the helm set up (adding backendState) configuration, activate JMX monitoring in all trino clusters
7. cluster classification of the gateway: Trino gateway knows which catalog is navigated to which cluster by the user or by tags for now.
8. How resource groups related to the gateway: The gateway only shows information about resource groups via the UI.
9. How to add new routing rules to the gateway? You need to configure in the routing rule yaml (config map) a routing group and map it to a specific cluster.
10. What is external routing service? Instead of configuring the routing in the routing rule yaml it can be more complex routing with an external routing service which classifying the the RoutingGroup with additinal selections before getting to the gateway.
11. Two types of cache in trino? file system cache and metadata cache (iceberg supports caching metadara in the coordinator meory, enabled by default)
12. What information is kept in the cache? file caching for example to file from table in hdfs.
13. Explain Analyze command - Execute the statement and show the distributed execution plan of the statement along with the cost of each operation.
14. What is more important strong worker or strong coordinator? strong coordinator, of course. The coordinator is the single point of failure of the cluster, which resposible of coordinating all the queries while a worker is just a work and there is fault tolerance execution plan for that.
15. Tools for understanding performance and progress of a query? Superset, grafana dashboards and elastic dashboards. JMX monitoring.
16. Pros and Cons of JVM in trino: It has memory management by the GC, common API to handle I/O, JMX metrics,compiled into bytecode. But the performance is slowerthen programs wrriten in native languages for specific platform with optimizations.
   

### 🔄 Alternatives
Assignment: You are required to research and write a comparative analysis between Trino and an industry alternative.

- Deliverable: A written summary (minimum 1–2 paragraphs).
- Add a real-life use case.
- Focus: Compare performance, architecture, and specific "pain points" this tool solves compared to legacy systems or competing technologies.
- Goal: You must be able to justify why the department uses this tool for our specific environment.

---
### 🎯 User Story & Scenario
Assignment: Based on your research and understanding of the department's pipeline, define a concrete Use Case for this technology.
- Deliverable: A written summary example/story (two paragraphs approx.).
- Requirement: Describe a real-world scenario (e.g., a specific client requirement) where this technology is the optimal solution.
- Data Flow: Map out the data flow and explain how this tool integrates with other components in the Data Pipeline.
---


## Wrapping Up :trophy:
Go over your answers with your mentor and clarify any uncertainties. Relate Trino concepts back to the broader data platform and how distributed query engines interact with storage systems and processing frameworks.

---

## Action Items
- Identify Trino topics you want to explore further.
- Search examples of industray usage of Trino.
- **Bonus** - what is the rabbit's name?
- Prepare questions for the next mentor Q&A session.

---

## Recommended Resources
- [Official Trino Documentation](https://trino.io/docs/current/) – the primary reference guide.
- [Trino: The Definitive Guide (O'Reilly)](https://dokumen.pub/trino-the-definitive-guide-sql-at-any-scale-on-any-storage-in-any-environment-2nbsped-109813723x-9781098137236.html)
- [Official Trino Gateway Documentation](https://trinodb.github.io/trino-gateway/)
