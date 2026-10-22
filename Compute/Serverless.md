

# Databricks Serverless Compute — Notes & Hands-on

## 1. What is Serverless Compute?

Serverless compute allows Databricks to manage the compute infrastructure for you. You don't need to manually create, configure, start, or scale clusters for supported workloads.

Databricks automatically provisions and manages compute based on workload requirements.

You can use serverless compute for:

- Notebooks
- Jobs and workflows
- Lakeflow Declarative Pipelines (formerly Delta Live Tables / DLT)
- SQL warehouses, through their serverless option where supported

Key idea: You focus on your code and data; Databricks manages the underlying compute.

## 2. Classic vs Serverless Architecture

### Classic compute

Control plane

Workspace management, APIs, scheduling

Customer data plane

Customer-managed or configured cloud compute

Clusters process data and execute workloads

### Serverless compute

Control plane

Workspace management, APIs, scheduling

Serverless compute plane

Databricks-managed compute infrastructure

Processes data using supported connectivity and security configurations

Data sources and storage

Cloud object storage, tables and other connected data

| Feature                      | Classic compute                                   | Serverless compute                                    |
| ---------------------------- | ------------------------------------------------- | ----------------------------------------------------- |
| Infrastructure management    | More user configuration                           | Managed by Databricks                                 |
| Startup                      | Can take minutes                                  | Often starts in seconds                               |
| Scaling                      | Depends on cluster configuration                  | Managed automatically                                 |
| Cloud infrastructure billing | May include separate cloud infrastructure charges | Generally included in Databricks serverless pricing   |
| Cost tracking                | Cluster and workload cost attribution             | Budget policies and usage attribution where supported |

Important: Serverless compute runs in Databricks-managed infrastructure, but that does not mean your data itself is moved into the control plane. Data access, networking and security depend on the workload and configuration.

## 3. How Serverless Works

1\. Select serverless

2\. Databricks provisions compute

3\. Run notebook, job or pipeline

4\. Compute scales and is managed automatically

5\. Pay for serverless usage

Databricks may reuse ready compute capacity, which helps reduce startup time. Actual startup and execution times depend on the workload and environment.

## 4. Enabling Serverless Compute

If serverless is not enabled by default, an account administrator may need to enable the relevant feature.

1. Open the Databricks Account Console.
2. Go to Settings → Feature enablement.
3. Find the relevant serverless feature, such as serverless compute for workflows, notebooks and pipelines.
4. Enable it if available.
5. Return to your workspace and check whether serverless is available for your workload.

Some workspace types, including certain Express editions, may already have serverless enabled.

Regional availability matters. Check the current Databricks serverless availability documentation for your cloud provider, region, workload type and workspace configuration. Availability can change over time.

## 5. Budget Policies — Tracking Serverless Costs

A budget policy lets you attach tags to supported serverless usage so you can organize and attribute costs.

For example:

- Budget policy: `demo_budget`
- Tag key: `demo_bp`
- Tag value: `development`

These tags help identify usage in supported billing and usage records. A budget policy is not necessarily a spending cap and does not automatically stop workloads when a budget is exceeded.

### Create a budget policy

1. Open your workspace.
2. Go to Settings → Compute → Budget policies.
3. Select Manage → Create.
4. Enter a policy name, such as `demo_budget`.
5. Add a tag key and value.
6. Create the policy.
7. Open its permissions and grant other users the appropriate access, such as the ability to use the policy.

Then attach the policy to supported notebooks or jobs.

## 6. Hands-on: Serverless Notebook

### Step 1: Connect to serverless

1. Open or create a Python notebook.
2. Select Connect.
3. Choose Serverless.
4. Wait until the notebook shows that it is connected.

### Step 2: Run PySpark

```
display(spark.range(100))
```

This creates a Spark DataFrame containing numbers from `0` through `99` and displays the result.

### Step 3: Run SQL

```
%sql
SELECT *
FROM range(100);
```

This generates the same range using SQL.

### Step 4: Configure serverless

Depending on your workspace and interface, serverless settings may include:

- Environment: Select a supported environment version.
- Python memory: Choose the available standard or high-memory option, if offered. This is distinct from the memory available to Spark execution.
- Libraries: Install supported Python packages.
- Logs: Inspect available environment or compute logs.
- Budget policy: Attach a policy for usage attribution.

Not every configuration option is available for every serverless workload.

## 7. Using Serverless for Jobs

You can switch a supported job from classic compute to serverless.

1. Open Workflows → Jobs.
2. Select the existing job.
3. Edit the task's compute configuration.
4. Choose serverless where supported.
5. Attach a budget policy if required.
6. Save and run the job.

For multi-task jobs, check each task's compute configuration and any workload-specific limitations before switching.

## 8. Using Serverless for Lakeflow Declarative Pipelines

For supported pipelines:

1. Open Workflows → Pipelines.
2. Select the pipeline.
3. Open its settings.
4. Enable the serverless option, if available.
5. Save and run the pipeline.

Serverless pipelines can support managed scaling and additional capabilities depending on the pipeline configuration. Check current documentation for feature-specific support, including incremental refresh capabilities.

## 9. Cost and Auto-termination

Serverless is generally usage-based: charges depend on the compute consumed and the applicable pricing model.

- Serverless does not mean free.
- You generally do not pay separate customer-managed VM charges for the serverless infrastructure itself, but other cloud resources and services may still incur costs.
- Idle serverless notebook compute may automatically terminate after a period of inactivity.
- You can manually disconnect or terminate a notebook session when finished.
- Budget policies help with cost attribution, not necessarily cost prevention.

Since you're working with a Databricks trial, monitor your remaining credits and usage before running long jobs or large pipelines.

## 10. Limitations to Remember

- Serverless is not available for every region, workload or configuration.
- Python and SQL support in notebooks depends on the current serverless offering; do not assume all notebook languages are supported.
- Some Spark configurations, libraries, dependencies and low-level infrastructure customizations may be restricted.
- Network access, external data sources and security settings must meet serverless requirements.
- Existing classic jobs and pipelines may require configuration changes before switching.
- Availability and supported features evolve, so check the current documentation before deploying production workloads.

## Quick revision

| Question                                       | Answer                                                                     |
| ---------------------------------------------- | -------------------------------------------------------------------------- |
| What is serverless?                            | Databricks-managed compute for supported workloads                         |
| Who manages the infrastructure?                | Databricks                                                                 |
| Why does it start quickly?                     | Managed, readily available compute capacity can reduce provisioning delays |
| Where does classic compute run?                | In the customer data plane                                                 |
| Where does serverless compute run?             | In the Databricks-managed serverless compute plane                         |
| How are serverless costs tracked?              | Usage records and budget policies, where supported                         |
| Does serverless mean zero cost?                | No                                                                         |
| Can you use serverless for jobs and pipelines? | Yes, for supported configurations                                          |
| What should you check first?                   | Regional availability, workload support, permissions and pricing           |

One-line takeaway: Serverless compute removes much of the infrastructure management burden by letting Databricks provision, scale and manage compute for supported notebooks, jobs, pipelines and SQL workloads.


<img width="848" height="449" alt="image" src="https://github.com/user-attachments/assets/10566979-5b46-444d-9e47-8a304fbfaea1" />



# Budget Policy

<img width="819" height="348" alt="image" src="https://github.com/user-attachments/assets/3bd1b1d3-8cb0-4b95-ac6a-bdfa2d6ca1a3" />
