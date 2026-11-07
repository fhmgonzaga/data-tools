---
title: Connect and Query an Azure SQL Database Container
description: Create a local Azure SQL Database container with the SQL Server Management Studio (SSMS) wizard on Windows x64, connect, and run your first T-SQL queries.
author: vamahtani
ms.author: vamahtani
ms.reviewer: mbarickman
ms.date: 11/07/2026
ms.service: sql-server-management-studio
ms.topic: quickstart
ms.collection:
  - data-tools
ms.custom:
  - intro-quickstart
ai-usage: ai-assisted
# CustomerIntent: As a developer, I want to create and query a local Azure SQL Database container using SSMS, so that I can develop and test without an Azure subscription.
---

# Quickstart: Connect and query an Azure SQL Database container using SQL Server Management Studio (SSMS)

[!INCLUDE [asdb](../includes/applies-to-version/asdb.md)]

<!-- DRAFT: Public Preview is planned for November 17, 2026. Confirm release status, licensing, SSMS minimum version, and exact wizard labels before publication. The SSMS wizard starts Azure SQL Database containers and supports Windows x64 only, per product-owner guidance. Confirm article ownership with the SSMS documentation team. -->

Create a local Azure SQL Database container by using the SSMS provisioning wizard, then connect and run Transact-SQL (T-SQL) commands. You don't need to write container startup commands. The container is free for local development and testing and doesn't require an Azure subscription or credit card.

The SSMS container provisioning wizard supports **Azure SQL Database containers only**, not SQL Server containers. This SSMS workflow supports **Windows x64**. ARM devices aren't supported for this workflow.

> [!NOTE]
> Azure SQL Database container is in public preview. 

This quickstart demonstrates the following steps:

> [!div class="checklist"]
>
> - Create and start a local Azure SQL Database container with the SSMS wizard
> - Connect to the container in SSMS
> - Create a database and connect directly to it
> - Create a table, insert rows, and query the results
> - Verify the database and engine identity
> - Clean up the local resources

## What is an Azure SQL Database container?

A container packages the Azure SQL Database engine so that a container runtime can run it on your computer. Your application connects to a local address such as `localhost,1433`, rather than an Azure SQL logical server.

The container runs the Azure SQL Database engine, not the SQL Server container image. It supports Azure SQL Database T-SQL and the Tabular Data Stream (TDS) protocol used by SQL drivers. `SERVERPROPERTY('EngineEdition')` returns `5`, identifying the Azure SQL Database engine. Native vectors, vector search, and JSON let you explore AI-ready data scenarios locally.

The container doesn't provide the managed cloud service. Azure-managed backups, high availability, elastic pools, and serverless auto-pause are cloud capabilities, not local container features. For more information, see [What is the Azure SQL Database container?](https://microsoft.github.io/azure-sql-database-container/what-is-the-container.html).

## Choose your development database

| Option | Choose it when | Cost and account requirements |
| --- | --- | --- |
| Azure SQL Database container through SSMS | You want to learn SQL or build and test an application targeting Azure SQL Database locally on Windows x64. | Free for local development and testing. No Azure subscription or credit card required. |
| Azure SQL Database in Azure | You need a managed database, cloud integration, or a cloud validation environment. | Requires an Azure subscription. The [Azure SQL Database free offer](/azure/azure-sql/database/free-offer) has separate eligibility requirements and usage allowances. |

> [!IMPORTANT]
> Azure SQL Database containers are for development, testing, CI, and demos, not production. For production, deploy your application and database to Azure SQL Database in Azure. The free local container and the Azure SQL Database free offer are separate options.

## Prerequisites

