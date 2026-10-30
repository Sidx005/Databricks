# Pricing Tiers

Azure Databricks provides two pricing tiers:

## Standard
- Provides basic platform capabilities.
- Suitable for basic requirements.

## Premium
- Provides additional enterprise capabilities.
- Includes features such as:
  - Role-Based Access Control (RBAC)
  - Audit Logs
  - Cluster Policies
  - Additional governance features

**Remember:**
- Pricing tier determines the capabilities and governance features available.
- It does not determine the compute size or type.

# Classic Compute

Classic compute uses infrastructure provisioned in your Azure environment.

You configure:
- Virtual machine type
- Number of workers
- Cluster size
- Databricks Runtime configuration

## Classic Compute Pricing

Classic compute generally has two major cost components:

1. Databricks DBU consumption cost.
2. Underlying Azure Virtual Machine cost.

**Classic Compute Cost = DBU Cost + Azure VM Cost + Applicable Additional Costs**

The longer the infrastructure runs, the more infrastructure charges may accumulate.

# Serverless Compute

With serverless compute, Databricks manages the underlying compute infrastructure.

You do not need to provision or manage virtual machines in the same way as classic compute.

## Key Characteristics

- Infrastructure is managed by Databricks.
- Less infrastructure management for users.
- Simpler operational experience.
- Users can focus on workloads instead of VM configuration.
- Serverless usage is still billable.

**Remember:** Serverless does not mean free.

# Classic vs Serverless Compute

| Parameter | Classic Compute | Serverless Compute |
|---|---|---|
| Infrastructure management | User manages configuration and provisioning | Databricks manages infrastructure |
| VM configuration | User selects VM types and worker count | No direct VM management in the same way |
| DBU charges | Applicable | Applicable |
| Azure VM charges | Generally applicable | Underlying infrastructure is managed by Databricks |
| Operational complexity | More infrastructure management | Simpler operational experience |
| Flexibility | More control over compute configuration | Depends on supported serverless capabilities |

## When to Choose Classic Compute?

Choose classic compute when:
- You need greater control over infrastructure.
- Your workload requires specific VM or cluster configurations.
- Your workload has requirements not supported by serverless.

## When to Choose Serverless Compute?

Choose serverless when:
- You want Databricks to manage the infrastructure.
- You want a simpler operational experience.
- Your workload is supported by serverless compute.

**Important:**
- Serverless is not automatically cheaper.
- Classic is not automatically more expensive.
- Actual cost depends on workload requirements, runtime, DBU consumption, and additional infrastructure charges.

# Cost Control Best Practices

Five practices to prevent unnecessary Databricks costs:

1. **Stop unused interactive compute:** Avoid leaving development clusters running unnecessarily.
2. **Right-size compute:** Choose a configuration that matches your workload instead of overprovisioning.
3. **Use job-oriented compute:** For automated workloads, use appropriate job compute rather than interactive development clusters.
4. **Monitor DBU consumption:** Regularly review Databricks usage and Azure infrastructure costs.
5. **Configure Azure Cost Management budgets and alerts:** Set spending budgets and notifications to track costs.

For example, an organization could configure a $2,000 budget and receive an alert when spending approaches or exceeds the configured threshold.

# Summary

| Concept | What to Remember |
|---|---|
| DBU | Normalized unit used to measure Databricks platform consumption |
| DBU Consumption | DBU Rate × Runtime |
| DBU Cost | DBU Consumption × Price per DBU |
| Standard Tier | Basic platform capabilities |
| Premium Tier | Additional enterprise and governance capabilities |
| Classic Compute | More infrastructure control; generally includes DBU and Azure VM costs |
| Serverless Compute | Databricks manages underlying infrastructure; usage is still billable |
| Cost Management | Monitor usage, right-size compute, and configure budgets and alerts |

## Important Points

- DBU cost and Azure VM cost are separate components.
- Pricing tier is different from compute type.
- Serverless does not mean free.
- Serverless is not always cheaper than classic.
- Budget alerts notify you about spending; they do not automatically stop compute.
