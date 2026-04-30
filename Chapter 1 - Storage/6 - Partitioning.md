# Data Partitioning :

## Overview
This session isolates the concept of data partitioning. Rather than bundle it with Hive or other systems, we’ll treat partitioning as a fundamental data modeling and storage optimization technique. Expect five deep questions that cover motivation, strategies, pruning, bucketing, and real-world considerations.

**The focus is on understanding why and how data is partitioned across storage systems.**

## Goals
- Learn what partitioning means in the context of databases and data lakes.
- Explore different partitioning strategies and their trade-offs.
- Examine how partitioning interacts with query optimization and maintenance.

:warning: **Note:**
- This day is strictly theoretical; no specific software or engines are required.
- Discuss unclear points with your mentor.

### ⏳ Timeline
Estimated Duration: 0.5 Days
- Day 1: Learn what partitioning is and the core concepts; spend half a day.
    - Have a Q&A session the same day

## Core Questions

1. **Motivation & Definitions:** What problems does partitioning solve? Distinguish between horizontal and vertical partitioning, and between logical and physical partitions.
2. **Strategies:** Describe common partitioning techniques (range, list, hash, composite) and when each is appropriate. Include considerations for time-series data.
3. **Pruning & Optimization:** Explain how partition pruning works and why it’s critical for performance. How do query planners determine which partitions to scan?
4. **Maintenance & Evolution:** What challenges arise when partitions grow or have inconsistent metadata? Discuss operations like adding, dropping, or merging partitions.
5. **Bucketing & Data Layout:** What is bucketing, and how does it differ from partitioning? When is bucketing useful (e.g., joins, load balancing, reducing shuffle)? How can bucketing complement partitioning in large datasets?

## Core Questions - Answers

1. **Motivation & Definitions:** What problems does partitioning solve?\
Performance, scalability, resource utilization.
Distinguish between horizontal and vertical partitioning, and between logical and physical partitions.
- vertical partitioning - seperated data based on columns. Each partition contains a subset pf columns for all rows.
- horizontal partitioning - divides data by rows. When the horizontal partitions are scale out it called sharding.
- Logical partition - organizing data by design. Based on partition key.
- Physical partition - handle the underlying storage structure such as compactions, sharding ensure efficient storage distribution for optimal performance. Happens by automatic mechanismns of the system.

2. **Strategies:** Describe common partitioning techniques (range, list, hash, composite) and when each is appropriate. Include considerations for time-series data.
- Key-based partitioning - divide data based on key. Stores data with the same key together for efficient lookups.
- Range patitioning - Range partitioning divides data into partitions based on specific value ranges such as dates, IDs, or numbers.
- Hash partitioning - distributes data across partitions using a hash function applied to a specific key. Provides even distribution.
- Composite partitioning - Unlike basic partitioning, which uses a single criterion, composite partitioning applies multiple criteria, allowing for a more granular and efficient organization of data. Based on two methods for example range-hash, range-list, hash-hash, etc.

3. **Pruning & Optimization:** Explain how partition pruning works and why it’s critical for performance. How do query planners determine which partitions to scan?\
Partition pruning prevents from the database from scanning irrelevant partitions while executing queries. It determines which partitions to scan based on the partitioned column that is used in query filters. If the partition column is not used in the query's filter condition, then the system won't know which of the partitions to ignore.\
Static partition pruning - happens at the compile time according to the query.
Dynamic partition pruning - happens at runtime. It only knows what to prune (the search condition) in the runtime.

4. **Maintenance & Evolution:** What challenges arise when partitions grow or have inconsistent metadata? 
When the amount of partitions is growing, if the data in each partition is small it will cause a lot of metadata overhead while running operations over the data (small files problem and enormous amount of partitions) leading the needs of merging files (an action that needs to be seamless to the client which won't break their APIs, I read about someone who even needed to merge partitions, which is rebuilt the whole system again) and dropping very old partitions to maximize efficiency.

5. **Bucketing & Data Layout:** What is bucketing, and how does it differ from partitioning? When is bucketing useful (e.g., joins, load balancing, reducing shuffle)?
Bucketing distributed data into fixed-size buckets (files) based on the hash of a specific column.
- Bucketing are files and partitioning are directories.
- bucketing used for columns with high cardinality (customer id) and partitioning for low cardinality (country).
- bucketing Are a fixed size which decides while creating the table (CLUSTERED BY) while partitioning are directories according to columns values (PARTITION BY).\
How can bucketing complement partitioning in large datasets? On large datasets we want to reduce the number of scans. We use partition to divide the data according to the most used queries used, so the system could prune the data quickly as posible by the partition key on a low cordinality columns (low amount of elements). We can do a subpartition but on data of customer_id for example it will create a lot od subdirectories causing the small files problems. Therefore, to even be more efficient and scanning even less data we use bucketing. The buckets number is decided when the table is created so we have the ability to prevent the small files problems and also reduce the amount of scans inside a partition.

## Wrapping Up :trophy:
Review your answers with your mentor and identify scenarios where partitioning could dramatically improve or hurt a workload.

## Action Items
- Identify storage systems you want to try partitioning in (e.g., Hive, Iceberg, PostgreSQL).
- Prepare questions for the next mentor Q&A.

## Q&A Answers - Partitions & Table Formats

1. Does the metadata layet must be store in the same place of the data?
2. Horizontal and Partitioning combination is called hybrid partitioning.
3. Data skew - uneven distribution of data across different partitions.
4. Partitioning which are unbalanced are data skew (very funny)
5. What if I want to add information after bucketing the table? 
