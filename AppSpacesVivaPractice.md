# Viva Practice

**1. What are Databricks Apps?**
Databricks Apps is a platform that allows to build,deploy & run interactive data and AI applications directly within the Databricks environment.

It provides a managed serverless infrastructure, authentication, authorization & integration with services such as:
- Unity Catalog
- Databricks SQL
- Model Serving
- Genie
- OAuth

**eg:** Dashboards, AI Chatbots, data-entry applications and operational interfaces


**Q2. Why did databricks introduce Apps?**
To simplify the application hosting & integration with the existing Databricks Ecosystem.
Instead of developers needing separate infra to host apps, configure auhentication & manage dat access anf deployment environments.

**Q3. What is the difference between Databricks Apps and traditional web applications?**
Traditional web applications generally require developers to provision and maintain their own hosting infrastructure.

Databricks Apps provides managed serverless hosting and native integration with Databricks authentication, data governance and platform resources.

**Q4. Does Databricks Apps require a separate cloud server?**

No. Databricks manages the application hosting infrastructure through its serverless platform.

**Q5. What is a Databricks App runtime?**
It is the isolated execution environment in which an application runs. Each app has its own runtime environment and identity.

**Q6. Can a Databricks App access Unity Catalog?**
Yes. It can access Unity Catalog resources using either its own application identity or user authorization, depending on the architecture.

**Q7. What is the difference between an app and a workspace?**
A workspace is the environment containing Datbricks resources, users, notebooks, jobs and applications.
An app is an individual application deployed within that workspace.

**Q8. What is the difference between an app and a workspace?**
Yes. The developer can share an app with users or groups and assign permissions such as CAN USE and CAN MANAGE.

**Q9. Are Databricks Apps always running?**
Tradtional Apps use dedicated serverless instances with their configured compute behaviour. Serverless Micro Apps, on the other hand, can scale to zero when idle and restart when requested.

**Q15. What are the main components of a Databricks App?**
- Application source code.
- Application configuration.
- Dependencies.
- Runtime environment.
- Application identity.
- Resource permissions.
- Authentication and authorization configuration.
- Deployment configuration.


## App Spaces

**Q1. What is an App Space?**
  An App space is a governance boundary in Databricks that allows workspace administrators centraloy control:
  1. Who can create applications.
  2. What permissions applications can request.
  3. Which resources applications can access.
  4. Which user authorization settings apply.
  5. Which serverless usage policy appies.
  Instead of configuring every application independently, administrators configure governance settings at the App Space Level.

**Q2. Why do we need App spaces when Databricks Apps already exist?**
Databricks Apps provides application hosting.
App Spaces adds centralized governance around application creation, authorization scopes, resource access and serverless spending.


**Q3. What problem does App Spaces solve?**
Suppose an organization has Sales, Finance and HR teams.

Without centralized governance, different developers could create applications with inconsistent data access and security configurations.

With App Spaces, administrators can create separate governance boundaries for different teams and define their allowed permissions and resources.

**Q4. Can an App Space exist without Genie App Builder?**
During the documented Beta offering, no. Genie App Builder is the supported way to build applications inside App Spaces.

**Q5. Can Genie App Builder work without an App Space?**
No. During Beta, Genie App Builder creates applications inside an App Space.

**Q6. Can an existing traditional Databricks App be converted into a Serverless Micro App?**
Not through the documented Beta functionality. Serverless Micro Apps are created through Genie App Builder inside an App Space.



## Authentication , Authorization & OBO

**Q1. What is authentication?**
Authentication verifies the identity of the user attempting to access an application.
Example: A user signs in using Databricks SSO.

**Q2. What is authorization?**
Authorization determines what an authneitcated user or application is allowed to access.
eg: A Sales User can read Sales Data but cannot read Finance Data

**Q3. What authentication mechanism does Databricks Apps use?**
Databricks Apps uses OAuth 2.0-based authentication and authorization.


**Q4. What is OBO Authorization?**
OBO stands for On-Behalf-Of authorization.
It allows an application to access Databricks resources using the identity and permissions of the user currently interacting with the application.

A Sales employee opens an application. The app accesses Sales data using the employee's identity, rather than using a shared application identity.

**Q5. What is OAuth Scopes?**
An OAuth scope deifnes the specific API or resource permissions that an application can request when acting on behalf of a user.
It limits the functionality the application can access through user authentication.

**Q6. Why are scopes required?**
Scopes follow the principle of least privilege.
Even if a user has extensive Datbricks permissions,an aplication cannot use user authorization to access APIs outside its approved scopes.

**Q7. What are some examples of OAuth Scopes?**
It is the default colection of 11 user authorization scopes available when creating an App Space through the documented configuration.
These include:
- sql (Databricks SQL functionality)
- sql:restricted-query (Read-only SQL query access)
- genie (Genie functionality)
- model-serving (Model serving)
- files (File operations)
- model-serving (Model Serving)
- vector-search (Vector Search)
- catalog.tables:read (Read Catalog Table Metadata)
- catalog.schemas:read (Read schema metadata)
  
**Q8. What happens when a user first accesses an application requesting OAuth scopes?**
The user is prompted to provide consent for the requested scopes.
Consent is granted for the application and the associated set of scopes. Users can review and revoke their consent.


## Bundles

Important Commands:
```
# Validate configuration
databricks bundle validate -t dev

# Deploy resources
databricks bundle deploy -t dev

# Run a deployed resource
databricks bundle run <resource-name> -t dev

# Destroy bundle-managed resources
databricks bundle destroy -t dev

```
