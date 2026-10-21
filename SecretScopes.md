<img width="991" height="526" alt="image" src="https://github.com/user-attachments/assets/4f4f507b-3a1e-465f-aa0e-ecf5bb132cee" />


<img width="989" height="516" alt="image" src="https://github.com/user-attachments/assets/be68bc50-0e95-43b4-af4c-5a994c54d874" />


<img width="977" height="562" alt="image" src="https://github.com/user-attachments/assets/c052b257-b397-4bc4-935d-1537e2f1cc58" />


<img width="993" height="376" alt="image" src="https://github.com/user-attachments/assets/351a7548-b094-4e37-957e-e8e7a29901ce" />



# CLI

<img width="684" height="486" alt="image" src="https://github.com/user-attachments/assets/9b0a3068-23d2-43c5-909c-482c79aee544" />


<img width="1282" height="685" alt="image" src="https://github.com/user-attachments/assets/e57bc246-963e-407c-859e-981ff926ee12" />


<img width="218" height="126" alt="image" src="https://github.com/user-attachments/assets/6d2b2af4-6af3-417a-a419-0dc578ab3d69" />


<img width="906" height="409" alt="image" src="https://github.com/user-attachments/assets/69e77170-6c89-49b0-89b1-a4a310701ecc" />

<img width="450" height="324" alt="image" src="https://github.com/user-attachments/assets/fb63080e-7aac-4b0b-b29b-89e83a20e1cd" />

<img width="577" height="284" alt="image" src="https://github.com/user-attachments/assets/fa3aba12-a770-4ff1-8ce1-a6dadf4652e1" />


<img width="379" height="141" alt="image" src="https://github.com/user-attachments/assets/d247b7be-68c8-4a08-bc47-47c54530eddd" />

<img width="201" height="38" alt="image" src="https://github.com/user-attachments/assets/b0d4b939-689c-4f5c-98d0-c03c7c04824a" />


<img width="754" height="205" alt="image" src="https://github.com/user-attachments/assets/bf03bdf0-b049-4bce-91c9-4eee28dccb19" />



## Using Scopes in Notebook

<img width="1005" height="478" alt="image" src="https://github.com/user-attachments/assets/b0e03b1e-3e70-44e9-be4b-ab0ec3f3184f" />

<img width="755" height="294" alt="image" src="https://github.com/user-attachments/assets/c9fa69bf-d6d4-4da1-894c-808ec4c8c223" />



<img width="647" height="555" alt="image" src="https://github.com/user-attachments/assets/124dc1fc-849d-4c30-87f9-156e9e4a794b" />


## Creating Key Vault

<img width="664" height="526" alt="image" src="https://github.com/user-attachments/assets/5d60a71f-101c-4a13-9927-be4f9b7b5b12" />

<img width="556" height="495" alt="image" src="https://github.com/user-attachments/assets/fea13653-ec45-4c74-bf31-dceda1195fd2" />


<img width="1024" height="526" alt="image" src="https://github.com/user-attachments/assets/5442c7cc-d3fe-4014-a79d-fa1d815f6db7" />

<img width="906" height="478" alt="image" src="https://github.com/user-attachments/assets/76ff75a7-d56a-493e-a54e-e35c2e3735f4" />




# Databricks Secret Scopes — Notes and Hands-on

## 1. What are Secret Scopes?

A Secret Scope is a secure storage container in Databricks used to store sensitive information such as:

- Access keys
- Passwords
- API tokens
- Database credentials
- Account names and other configuration values

A scope is a logical container that groups related secrets. A workspace can have multiple scopes, and each scope can contain multiple secrets.

## 2. Why use Secret Scopes?

Never hardcode credentials directly in your notebook or application code.

Bad practice:

```
access_key = "my_actual_storage_access_key"
```

Good practice:

```
access_key = dbutils.secrets.get(    scope="my_scope",    key="access_key")
```

Benefits:

- Security: Avoid exposing credentials directly in source code.
- Reusability: Access the same secrets from multiple notebooks.
- Environment portability: Use different scopes or values for development, testing, and production.
- Access control: Restrict which users or groups can access each scope.

## 3. How Secret Scopes work

Databricks Workspace

Secret Scope

Example: `my_scope`

`access_key`

`db_password`

Notebook / Application

Retrieves secrets using `dbutils.secrets`

There are two storage backends discussed in the lesson:

| Backend                | Description                                                                    |
| ---------------------- | ------------------------------------------------------------------------------ |
| Databricks-backed      | Secrets are managed and stored through Databricks' internal storage.           |
| Azure Key Vault-backed | Secrets are stored in Azure Key Vault and accessed through a Databricks scope. |

Recommendation from the lesson: Use Azure Key Vault when available, because it supports centralized secret management across environments.

## 4. Create and manage Secret Scopes using the CLI

The lesson demonstrates using the Databricks CLI to manage scopes and secrets.

### Step 1: Install the CLI

The video uses an older CLI installation method:

```
pip install databricks-cli
```

Authenticate using the legacy token-based command:

