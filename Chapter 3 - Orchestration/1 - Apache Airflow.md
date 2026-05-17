# Orchestration Fundamentals:
## Overview
This section will go over the fundamentals of _Apache Airflow_, consisting of the client side, and the backend.

**We will focus on general concepts of airflow, the flow of the tasks and how does client code look like.**

## Goals
- Develop a foundational understanding of how scheduling is done.
- Learn the common terminology used by most schedulers.
- Practice planning a self-study day and estimating time for learning.

:warning: **Note:**
- This is a self-study day. Independence and time management are essential.
- Many newcomers struggle with self-study; take a moment to plan your day and stick to it.
- Understand the **big picture** of each concept. If you can't explain it, you probably haven't learned it.
- Be prepared to describe how concepts relate to one another and to real-world scenarios.
- When in doubt about what you need to learn, ask your mentor.

### Notes
- workflow - work floowing from one stage to the next.
- jinja - template engine. Use jinja2 for python3, jinja is deprecated. The template separates the structure from its input data, which means that can reuse the same structure without restarting from scratch. Used in python projects (Flask, Django, Ansible).
- dag - model which wraps everything needed to execute a workflow. Dag is how Airflow represents a workflow.
- sdk - software development kit, collection of downloadable tools. A package.
- operator - a template for a predefined task. (BashOperator, PythonOperator.
- @task decorator is recommended over the classic PythonOperator.
### Core Concepts

Think through the following questions; by answering them you’ll touch every major topic listed above:


1. **Airflow User API & Concepts:** Explain the difference between a DAG and a DagRun? How do tasks share small metadata versus global configuration? What is Jinja Templating, and why would you use {{ ds }} instead of Python's datetime.now()? Contrast the TaskFlow SDK with Classic Operators. How does the TaskFlow SDK handle XComs differently than the old xcom_pull method? What are Assets? What types of Operators exist? Why is it not recommended to run any time consuming code in top level dag code? How does this affect the DAG Processor's performance? What is a Hook? what is the connection between Hooks, Connections and Operators?
- Dag vd GagRun
    - Dag - model which wraps everything needed to execute a workflow. Dag is how Airflow represents a workflow.
    - DagRun - an instance of a Dag. You can have many runs of a Dag at a time. When a Dagrun is created all tasks inside it are executed. DagRun statueses can be `success`, `failed`, `skipped`, according to it's tasks.
- task sharing small metadata vs global configuration -
    - small metadata - shared by xcom (cross communication) stroed in the airflow db (postgress, mySql), used for small data and metadata that needs to be shared between tasks.
    - global configuration - shared by variables, general key/value store that can be set via Airflow's user interfce, or a JSON file. You can add variables via the ui, cli and by code. Airflow automatically encrypted variables in the Airflow metastore which contains strings like (access_token, api_key, password, etc). Global variables are shared to all the entire installation while team based Variables related on one specific team.
- Jinja Templating - template engine. Use jinja2 for python3, jinja is deprecated. The template separates the structure from its input data, which means that can reuse the same structure without restarting from scratch. You should use Jinja templating instead of Python's methods like datetime.now(), because it delays reading the value until the task execution. Jinja templates do not produce a request until a task is running, whereas in python method it will execute every time and wont provide the logical date.
- TaskFlow SDK vs Classic Operators - TaskFlow SDK decouples the Dag authoring from Airflow internals, by providing a stable interface for writing Dags across Airflow versions. In classic operators you have to do xcom pull/push while in Task SDK you can pass data just by setting your task dependencies.
- Assets - needs to be defined in a Dag and use them to create cross-Dag dependencies. Asset is an object in Airflow that represents a concrete or abstract data by a unique name (URI can be attached) which represents data entity of file or object storage or table. Dag can be scheduled with trigger on an asset event is being created.
- Assets partitioning (3.2, makes this granular) downstream Dags trigger only when specific partition they care about gets updated.
- Operator Types - a template for a pre-defined task. BashOperator, PythonOperator, EmailOperator, HttpOperator, SQLExecuteQueryOperator, etc.
- Top level code refers to any code that isn't part of the DAG or operator instantiations, for example making requests to external systems. This is problematic because airflow's dag_folder executes all code in it every 30 seconds by default, it can cause performances issues sice there requests and connections are being made every 30 seconds instead of only when the DAG is scheduled to run. It can stress both Airflow infrustructure and the system you're doing the requests. For avoiding it you can use templating inside the operator.
- Hooks - Interfaces to external platforms and databases that lets you quickly and easily communicate without having a low level code.
- Connection between hooks, connections and operator - hooks integrate with Connection to gather credentials and hooks also often the building blocks that operatora are built out of.
 
2. **Airflow Backend & Architecture:** What are the different components in the airflow architecture? Define the roles of each component. Why is the Executor considered a mechanism/logic rather than a standalone service? Explain the Deferrable Operator. Which component makes these possible, and how do they save money/resources in a Big Data stack? What are Airflow Providers?
- Airflow Architecture -
    - scheduler - responsible of triggering scheduled workflows and submitting Tasks to the executir to run.
    - Dag processor - parses Dag files and serializes them into the metadata database.
    - sensors - special type of opertor that are desighned to do exactly one thing - wait for something to occur.
    - Web server - ui th trigger debug and inspect the behaviour of Dags and Tasks.
    - /Dags - folder that is read by the scheduler to figure out what tasks to run and when.
    - metadata database - Postgresql or mySql, stores state of tasks, dags and variables, xcom data.
    - worker - which executrs the tasks giver by the scheduler.
         - CeleryExecutor - a task queue, The Celery Executor distributes the workload from the main application onto multiple celery workers with the help of a message broker such as RabbitMQ or Redis.
         - KubernetesExecutor - Runs a fixed-single Pod as the scheduler that only requires access to the Kubernetes API. The pods only run when tasks are required to be executed, which helps to save resources when there are no jobs to run. In other executors, the workers are statically configured and are running all the time, regardless of workloads. 
         - LocalExecutor - Airflow tasks run locally within the scheduler process. (easy to use but limited in capabilities).
    - triggerer - executes deferred tasks in an asyncio event loop. If there are no deferred tasks this is not necessary.
    - plugins - extand airflow's functionality, a set of tools to parse Hive logs and expose Hive metadata.
- Executor considered a mechanism/logic rather than a standalone service because it can run in multipule variations, locall and remote, parallel and sequential.
- Deferrable operator - to improve resource utilization and not use sensors which locking the resources instead of other operators running, an operator can suspent itself and free up the worker for other processes. When operator defers, execution moves to the triggerer and the trigger specified by the operator will run.
- Which component makes these possible? by the triggerer.
    - A task instance (running operator) reaches a point where it has to wait for other operations or conditions, and defers itself with a trigger tied to an event to resume it.
    - The new trigger instance is registered by Airflow, and picked up by a triggerer process.
    - The trigger runs until it fires, at which point its source task is re-scheduled by the scheduler.
    - The scheduler queues the task to resume on a worker node.
- How do they save money/resources in a Big Data stack? dynamic allocation
- What are Airflow Providers? the capabilities of Airflow can be extended by installing additional packages, called providers. They can contain operators, hooks, sensor and transfer operators to communicate with a multitude of external systems or extend Airflow core with new capabilities.

3. **Airflow Workflow Synchronization:** How were DAGs typically synchronized to the Scheduler and Workers in Airflow 2? What where the risks with the approach? How was this solved in Airflow 3? How did it solve the main issue with the Airflow 2 approach? What are the other advantages DagBundles give us?
- In Airflow 2 Airflow loads DAGs from Python source files, which it looks for inside its configured /dags folder. It will take each file, execute it, and then load any DAG objects from that file. The Dags were required to be in one place on the local disk, and getting the Dags there was solely the responsibility of the deplyment manager. In contrast to airflow 3 which provide more flexible options with Dag bundles - a collection of one or more Dags (files along with thier associated files. There can be multipule types of Dag Bundles. `LocalDagBundle` reference a local directory containing Dag files, `GitDagBundle` allowing Airflow to fetch Dags directly from the repository and supports versioning. `S3DagBundle` reference an S3 bucket containing Dag files, dont support versioning and `GCSDagBundle`bucket containing Dag files.

4. **Airflow Task Lifecycle:** What is the full flow of a dag from being written to being run? What happens when the DAG Processor encounters your file? How is Jinja parsing different in dag processing than execution time? At which state does the Scheduler stop managing the task and hand it over to the Executor? What is the flow when a task gets to a worker? when does it become running?
- Parsing - the scheduler parses the DAG from the /dags folder and identified the task.
- Scheduling - the scheduler obsevs that all upstream dependencies for this task are complete. It then creates a queued  entry for the task instance in the metadata database.
- Dispatch - The executor checks the database for queued tasks. (In a celery based setup, it publishes a message containing the tasks information to the message queue.
- Execution - worker processes task.
- State update - the worker updates the task state in the metadata database (running, success, failed, skipped
- I explained the jijna parsing earlier.

5. **Airflow Critical Sections:** What is the "Critical Section" of the Scheduler? Describe the three primary "loops" or critical sections (DagRun Creation, Task Instance Creation, Task Scheduling).
- Critical Section - is where TaskInstances go from scheduled state and are enqueued to the executor, whilst ensuring the various concurrency and pool limits are respected. The critical section is obtained by asking for a row-level write lock on every row of the Pool table.
- DagRun creation - 
- Task Instance Creation - 
- Task Scheduling - 

### Q&A Questions

1. What defines an asset? a URI which formatted as a string and provide the unique name for it, Thats how I know two assets refers to the same thing.
2. assets terms -
- Dags can be scheduled based on asset events created for one or more assets.
- inlets - task parameter. contains the list of assets a specific task has access to.
- outlets - task parameter. contains the list of assets a specific task produces updates to.
3. Operator types: sensor, emptyOperator, taskSDK operator, deffered operator.
4. Callback - component of logging and monitoring. Yo may want to invoke a callback when your Dag succeeds or fails.
5. Connections, Operators and Hooks (used to access an external systems)
- Connections - Named configuration that stores credentials and endpoint information to the external system (instead of hard-coding passwords or tokens in our DAG). Connections can be reuse.
- Hooks - Interface. An abstraction to how we access the external system. How we interact with this connection, retrieve the credentials from the Airflow connection object.
- Operators - abstraction of the task. When we create operator, we can use the hook as part of the functionality of the operator. For example, we have a MyDatabaseOperator that when it used we connect to the database with the hook which needs the connection as a parameter.
6. FileProcessor - 
7. Backfill vs Catchup, designed to handle past DAG runs:
- Backfill - manually triggering DAG runs for a specific past date range. `airflow dags backfill -s <START_DATE> -e <END_DATE> <DAG_NAME>`. Allows you to skip tasks that have already succeeded and rerunning those that need attention.
- Catchup - automatically schedules DAG runs for all previous, unexecuted intervals when a DAG is first deployed or if it has been paused for a while. Used for ensuring continuity or don't miss any data in batch processing. 

### Real-World Context
Rather than focusing on one technology, think about how data workflows are shceduled, and think about when running and ocrhestrating data workflows.

## 🔄 Alternatives

Assignment: You are required to research and write a comparative analysis between Airflow and an industry alternative.

    Deliverable: A written summary (minimum 1 or 2 sentences).
    Focus: Compare performance, architecture, and specific "pain points" this tool solves compared to legacy systems or competitors.
    Goal: You must be able to justify why the department uses this tool for our specific environment.

## 🎯 User Story & Scenario

Assignment: Based on your research and understanding of the department's pipeline, define a concrete Use Case for this technology.

    Deliverable: A written summary example/story (two sentences approx.).
    Requirement: Describe a real-world scenario (e.g., a specific client requirement) where this technology is the optimal solution.
    Data Flow: Map out the data flow and explain how this tool integrates with other components in the Data Pipeline.


## Wrapping Up :trophy:
Discuss your answers and any areas of confusion with your mentor. Reflect on how these general concepts will help when you later when using scheduled jobs.

## Additional Topics from Review
- A deep dive into the Airlfow database and the inner workings of Airflow.
- A deep dive into bugs solved and unsolved inside Airflow.

## Action Items
- Review your notes and identify topics you want to explore deeper.
- Collect a list of real-world schedulers and their algorithms.
- Prepare questions for the upcoming mentor Q&A session.

## Recommemded Resources
- [Airflow Docs](https://airflow.apache.org/docs/)
