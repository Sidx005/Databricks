# Serverless Policies
Serverless policies also previously called budget policies are for tracking and attributiing costs of the resources used.
(eg: compute, jobs,noteboks,etc)


# Steps:
## 1. Go to Profile Settings
<img width="657" height="362" alt="image" src="https://github.com/user-attachments/assets/39ce1199-ab7e-4e9c-9e1a-16107970ba78" />


## 2. Navigate to Compute
<img width="130" height="239" alt="image" src="https://github.com/user-attachments/assets/f0a5c43b-f088-47f1-b1fe-04de0b55164d" />


## 3. Create a Budget Policy by clicking on Create Button

<img width="130" height="239" alt="image" src="https://github.com/user-attachments/assets/a37d97ff-f2b8-44c6-a13d-26948c929ef5" />

<img width="476" height="292" alt="image" src="https://github.com/user-attachments/assets/d3341fd9-da69-4f91-b082-94c2770e82cc" />

<img width="432" height="317" alt="image" src="https://github.com/user-attachments/assets/3d06f1b1-a281-4914-94d4-2b141a5996c8" />


## 4. Grant Access to Policy

<img width="582" height="274" alt="image" src="https://github.com/user-attachments/assets/e196293b-0ed6-421d-8f84-23632bd99163" />

<img width="560" height="295" alt="image" src="https://github.com/user-attachments/assets/1da1da6c-1f60-4013-90ea-78447bd07807" />


## 5. Using a Policy (in Notebook, Jobs,etc)
### eg: Notebook

<img width="601" height="334" alt="image" src="https://github.com/user-attachments/assets/edb911a4-7806-4c5c-810b-c76a3626450b" />

<img width="158" height="219" alt="image" src="https://github.com/user-attachments/assets/f8e4e9e6-3253-43a9-adca-41dae31e9bd5" />


<img width="453" height="231" alt="image" src="https://github.com/user-attachments/assets/02355fcd-a9a2-4616-a1be-1e016e7a78d9" />


### eg: Job

<img width="608" height="303" alt="image" src="https://github.com/user-attachments/assets/e239c70c-2f29-4396-93af-8d49adce41c8" />

<img width="323" height="288" alt="image" src="https://github.com/user-attachments/assets/cf843f10-0042-4960-a979-3fcfb542c58d" />

<img width="424" height="196" alt="image" src="https://github.com/user-attachments/assets/b2ba8b0b-132a-4b14-9932-1cc552347479" />



## Analyzing the Policies
After applying a policy to a notebook, job or a pipeline, any tafs contained in the policy are propagated in the system.billing.usage system table in the custom_tags 

### Query:
1. Querying the systems table
```sql
SELECT * FROM system.billing.usage;
```
This gives you a table with columns:
| Column name | Description |
|---|---|
| `account_id` | Unique identifier of the Databricks account. |
| `workspace_id` | Unique identifier of the workspace. |
| `record_id` | Unique identifier of the usage record. |
| `sku_name` | Name of the SKU associated with the usage. |
| `cloud` | Cloud provider (AWS, Azure, or GCP). |
| `usage_start_time` | Start timestamp of the usage interval. |
| `usage_end_time` | End timestamp of the usage interval. |
| `usage_date` | Date associated with the usage. |
| `custom_tags` | Custom key-value tags associated with the usage. |
| `usage_unit` | Unit in which usage is measured, such as DBU. |
| `usage_quantity` | Quantity of usage consumed. |
| `usage_metadata` | Additional metadata about the resource generating usage. |
| `identity_metadata` | Identity information associated with the usage. |
| `record_type` | Indicates whether the record is original, an update, or a retraction. |
| `ingestion_date` | Date the usage record was ingested into the system table. |
| `billing_origin_product` | Product that generated the usage, such as Jobs, SQL, or Apps. |
| `product_features` | Information about product features associated with the usage, including serverless indicators. |
| `usage_type` | Classification of the usage type. |
| `usage_policy_id` | Identifier of the serverless usage/budget policy associated with the usage, where applicable. |


2. Inspect usage by team
```sql
SELECT
    custom_tags['team'] AS team,
    usage_unit,
    SUM(usage_quantity) AS total_usage
FROM system.billing.usage
WHERE product_features.is_serverless
  AND custom_tags['team'] IS NOT NULL
GROUP BY
    custom_tags['team'],
    usage_unit
ORDER BY total_usage DESC;
```
3. Inspect usage by project
```sql
SELECT
    custom_tags['app_space'] AS app_space,
    custom_tags['team'] AS team,
    usage_date,
    SUM(usage_quantity) AS total_usage,
    usage_unit
FROM system.billing.usage
WHERE product_features.is_serverless
  AND custom_tags['app_space'] IS NOT NULL
GROUP BY
    custom_tags['app_space'],
    custom_tags['team'],
    usage_date,
    usage_unit
ORDER BY usage_date DESC;
```
