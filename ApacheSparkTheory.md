# Databricks Notes: Apache Spark and Introduction to Databricks

## 1. What is Apache Spark?

Apache Spark is a fast, general-purpose distributed computing engine used to process large amounts of data across multiple machines.

The key concept is distributed computing.

### Why do we need distributed computing?

Imagine processing increasingly large datasets:

- 1 GB: A normal laptop can generally handle this easily.
- 1 TB: Processing may require significant memory and computing resources.
- 100 TB: A single machine may become impractical due to resource limitations, processing time, and reliability concerns.

There are two approaches to handling larger workloads.

1\. Vertical scaling (Scaling Up)

Increase the resources of a single machine, such as CPU, RAM, and storage.

Limitation: A machine has a maximum capacity, and it can become a single point of failure.

2\. Horizontal scaling (Scaling Out)

Distribute the workload across multiple machines.

Example: Divide a 100 TB dataset across 10 machines, with each machine processing approximately 10 TB, assuming the workload can be evenly distributed.

### Vertical vs. Horizontal Scaling

[Vector Server Flat Icon.](https://images.openai.com/static-rsc-4/kVPIBi68-h2YHSizbqRAZB75vGyyb96UsNt8WvN_2ldZnvL_g4asehb4rwvdW0MM5mIS6ZHFbLJ7haQDaalrs9jFUkXbq4VPw1nygT3-0bQl_fHw-XV0RyXX1lkvQ-jMCq0GlXLM0t0OM2dU1DP_6_44bvD9CVXS7BVwSsNySE_orH5QGVGTz6Ux2nCmKJ5n?purpose=inline)

Vertical scaling

One machine with more CPU and RAM.

[server room](https://images.openai.com/static-rsc-4/5E-oy6y7DByDttgJRfTjRtnt-7eLcJekeyY5RgvbiW_sLTbmBBehBoNSXjgAIiKm0NyRl7Khzu4eHupGWNMqPoEi2fvFa0mloSVf8W5qQuq2O-PE2FMch4A46f4HgfqVBmHW3rPwLmfVveK6XyKv4IZASFhB5hPCX3bpLiPK9EJPwzppl3obh6xXvzAfhpBq?purpose=inline)

Horizontal scaling

Multiple machines sharing the workload.

Horizontal scaling can improve processing speed and resilience. However, fault tolerance depends on data replication, task retries, and how the system is configured.

## 2. Apache Hadoop MapReduce vs. Apache Spark

Before Spark, Hadoop MapReduce was widely used for distributed data processing.

| Feature           | Hadoop MapReduce                             | Apache Spark                                              |
| ----------------- | -------------------------------------------- | --------------------------------------------------------- |
| Processing        | Map and Reduce stages                        | General-purpose distributed processing                    |
| Intermediate data | Commonly writes intermediate results to disk | Can reuse cached data in memory                           |
| Performance       | Often slower for iterative workloads         | Often faster for iterative and multi-stage workloads      |
| Memory usage      | Relies heavily on disk-based processing      | Uses memory and disk                                      |
| Fault tolerance   | Replication and task re-execution            | Lineage, task re-execution, and other recovery mechanisms |

Why is Spark often faster?

Spark can keep intermediate results in memory instead of repeatedly writing them to disk. This reduces disk I/O, especially for iterative computations.

Important: Spark does not process everything in memory. It can spill data to disk when necessary.

## 3. What is an Apache Spark cluster?

A Spark cluster is a group of machines that work together to execute distributed data-processing tasks.

For example, processing 100 TB of data may require multiple machines to run tasks in parallel.

A Spark cluster generally involves:

- Driver: Coordinates the Spark application and schedules work.
- Executors: Execute tasks and may cache data.
- Cluster manager: Allocates computing resources to applications.

The exact infrastructure and cluster-management responsibilities depend on the deployment environment.

## 4. What is Databricks?

Databricks is a unified data and AI platform built around technologies including Apache Spark. It provides tools for data engineering, analytics, data science, machine learning, and AI development.

Managing Spark infrastructure yourself can require provisioning machines, configuring networking, managing clusters, handling scaling, and maintaining the environment.

Databricks simplifies much of this work by providing managed computing infrastructure and an integrated development environment.

Databricks was founded by the creators of Apache Spark.

### What does Databricks provide?

| Component                      | Purpose                                                                    |
| ------------------------------ | -------------------------------------------------------------------------- |
| Compute and clusters           | Run distributed data-processing workloads                                  |
| Notebooks                      | Develop and execute Python, SQL, Scala, and other supported code           |
| Photon                         | Accelerates supported workloads using a native vectorized execution engine |
| Unity Catalog                  | Centralized data and AI governance, permissions, and discovery             |
| Delta Lake                     | Reliable data storage with transactional capabilities                      |
| Lakeflow Declarative Pipelines | Define and manage data pipelines declaratively                             |
| Databricks Jobs                | Schedule and orchestrate workloads                                         |
| SQL warehouses                 | Run SQL queries and support BI workloads                                   |
| Alerts and monitoring          | Monitor query results, job outcomes, and pipeline health                   |

These capabilities make Databricks more than a platform for running notebooks.

## 5. Apache Spark vs. Databricks

This is an important distinction to remember.

| Apache Spark                                                          | Databricks                                               |
| --------------------------------------------------------------------- | -------------------------------------------------------- |
| Open-source distributed computing engine                              | Managed data and AI platform                             |
| Provides distributed data-processing capabilities                     | Provides Spark plus integrated platform services         |
| Can run in different environments                                     | Provides managed compute options and development tools   |
| Infrastructure may need to be configured separately                   | Simplifies many infrastructure-management tasks          |
| Does not inherently provide the full Databricks governance experience | Integrates Unity Catalog and other platform capabilities |

Simple real-world analogy:

Apache Spark is the engine that processes the data. Databricks is the platform that provides the engine along with the tools, compute environment, governance, and workflows needed to build and operate data solutions.

Databricks does not eliminate all infrastructure management, and Spark can also run without Databricks.

## 6. Databricks in a nutshell

The evolution can be understood in three stages:

1. Distributed computing: Process large datasets across multiple machines.
2. Apache Spark: Make distributed data processing more flexible and efficient.
3. Databricks: Combine Spark-based processing with managed compute, collaborative development, data governance, orchestration, analytics, and AI capabilities.

The main takeaway is that Apache Spark provides distributed processing capabilities, while Databricks provides an integrated platform for developing and operating data and AI workloads.

Next topic: Databricks architecture — understanding how its control plane, compute plane, clusters, and workspace components fit together.