```
databricks configure --token
```

You provide:

- Databricks workspace URL (host)
- Personal access token

The token is generated under workspace user settings. Treat it like a password and never share it.

Version note: These are legacy CLI commands from the lesson. Modern Databricks CLI versions use different authentication and command syntax; see the current CLI documentation before copying them into a new environment.

### Step 2: List existing scopes

Legacy CLI:

```
databricks secrets list-scopes
```

This lists the secret scopes available in the workspace.

### Step 3: Create a scope

```
databricks secrets create-scope --scope my_scope
```

Example structure:

- Scope: `my_scope`
- Secret key: `access_key`
- Secret value: your actual storage access key

### Step 4: Add a secret

```
databricks secrets put --scope my_scope --key access_key
```

The CLI prompts you to enter the secret value in an editor.

Important: The secret key is the name used to retrieve a secret; the secret value is the actual sensitive credential.

### Step 5: List secret keys

```
databricks secrets list --scope my_scope
```

This lists the secret names, not their actual values.

## 5. Retrieve secrets inside a notebook

Use the `dbutils.secrets` API.

```
access_key = dbutils.secrets.get(    scope="my_scope",    key="access_key")
```

To use the secret in a storage connection, retrieve it first and then pass it to the relevant configuration.

For example, the lesson uses a storage account access key to read a CSV file from Azure Blob Storage.

Conceptually:

1. Retrieve the access key from the secret scope.
2. Configure the storage connection.
3. Read the required file.

The exact storage configuration depends on the authentication method and storage setup.

### Secret output redaction

When Databricks recognizes a retrieved secret in notebook output, it redacts the value to help prevent accidental exposure.

However, redaction is not a security boundary. A user who has permission to access a secret may be able to retrieve and misuse it. Never print credentials intentionally or expose them in logs.

## 6. Secret Scope permissions (ACLs)

Access Control Lists (ACLs) control who can access or manage secrets in a scope.

The lesson describes three permission levels:

| Permission | Purpose                                     |
| ---------- | ------------------------------------------- |
| `MANAGE`   | Manage the scope, secrets, and permissions. |
| `WRITE`    | Create or update secrets.                   |
| `READ`     | Retrieve secret values.                     |

Legacy CLI examples:

```
databricks secrets list-acls --scope my_scope
```

To grant access:

```
databricks secrets put-acl \
  --scope my_scope \
  --principal user@example.com \
  --permission READ
```

The principal can be a user or supported group identity.

The lesson notes that granular secret-scope permissions require the Databricks Premium plan in its demonstrated environment. Verify current workspace and plan requirements before implementing ACLs.

Best practice: Separate scopes by audience or purpose, and give each team only the permissions it needs.

## 7. Databricks-backed vs. Azure Key Vault-backed scopes

Databricks-backed scope

- Secrets are managed through Databricks.
- Convenient for workspace-level secret management.
- The lesson creates and manages it through the CLI.

Azure Key Vault-backed scope

- Secret values are stored in Azure Key Vault.
- Centralizes secret management in Azure.
- Secret values are created or updated in Key Vault.
- Databricks provides access through the configured scope.

### Creating an Azure Key Vault-backed scope

The lesson follows this workflow:

1. Create an Azure Key Vault.
2. Obtain its DNS name and resource ID from its properties.
3. Open the Databricks secret-scope creation URL:
   ```
   https://<databricks-workspace>/#secrets/createScope
   ```
4. Enter the scope name and Key Vault details.
5. Create the scope.
6. Add the required secret in Azure Key Vault.
7. Retrieve it from the notebook using the new scope name.

Example:

```
access_key = dbutils.secrets.get(    scope="my_scope_keyvault",    key="access_key")
```

The notebook retrieval code stays the same. You change the scope or the underlying secret configuration, rather than hardcoding credentials into your notebook.

Azure Key Vault integration details and scope-management interfaces can vary by Databricks workspace and cloud configuration.

## 8. Important points to remember

- A scope is a container; a secret is a named sensitive value inside it.
- `dbutils.secrets.get()` retrieves a secret inside a notebook.
- The CLI manages scopes, secret keys, and ACLs.
- Secret redaction helps prevent accidental disclosure but does not replace access control.
- Separate scopes by team, audience, or application purpose.
- Never hardcode credentials in notebooks, source files, or Git repositories.
- Azure Key Vault is useful when secrets need centralized management.
- Follow the current Databricks CLI syntax for your installed version.

## 9. Quick revision

## Test yourself

1\. What is a Secret Scope?

A compute cluster

A logical container for secrets

A Delta table

2\. Which API retrieves a secret in a notebook?

dbutils.fs.get()

dbutils.notebook.run()

dbutils.secrets.get()

3\. Does output redaction make a secret completely secure?

Yes, always

No, access permissions still matter

Only on SQL warehouses

4\. Where are values stored in a Key Vault-backed scope?

Azure Key Vault

The notebook source code

A Spark DataFrame

Score: 0 / 4

Reset quiz
