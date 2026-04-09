# Hadoop Distributed File System (HDFS) :elephant:

## Overview
This session focuses on the core concepts of HDFS, the distributed storage layer of the Hadoop ecosystem. Understanding its architecture will help you appreciate how big data clusters store and manage massive datasets across many machines.

**Study the key components, design decisions, and how they work together to provide fault-tolerant, scalable storage.**

## Goals
- Learn the architecture and roles of HDFS components (NameNode, DataNode, etc.).
- Understand how HDFS handles storage, replication, and availability.
- Practice planning a self-study day and managing your time.

:warning: **Note:**
- This is a self-study day; independence and time management matter.
- Focus on grasping the full picture of each concept; if you can’t explain it, you haven’t learned it.
- When in doubt, consult your mentor about what to study.

### ⏳ Timeline
Estimated Duration: 3 Days
- Day 1-3: Learn the concepts of HDFS; spent time on what is it? on fault tolernce, on failover process and on how reads and writes are being done?
    - Have a Q&A session at the third day and in between sessions each day

## Core Concepts

Consider the following five questions to cover the major HDFS topics:

1. **Architecture & Roles:**  Describe HDFS’s overall architecture, including NameNode(s), DataNodes, blocks, and how the namespace and metadata are managed. Don’t forget the role of ZooKeeper in coordinating HA and keeping track of leases.
2. **Storage & Fault Tolerance:**  Explain how HDFS divides files into blocks, uses replication (default factor three), and how it detects and recovers from node failures.
3. **Topology Awareness & Performance:**  What is rack awareness and why does HDFS replicate across racks? Discuss how block placement, snapshots, and checksums contribute to performance and data integrity.
4. **High Availability :**  Outline HDFS High Availability (Active/Standby NameNode, JournalNodes). How do these features improve scalability and uptime?
5. **Protocol & Operations:**  Describe how clients read and write data to HDFS via RPC, how they locate NameNodes and DataNodes, how DataNodes send block reports, and why these mechanisms matter for everyday operations. Cover the runtime behaviour of leases and pipeline formation.

## Core Concepts - Answers :elephant::elephant::elephant:

1. **Architecture & Roles:**
HDFS stands for Hadoop Distributed Filesystem.\
HDFS architecture works in a master-slave pattern.
- Blocks - A disk  has a block size, which is the minimum amount of data that it can read or write. HDFS, too, has a concept of a block, but it much larger - 128MB by default (normally a disk block is 512 bytes). A file that is smaller than a single block does not occupy a full block's, its uses 1MB of disk space. HDFS blocks are large compared to disk blocks, because it minimized the cost of seeks.
- Name Node (master) - Master server that manages file system namespace and regulate access to files by clients. The name node is responsible for all client operations in the cluster. It does not store block locations persistantly, because this information is reconstructed from datanodes when the system starts.\
- Data Nodes (slaves) - serves read or write requests, it also creates, deletes, and replicates blocks based on the instructions from the name node. They report back to the namenode periodically with lists of blocks that they are storing.
- namespace - The hierarchy of where data is stored. Allows user data to be stores in files. The namenode manages the file system namespace allowing the clients to work with files and directories with operations of create, remove, move, rename, etc. NameNode maintains file system namespace. Any changes to file system namespace or its properties is recorded by the NameNode and also it has replication factor - number of copies of a file.

4. **High Availability :**
Without the namenode, the filesystem cannot be used, all the files on the filesystem would be lost since there would be no way knowing how to reconstruct the files from the blocks on the datanodes. To solve this there are two mechanisms: Backup up files that make up the persistent state of the filesystem metadata. Can be written to local disk as well as remote NFS mount.
Another way is a secondery namenode which also called the standby node. The standby node reads the changes made to edit logs and applies it to its own namespace in a consistent manner. In event of a failover the standby node will ensure that is has read all the edits before promoting itself to the active state. This is a manual process which has to be performed by admin unless you have a zookeeper which manages failovers automatically with failover controllers. The zookeeper periodically managing health checks to the namenode and when the master will marked as unhealthy a new name node will be elected.

### 🔄 Alternatives
Assignment: You are required to research and write a comparative analysis between HDFS and an industry alternative.
- Deliverable: A written summary (minimum 1 or 2 sentences).
- Focus: Compare performance, architecture, and specific "pain points" this tool solves compared to legacy systems or competitors.
- Goal: You must be able to justify why the department uses this tool for our specific environment.

### 🎯 User Story & Scenario
Assignment: Based on your research and understanding of the department's pipeline, define a concrete Use Case for this technology.
- Deliverable: A written summary example/story (two paragraphs approx.).
- Requirement: Describe a real-world scenario (e.g., a specific client requirement) where this technology is the optimal solution.
- Data Flow: Map out the data flow and explain how this tool integrates with other components in the Data Pipeline.


## Wrapping Up :trophy:
Review your answers with your mentor and discuss any unclear points. Relate these concepts back to real-world usage scenarios you might encounter.

## Action Items
- Note topics you want to investigate further.
- Prepare questions for the mentor Q&A session.
- Continue the Day 01 challenge by linking these HDFS concepts to other chapters.

## Recommended Resources
- [Official HDFS User Guide](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html)
- [Hadoop: The Definitive Guide (O'Reilly)](https://piazza-resources.s3.amazonaws.com/ist3pwd6k8p5t/iu5gqbsh8re6mj/OReilly.Hadoop.The.Definitive.Guide.4th.Edition.2015.pdf)
