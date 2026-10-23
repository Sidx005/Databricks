**What is Databricks, and why do we use it instead of running Apache Spark directly on our local machine?**
Think about:
- What Databricks provides beyond Spark.
- How it helps with large-scale data processing.

**Ans:**
Databricks is an all-in-one data processing and analytics platform that provides Apache Spark along with managed compute clusters, data storage integrations, collaborative notebooks, job scheduling, and other data engineering tools.

It helps process large amounts of data by distributing workloads across multiple machines in a cluster rather than relying only on the CPU cores of a single local machine. It also simplifies cluster management and makes it easier to scale processing as data grows.


**What is the difference between a Spark driver and Spark executors?**
**Ans:**

**Driver:** Runs the main application, creates the execution plan, divides the job into tasks, and schedules them on the executors.
**Executors:** Run tasks on the data partitions, perform computations, and return results or status information to the driver.