- [Install SQL Server Management Studio](../install/install.md) on a supported **Windows x64** computer. Use a release that includes the Azure SQL Database container provisioning wizard. The minimum wizard version is pending confirmation.
- Install and start a container runtime that runs Linux containers. For Docker Desktop, use the Windows Subsystem for Linux 2 (WSL 2) backend. See [Docker Desktop installation on Windows](https://docs.docker.com/desktop/setup/install/windows-install/). The runtime must also be supported by the SSMS wizard; its complete runtime support list is pending confirmation.
- Allocate at least two CPU cores, 4 GB of memory, and 10 GB of free disk space to the container runtime.
- Make a local TCP port available for the database connection. This quickstart uses `1433`; use your actual host port if it differs.
- Have internet access for the initial image download. After the image is cached, local queries don't require a cloud database. External AI endpoints and other online services still require connectivity.

For engine resource requirements, see [container prerequisites](https://microsoft.github.io/azure-sql-database-container/prerequisites.html). The engine's broader platform support doesn't change the Windows x64-only scope of the SSMS container provisioning experience. ARM isn't supported for this SSMS workflow.

## Create an Azure SQL Database container

Use the SSMS wizard to create and start the container. The wizard provisions an Azure SQL Database container; there isn't a SQL Server container option in this SSMS experience.

1. Start SSMS and open the Azure SQL Database container provisioning wizard.

   **Draft placeholder:** Insert the verified SSMS menu or Object Explorer command here.

1. Review the local development container information. Make sure that your container runtime is installed, running, and configured for Linux containers before proceeding. Resolve any prerequisite errors shown by the wizard.

   <!-- DRAFT: Verify the SSMS prerequisite page, supported runtime choices, and exact navigation labels. Don't copy the MSSQL extension's runtime detection UI. -->

1. Configure the container and connection. Use the following values for this quickstart, where the wizard exposes the corresponding settings:

   | Setting | Tutorial value or guidance |
   | --- | --- |
   | Container name | `ssms-sqldb`, or another unique name |
   | Host port | `1433`, or another available local port |
   | SQL login | `sa` |
   | Password | A unique, strong password for the local database login |

   The password must contain at least eight characters and include characters from at least three of these categories: uppercase letters, lowercase letters, digits, and symbols. Store the password securely, not in source control or AI prompts.

   Record the actual container name and host port for reconnection and cleanup. Keep access limited to your local computer. If the wizard offers data persistence settings, review them before provisioning so that you understand whether database files remain after the container is removed.

   <!-- DRAFT: Confirm actual field names, which values are configurable, login defaults, port selection, loopback binding, and volume behavior. The table is guidance, not a verified inventory of SSMS fields. -->

1. Review the license terms and the proposed configuration. Start provisioning from the wizard and wait for the image download and database engine startup to complete.

   **Draft placeholder:** Insert the verified review page and action button labels here.

1. After the engine is ready, note the local server endpoint and continue with [Connect to the container in SSMS](#connect-to-the-container-in-ssms).

   If the wizard opens a connection automatically, check its database before using it. A running container, an SSMS connection, and a user database are different resources. Don't assume that provisioning the container has already created `TutorialDB`.

   <!-- DRAFT: Verify progress states, automatic connection behavior, startup database creation, and failure recovery in the release build. -->

### Wizard screenshots placeholder

**Placeholder: Add screenshots captured from the verified SSMS release build.**

| Screenshot to capture | Details to verify before adding the image |
| --- | --- |
| Wizard entry point | Actual menu or Object Explorer command and its label. |
| Prerequisite checks | Runtime detection, required resources, and actionable error messages. |
| Container configuration | Actual fields for image, name, port, authentication, and persistence. |
| Database configuration, if present | Whether the wizard creates a database and the supported database settings. |
| Review and provisioning | Actual review page, progress indicators, and failure handling. |
| Successful connection | Server endpoint, selected database, and resulting SSMS connection. |

<!-- DRAFT: Add Learn :::image directives only after the screenshots exist. Store them in media/ssms-connect-query-azure-sql-database-container and supply descriptive alt text. Don't include passwords or registry credentials in screenshots. -->

## Connect to the container in SSMS

1. Start SSMS. Open a Database Engine connection through **Object Explorer** > **Connect** > **Database Engine**.

1. Enter the following connection settings. The layout depends on your SSMS version.

   | Setting | Value |
   | --- | --- |
   | Server type, if shown | **Database Engine** |
   | Server name | `localhost,1433`, or the host port you selected |
   | Authentication | **SQL Server Authentication** |
   | Login | The SQL login configured during provisioning, such as `sa` |
   | Password | The local database password you chose in the wizard |
   | Database name / Connect to database | `master` |
   | Encryption | **Mandatory**, or enable **Encrypt connection** in older dialogs |
   | Trust server certificate | Selected |

   In dialogs that don't show the database field on the main page, select **Options** > **Connection Properties** and enter `master` under **Connect to database**. See [Connect to server (Connection Properties page)](../f1-help/connect-to-server-connection-properties-page-database-engine.md).

   > [!IMPORTANT]
   > Trusting the server certificate bypasses certificate validation for the container's self-signed certificate. Use this setting only for this local development connection. Don't apply it to production Azure SQL Database connections.

1. Select **Connect**. This workflow uses a local endpoint and SQL authentication, not an Azure logical server, Azure firewall rule, or Microsoft Entra sign-in.

1. Open **New Query** for this connection. Confirm that the query targets `master`, then select **Execute** or press **F5** to run:

   ```sql
   SELECT DB_NAME() AS DatabaseName;
   ```

   The result should be `master`. Use this connection only to provision or remove user databases.

## Create a database

Use `TutorialDB` as the example database name. Substitute your project's database name consistently if you choose another name.

If the wizard already created `TutorialDB`, skip the following statement and connect directly to that database. Otherwise, on the `master` connection, run:

```sql
CREATE DATABASE [TutorialDB];
GO
```

Wait for the command to complete before connecting to `TutorialDB`. If a database with that name already exists, use a different name throughout this quickstart or confirm that the existing database is safe to use. Don't delete an existing database just to repeat the tutorial.

Connecting to a database that doesn't exist doesn't create it. Use `master` for database provisioning and your user database for application queries.

## Connect directly to the new database

1. Open a new Database Engine connection in SSMS with the same server, SQL login, password, encryption, and certificate settings.
1. Enter `TutorialDB` in **Database name** or **Connect to database**, then select **Connect**.
1. Open **New Query** for this new connection and run:

   ```sql
   SELECT
       DB_NAME() AS DatabaseName,
       SERVERPROPERTY('EngineEdition') AS EngineEdition,
       SERVERPROPERTY('Edition') AS Edition;
   ```

   Expected results:

   | DatabaseName | EngineEdition | Edition |
   | --- | --- | --- |
   | TutorialDB | 5 | SQL Azure |

   If `DatabaseName` isn't `TutorialDB`, open a connection that explicitly targets it before continuing. If `EngineEdition` isn't `5`, check that you started the Azure SQL Database image, not a SQL Server image.

> [!IMPORTANT]
> Select the database when opening the connection. Don't use `USE` or the query editor database dropdown to switch from `master` for this workflow. A direct user-database connection enforces Azure SQL Database semantics; `USE` to switch databases isn't supported in that session.

## Create a table

In the query window connected directly to `TutorialDB`, run:

```sql
CREATE TABLE dbo.Customers
(
    CustomerId int NOT NULL PRIMARY KEY,
    Name nvarchar(50) NOT NULL,
    Location nvarchar(50) NOT NULL
);
GO
```

Keep using this `TutorialDB` connection for the remaining table operations.

## Insert rows

Run the following statement once to add sample data:

```sql
INSERT INTO dbo.Customers (CustomerId, Name, Location)
VALUES
    (1, N'Orlando', N'Australia'),
    (2, N'Keith', N'India'),
    (3, N'Donna', N'Germany'),
    (4, N'Janet', N'United States');
GO
```

## Query the table and view the results

Run:

```sql
SELECT CustomerId, Name, Location
FROM dbo.Customers
ORDER BY CustomerId;
```

SSMS displays four rows in the **Results** pane:

| CustomerId | Name | Location |
| --- | --- | --- |
| 1 | Orlando | Australia |
| 2 | Keith | India |
| 3 | Donna | Germany |
| 4 | Janet | United States |

The query window status bar also identifies the server and database. Verify that it shows your local endpoint and `TutorialDB`, not an Azure database used for production.

## Import your schema and data

If you already have a database package, you can use Azure SQL-compatible [SqlPackage](/sql/tools/sqlpackage/sqlpackage) operations instead of creating the sample table manually.

| Package | What it contains | Operation |
| --- | --- | --- |
| `.dacpac` | Database schema, without table data | [Publish](/sql/tools/sqlpackage/sqlpackage-publish) to deploy the schema to a development database. |
| `.bacpac` | Database schema and table data | [Import](/sql/tools/sqlpackage/sqlpackage-import) into a new or empty development database. |

Target the local server, host port, and intended user database. Review the target and proposed changes before proceeding, then connect directly to the user database to inspect the result. Don't import a BACPAC into `TutorialDB` after creating the sample table because the database isn't empty.

<!-- DRAFT: Verify SSMS data-tier wizard compatibility before documenting an SSMS-specific package import or deployment menu path. -->

SQL Server `.bak` files aren't BACPAC files. `BACKUP DATABASE` and `RESTORE DATABASE` aren't supported by the Azure SQL Database container.

## Connect your application

Use the local endpoint, the wizard's selected host port, your database name, and your credentials with an Azure SQL-compatible driver or object-relational mapper (ORM). For example, this ADO.NET-style connection string targets `TutorialDB`:

```text
Server=localhost,<host-port>;Database=TutorialDB;User ID=<sql-login>;Password=<password>;Encrypt=True;TrustServerCertificate=True;
```

Replace the placeholders before connecting. `TrustServerCertificate=True` is for the local container's self-signed certificate only. Use certificate validation and an appropriate authentication method for Azure SQL Database in Azure.

Store connection strings in environment variables or your development secret store, not source control or AI prompts. Other drivers use different connection-string syntax. See [Get started with Microsoft.Data.SqlClient](/sql/connect/ado-net/get-started-sqlclient-driver).

## Develop with AI

You can use [GitHub Copilot in SSMS](../github-copilot/get-started.md) to explain SQL, draft queries, and plan schema changes against your local database. Review generated SQL and the selected connection before executing it. The local container is free; GitHub Copilot and external model services have their own access and pricing requirements.

After creating `dbo.Customers`, try this prompt:

```text
I'm developing against an Azure SQL Database container.
Use my existing TutorialDB connection. First confirm the active database
and EngineEdition = 5. Explain the dbo.Customers schema, then draft a
parameterized query to insert a customer and a query to list customers.
Don't execute changes, delete data, or create cloud resources.
Don't request or display passwords. Don't use USE to switch databases.
```

For code-first workflows, the [Azure SQL Database container agent skills](https://microsoft.github.io/azure-sql-database-container/agent-skills.html) provide setup, connection, migration, and local-to-cloud guidance for supported AI coding agents.

## Troubleshoot connectivity and tooling

| Symptom | Action |
| --- | --- |
| Wizard isn't available | Confirm that your SSMS release includes the Azure SQL Database container wizard and that you're using Windows x64. The ARM workflow isn't supported. |
| Runtime prerequisite check fails | Start the runtime and verify that it's configured for Linux containers. For Docker, run `docker info`. |
| Image download fails | Review the wizard's error and check network access and any required image access settings. |
| Container exits or SSMS can't connect after startup | Review provisioning errors and runtime logs for the actual container name. Check the password policy and runtime resources, then retry after the engine is ready. |
| Port is already allocated | Use an available host port during provisioning and the same port in the SSMS connection. |
| Certificate validation fails | For this local self-signed certificate only, enable **Trust server certificate** and keep encryption enabled. |
| Can't open `TutorialDB` | Create the database on a `master` connection first, then open a separate connection directly to `TutorialDB`. |
| `USE` returns error 40508 | Open a new connection that names the target database instead of switching databases within a session. |
| Object Explorer or another graphical feature fails | Check the error against the [known limitations](https://microsoft.github.io/azure-sql-database-container/known-limitations.html) for your preview build. The container's bundled `sqlcmd` is an alternative if a graphical feature blocks progress; see [Getting started](https://microsoft.github.io/azure-sql-database-container/getting-started.html#step-3-connect-and-run-your-first-query). |

Test your application against Azure SQL Database in the cloud before deploying it; preview restrictions, defaults, and feature support can differ.

## Manage the container lifecycle

You can reconnect to a running container with its local endpoint, user database, and credentials. Stopping the container makes it unavailable to SSMS and your application until it's started again. Removing a container and deleting its stored data are separate operations.

<!-- DRAFT: Add verified SSMS start, stop, reconnect, and delete commands if the release provides them. Don't copy the VS Code Connections view commands. -->

For a Docker-backed container, use the following PowerShell commands to stop and start it. Enter the actual container name recorded during provisioning:

```powershell
$containerName = Read-Host 'Enter the container name shown during provisioning'
docker stop $containerName
docker start $containerName
```

Run the stop and start commands separately as needed. After restarting, wait for the engine to be ready and reconnect directly to `TutorialDB` using the same credentials.

If the wizard created a persistent volume, database files can remain after removing the container. Confirm the actual persistence configuration before deleting anything. A persistent volume isn't a backup and doesn't provide the cloud service's managed recovery.

## Clean up resources

To remove the tutorial resources:

1. Close query windows and disconnect SSMS connections to `TutorialDB`.
1. On a connection explicitly targeting `master`, run the following only if `TutorialDB` is the database you created for this tutorial:

   ```sql
   DROP DATABASE [TutorialDB];
   GO
   ```

1. Disconnect the remaining SSMS connection. For a Docker-backed container, enter its actual name, then stop and remove it:

   ```powershell
   $containerName = Read-Host 'Enter the tutorial container name'
   docker stop $containerName
   docker rm $containerName
   ```

1. If provisioning created a Docker named volume, optionally remove it after confirming its name and contents:

   > [!WARNING]
   > Removing a volume permanently deletes its database files, including any other databases you created there. Run the following command only if you no longer need that data and no other container uses the volume.

   ```powershell
   $volumeName = Read-Host 'Enter the verified tutorial data volume name'
   docker volume rm $volumeName
   ```

Removing a container doesn't automatically remove its named volume. Stopping the container or retaining the volume doesn't provide the managed backup, high availability, or recovery services of Azure SQL Database in the cloud.

## Related content

- [What is the Azure SQL Database container?](https://microsoft.github.io/azure-sql-database-container/what-is-the-container.html)
- [Azure SQL Database container known limitations](https://microsoft.github.io/azure-sql-database-container/known-limitations.html)
- [Connect and query Azure SQL Database in the cloud](ssms-connect-query-azure-sql-database.md)
- [Query Editor (SQL Server Management Studio)](../f1-help/database-engine-query-editor-sql-server-management-studio.md)
