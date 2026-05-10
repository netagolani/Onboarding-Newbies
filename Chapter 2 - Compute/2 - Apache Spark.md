# Spark Fundamentals:
## Overview
This section will go over the fundamentals of _apache spark_.

**We will focus on general concepts such as the spark architecture, optimizations, caching and data geometry.**

## Goals
- Develop a foundational understanding of how scheduling is done.
- Learn the common terminology used by most schedulers.
- Practice planning a self-study day and estimating time for learning.

:warning: **Note:**
- This is a self-study day. Independence and time management are essential.
- Many newcomers struggle with self-study; take a moment to plan your day and stick to it.
- Understand the **big picture** of each concept. If you can't explain it, you probably haven't learned it.
- Be prepared to describe how concepts relate to one another and to real-world scenarios.
- Review the [Exercise](#exercise) before diving in so you know what to focus on.
- When in doubt about what you need to learn, ask your mentor.

### Core Concepts

Think through the following questions; by answering them you’ll touch every major topic listed above:

1. **Spark Architecture & Execution:** what are the main components of spark? what is the role of each component? what are their roles? what is the difference between a transformation and an action? how does spark achieve fault tolerance? what is lazy execution in spark? go over [this](assets/where_do_i_run.py) and for each line, comment where it runs.
- Main components of spark:
    - Driver Program - The execution coordinator process. Runs the main function. Creates the SparkContext which connects to the cluster manager.
    - SparkContext - EntryPoint for spark functionality. Represents the spark connection. Creates RDDs. Coordinates the execution of tasks.
    - Cluster Manager - An external service for managing and allocating resources on the cluster (YARN, K8s, standalone manager) 
    - Worker Node - Node that runs spark executors.
    - Executor - A process launched on a worker node. Runs tasks. Keeps data in memory or disk for caching and intermidiate storage. Each application has its own executors Communicates with the cluster manager and the driver program.
    - Task - A unit of work that will be sent to one executor.
    - Job - A parallel computation consisting of multiple tasks.
    - Stage - Each job gets divided into smaller sets of tasks called stages that depend on each other.
- The difference between a transformation and an action:
    - transformation - operation on RDDs, DFs or datasets that creates new distributed dataset from an existing one. Creates a logical execution plan. Evaluated lazily, meaning they are not executed until an action is called. The building blocks for constructing the logical flow of data. There are two types of transformation:
          - Narrow Transformation - one to one, narrow dependencies (map, filter).
          - Wide Transformation - one to many, wide dependencies, required shuffling and represent a stage boundary in the DAG. (groupByKey, reduceByKey, join)
  - action - operations that trigger the physical execution plan. Returns value to either the driver program or to external storage system. Operations which initiate computations and produce a result or a side effect. (colect, count, first, saveAsTextFile).
- Fault tolerance in Apache Spark -
    - Lineage Information - Is a DAG that represents the sequence of transformations applied to an RDD. Lineage information serves as a recipe or set of instructions to recompute lost or corrupted data partitions in case of node failures. It provides a clear history of how an RDD was derived from its source data, allowing Spark to re-execute the transformations that led to its creation.
    - Data Replication - for RDDs. By default, data is replicated at least once across different nodes, reducing the risk of data lose due to node failures.
    - Checkpointing - save the state of an RDD to a storage system like HDFS. Fast recovery instead of recomputong the entire lineage.
    - Task Re-execution - In case of node failure, Spark can reschedule the failed tasks on other available nodes. The data needed for those tasks is recomputed using the lineage information.
    - Pesistent Storage - Spark stores intermidiate results as a persisted data in case of node failures instead of recomputing it.
    - Driver Recovery - If the driver node fails, the driver’s state can be recovered by restarting the application and re-executing the driver code.
    - Dynamic Resource Aloocation - This means that if a node fails, its resources can be reclaimed and reallocated to other tasks, ensuring efficient resource utilization.
- Lazy execution in Spark - Lazy evaluation means that Spark doesnt executr transformations as soon as the are defined. Instead it builds a logical execution plan and waits until an action is called. It optimized execution, minimizes data movement and achieves fault tolerance.
  

2. **Spark Planning & Optimization:** Logical vs Physical Planning: Walk through the transition from Logical Plan to Physical Plan; What is the fundamental difference between Rule-Based (RBO) and Cost-Based Optimization (CBO), what are the common kinds of optimizations used? What is the AQE? Why is running ANALYZE TABLE recommended for performant CBO? and what is whole-stage code generation?
- The catalist optimizer is a query optimization framework to optimize the execution of data queries. The transition from logical plan to phisical plan:
    1. unresolved logical plan - gets an SQL query or DF. Output the first version of a logical plan where relation name and columns are not specifically resolved. Validate syntax and code.
    2. analyzed logical plan - resolve by the catalog (metastore) the unresolved datastructures, schema and types.
    3. optimized logical plan - reorder the logical plan by rule-based optimizations (RBO), predefined rules to simplify and optimize query plans focusing on query structure. (predicate pushdown, constant folding, projection pruning).
    4. physical plans - from logical plan, the plan is described how it will physicaly executed on the cluster in different kinds of execution strategies.
    5. "Cost Model" - comparing all the physical plans.
    6. selected physical plan - decides the final plan of which partitions should be joined first, type of join, broken down into stages,  divides jobs into tasks and assigns them to executors. Cost-based Optimization (CBO) analyzes data statistics to make informed decisions. Helps join reordering, broadcast selection and aggregation optimizations based on data distribution.
    7. Code Generation - is a physical query optimization in Spark SQL that fuses multiple physical operators (as a subtree of plans that support code generation) together into a single Java function. Improves the execution performance of a query by collapsing a query tree into a single optimized function that eliminates virtual function calls and leverages CPU registers for intermediate data.
- AQE - Adaptive Query Execution. New feature in Spark 3.0 which enables plan changes at runtime. It collects statistics during plan execution and if Spark detects better plan during execution, it changes them at runtime. If we want to see these changes it won't be in the explain() function, it will be in the Spark UI.  
- RBO are based on predefined rules for logical plan while CBO use some statistical properties of data for physical plan.
- Running ANALAZE TABLE ensures that the statistics are exposed, accurate and up to date in the metastore for CBO.


3. **Spark Shuffle & Joins:** Compare the different kind of joins, and when will spark use each? how can we tell spark to prefer one over the other? what is join reordering? and why is "broadcasting" considered a high-risk, high-reward optimization? What is a _Narrow_ transformation, and _Wide_ transformation? Why do some operations require shuffle? what exactly is written in shuffle?\
- Join Types:
  - Inner Join - returns only the rows where there is a match in both tables.
  - Left Outer Join - returns all the rows from the left table, along with matching rows from the right table. If there is no match, NULL values are returned for the columns from the right table.
  - Right Outer join - the opposite.
  - Full outer join - returns all the rows when there is a match in either the left or right table. It combines the results of both left and right outer joins.
  - Broadcast Hash join - In broadcast hash join, copy of one of the join relations are being sent to all the worker nodes and it saves shuffling cost. only supported for '=' join. supported for all join types except full outer joins. When the broadcast size is small, it is usually faster than other join strategies. Copy of relation is broadcasted over the network. Therefore, being a network-intensive operation could cause out of memory errors or performance issues when broadcast size is big. You can’t make changes to the broadcasted relation, after broadcast. Even if you do, they won’t be available to the worker nodes(because the copy is already shipped).
  - Shuffle Hash Join - moving data with the same value of join key in the same executor node followed by Hash Join. it’s an expensive join in a way that involves both shuffling and hashing.
  - Shuffle sort-merge join - shuffling of data to get the same join_key with the same worker, and then performing sort-merge join operation at the partition level in the worker nodes. The defult join strategy, and the join keys need to be sortable, supported for all join types.
  - Cartesian Join - of the two relations is calculated to evaluate join.
  - Broadcast nested loop join - Like nested look. Very slow strategy, the fall back option.
- We can tell spark to prefer one join via the other with join hints.
- Join reordering - is an optimization technique performed by the Catalyst optimizer to rearrange the sequence of multiple joins in a query for better performance. It aims to minimize intermediate data sizes, reduce shuffle operations, and lower overall execution costs by selecting an optimal join order. Happens in RBO & CBO (ANALYZE TABLE COMPUTE STATISTICS).
- "broadcating" considering a high risk because it can cause an OOM errors but a high reward because its the efficient wa[y to join if it fits.
- I explained the difference of wide and narrow transformations earlier.
- Operations required shuffle in wide transformations because the transformation cant be done on each partition it has to combine data from the partitions.
- What is wrriten in a shuffle? https://medium.com/@sairam94.a/what-i-learned-about-spark-shuffles-after-8-years-of-writing-production-jobs-33d454c92150 great article for that. In shuffle the data is wrriten to disk. For each task there is a shuffle data (contains output for all partitions) and index (byte offsets so reduce tasks can quickly find their data) files.

4. **Tungsten & Resources in Spark:** What is an RDD? Why did Spark move away from RDDs in favor of DataFrames/Datasets? Explain how Tungsten uses off-heap memory to avoid Garbage Collection pauses. Why is it a bad idea to give one executor a lot of resources (the "Fat Executor" problem)? What is the difference between Execution/Storage memory and the overhead memory? What happens when a task exceeds its allotted execution memory?

5. **Spark Skew, Partitioning & Caching:** What is data skew? how can it be solved? what is the difference between `repartition(n)` and `coalesce(n)`? What are the spark `StorageLevel`s? what is the difference between `cache` and `persist`? why are udf's (expecially in python) bad? how does spark solve the serde bottleneck with udf's?


### Real-World Context
Rather than focusing on one technology, think about how these ideas show up in distributed processing frameworks, how they are used by other procerssing frameworks and what are the core concepts of processing.

## 🔄 Alternatives

Assignment: You are required to research and write a comparative analysis between Spark and an industry alternative.

    Deliverable: A written summary (minimum 1 or 2 sentences).
    Focus: Compare performance, architecture, and specific "pain points" this tool solves compared to legacy systems or competitors.
    Goal: You must be able to justify why the department uses this tool for our specific environment.

## 🎯 User Story & Scenario

Assignment: Based on your research and understanding of the department's pipeline, define a concrete Use Case for this technology.

    Deliverable: A written summary example/story (two sentences approx.).
    Requirement: Describe a real-world scenario (e.g., a specific client requirement) where this technology is the optimal solution.

## Wrapping Up :trophy:
Discuss your answers and any areas of confusion with your mentor. Reflect on how these general concepts will help when you later write code and help clients.

## Additional Topics from Review
- A deep dive into spark internals: what are other optimizations that are implemented in spark? what is java off-heap memory? how does spark's memory allocation work?
- What are other well known processing frameworks? what are the use cases spark meets? when should I NOT use spark?

## Action Items
- Review your notes and identify topics you want to explore deeper.
- Collect a list of real-world use cases for apache spark.
- Prepare questions for the upcoming mentor Q&A session.

## Recommemded Resources
- [Apache Spark Documentation](https://spark.apache.org/documentation.html)
