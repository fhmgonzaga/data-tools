---
title: Connect and Query an Azure SQL Database Container
description: Create a local Azure SQL Database container, connect with SQL Server Management Studio (SSMS), and run your first T-SQL queries.
author: mbarickman
ms.author: mbarickman
ms.reviewer: mahyon
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

<!-- DRAFT: Public Preview is planned for November 17, 2026. Confirm release status, public image URI and tag, licensing, SSMS minimum version, and wizard behavior before publication. Current sources describe Private Preview. Confirm article ownership with the SSMS documentation team. -->

Use SQL Server Management Studio (SSMS) to connect to the Azure SQL Database engine running locally in a container and run Transact-SQL (T-SQL) commands. The container is free for local development and testing and doesn't require an Azure subscription. It isn't the SQL Server container and isn't intended for production workloads.

For an overview, see [What is the Azure SQL Database container?](https://microsoft.github.io/azure-sql-database-container/what-is-the-container.html).

> [!IMPORTANT]
> This article is a draft for the planned public preview. The current product documentation describes private preview access and incomplete SSMS graphical tooling compatibility. The manual setup below uses the currently documented preview image. The SSMS provisioning wizard instructions and screenshots are placeholders until the release build is available for verification.

This quickstart demonstrates the following steps:

> [!div class="checklist"]
>
> - Start a local Azure SQL Database container
> - Connect to the container in SSMS
> - Create a database and connect directly to it
> - Create a table, insert rows, and query the results
> - Verify the database and engine identity
> - Clean up the local resources

## Prerequisites

- [Install SQL Server Management Studio](../install/install.md) on a supported Windows computer. See [SSMS system requirements](../system-requirements.md). The minimum SSMS version for the container provisioning wizard is pending confirmation.
- Install and start a container runtime that runs Linux containers. This quickstart uses Docker Desktop with the Windows Subsystem for Linux 2 (WSL 2) backend on Windows. See [Docker Desktop installation on Windows](https://docs.docker.com/desktop/setup/install/windows-install/) and [container prerequisites](https://microsoft.github.io/azure-sql-database-container/prerequisites.html).
- Allocate at least two CPU cores, 4 GB of memory, and 10 GB of free disk space to the container runtime.
- Make local TCP port `1433` available, or choose a different host port.
- Have internet access to pull the container image initially. After the image is downloaded, the local SQL authentication workflow doesn't require an Azure subscription or an internet connection.
- For the currently documented private preview image, obtain registry access through the [private preview signup](https://aka.ms/sqldbcontainerpreview-signup). Don't share registry credentials.

The image provides native `linux/amd64` and `linux/arm64` variants. Docker selects the matching architecture automatically. Container runtime support on macOS or Linux doesn't mean that SSMS runs on those operating systems. This quickstart runs both SSMS and the container runtime on Windows.

## Provision a container with the SSMS wizard (draft placeholder)

> [!NOTE]
> The wizard entry point, page names, settings, and completion behavior haven't been verified in an SSMS release build. This section isn't an executable procedure yet. Don't substitute the VS Code MSSQL extension's container workflow. Use the manual procedure in the next section for the currently documented setup.

Before publication, replace this section with verified SSMS instructions that explain how to:

- Open the Azure SQL Database container provisioning wizard.
- Check runtime prerequisites and select the Azure SQL Database image.
- Configure the container name, host port, credentials, and data persistence.
- Confirm whether the wizard creates a user database or requires a separate provisioning step.
- Review the configuration, start the container, and handle startup failures.
- Confirm whether the wizard opens a connection automatically and which database that connection targets.

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

## Start a container manually

Until the wizard procedure is verified, use the following PowerShell commands to provision the container.

1. Open PowerShell and confirm that Docker is running:

   ```powershell
   docker info
   ```

   If the command fails, start Docker Desktop and make sure it's configured for Linux containers before continuing.

1. For the current private preview, sign in to the registry. Replace `<registry-username>` with the username supplied through the preview signup. Enter the registry password at the prompt, not in the command:

   ```powershell
   docker login sqldbpreview-dpgaeqhmgphzd4bk.azurecr.io --username <registry-username>
   ```

   <!-- DRAFT: Replace private preview signup, registry sign-in, and image references with the confirmed public preview distribution instructions before publication. Don't invent an MCR path. -->

1. Set the image and pull it:

   ```powershell
   $containerImage = 'sqldbpreview-dpgaeqhmgphzd4bk.azurecr.io/azure-sql/db-dev:latest'
   docker pull $containerImage
   ```

   Don't continue if the pull fails. Check your registry access and the image reference against the [current getting-started instructions](https://microsoft.github.io/azure-sql-database-container/getting-started.html).

1. In the same PowerShell window, choose a unique password for the local `sa` login:

   ```powershell
   $credential = Get-Credential -UserName 'sa' -Message 'Choose a strong password for the local container'
   $env:MSSQL_SA_PASSWORD = $credential.GetNetworkCredential().Password
   ```

   The password must contain at least eight characters and include characters from at least three of these categories: uppercase letters, lowercase letters, digits, and symbols. The database password is separate from the registry password. Don't save either password in source control.

1. Create the container with a named volume for database files:

   ```powershell
   docker run --detach --name ssms-sqldb `
     --env ACCEPT_EULA=Y `
     --env MSSQL_SA_PASSWORD `
     --publish 127.0.0.1:1433:1433 `
     --volume ssms-sqldb-data:/var/opt/mssql `
     $containerImage
   ```

   `ACCEPT_EULA=Y` accepts the image's license terms. Review those terms before running the command. The port mapping limits access to the local computer. The named volume keeps database files when you remove the container; it isn't a backup.

   If port `1433` is already in use, change the mapping to `127.0.0.1:1434:1433` and use `localhost,1434` in SSMS. If a container named `ssms-sqldb` already exists, choose a different name and use that name in the remaining commands. Don't remove an existing container without checking its data.

1. Check the container status and startup logs:

   ```powershell
   docker ps --all --filter "name=ssms-sqldb"
   docker logs ssms-sqldb
   ```

   A running container doesn't necessarily mean that the database engine is ready. Allow time for startup, then connect in the next section. If the connection fails, check the logs for password, memory, or startup errors before retrying.

1. Remove the password from the PowerShell environment when you're finished creating the container:

   ```powershell
   Remove-Item Env:\MSSQL_SA_PASSWORD
   Remove-Variable credential
   ```

   Keep the password available in a secure location for the SSMS connection. Docker retains the environment setting in the container configuration, so protect access to the Docker runtime.

## Connect to the container in SSMS

1. Start SSMS. Open a Database Engine connection through **Object Explorer** > **Connect** > **Database Engine**.

1. Enter the following connection settings. The layout depends on your SSMS version.

   | Setting | Value |
   | --- | --- |
   | Server type, if shown | **Database Engine** |
   | Server name | `localhost,1433`, or the host port you selected |
   | Authentication | **SQL Server Authentication** |
   | Login | `sa` |
   | Password | The local database password you chose |
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

The current manual workflow doesn't create a user database automatically. On the `master` connection, run:

```sql
CREATE DATABASE [TutorialDB];
GO
```

Wait for the command to complete before connecting to `TutorialDB`. If a database with that name already exists, use a different name throughout this quickstart or confirm that the existing database is safe to use. Don't delete an existing database just to repeat the tutorial.

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

## Troubleshoot connectivity and tooling

| Symptom | Action |
| --- | --- |
| Docker commands fail | Start the runtime and verify that it's configured for Linux containers. Run `docker info`. |
| Image pull fails | Check the registry sign-in and current preview image instructions. |
| Container exits or SSMS can't connect after startup | Run `docker logs ssms-sqldb`. Check the password policy and runtime resources, then retry after the engine is ready. |
| Port is already allocated | Choose another host port when creating the container and use that port in the SSMS server name. |
| Certificate validation fails | For this local self-signed certificate only, enable **Trust server certificate** and keep encryption enabled. |
| Can't open `TutorialDB` | Create the database on a `master` connection first, then open a separate connection directly to `TutorialDB`. |
| `USE` returns error 40508 | Open a new connection that names the target database instead of switching databases within a session. |
| Object Explorer or another graphical feature fails | Consult the [known limitations](https://microsoft.github.io/azure-sql-database-container/known-limitations.html). Current documentation warns that SSMS graphical tooling isn't fully compatible. Use the container's bundled `sqlcmd` as documented in [Getting started](https://microsoft.github.io/azure-sql-database-container/getting-started.html#step-3-connect-and-run-your-first-query) if the UI blocks progress. |

The container follows the Azure SQL Database feature surface, not the full SQL Server feature surface. SQL Server `BACKUP DATABASE` and `RESTORE DATABASE` aren't supported. Test your application against Azure SQL Database in the cloud before deploying it; current preview limitations can affect parity.

## Clean up resources

If you want to keep the tutorial data for another session, disconnect SSMS and stop the container:

```powershell
docker stop ssms-sqldb
```

Start it again with `docker start ssms-sqldb` and reconnect directly to `TutorialDB` using the same credentials.

To remove the tutorial resources instead:

1. Close query windows and disconnect SSMS connections to `TutorialDB`.
1. On a connection explicitly targeting `master`, run the following only if `TutorialDB` is the database you created for this tutorial:

   ```sql
   DROP DATABASE [TutorialDB];
   GO
   ```

1. Disconnect the remaining SSMS connection, then stop and remove the tutorial container:

   ```powershell
   docker stop ssms-sqldb
   docker rm ssms-sqldb
   ```

1. Optionally remove the named volume:

   > [!WARNING]
   > The following command permanently deletes all database files in `ssms-sqldb-data`, including any other databases you created there. Run it only if you no longer need that data and no other container uses the volume.

   ```powershell
   docker volume rm ssms-sqldb-data
   ```

Removing a container doesn't automatically remove its named volume. Stopping the container or retaining the volume doesn't provide the managed backup, high availability, or recovery services of Azure SQL Database in the cloud.

## Related content

- [What is the Azure SQL Database container?](https://microsoft.github.io/azure-sql-database-container/what-is-the-container.html)
- [Azure SQL Database container known limitations](https://microsoft.github.io/azure-sql-database-container/known-limitations.html)
- [Connect and query Azure SQL Database in the cloud](ssms-connect-query-azure-sql-database.md)
- [Query Editor (SQL Server Management Studio)](../f1-help/database-engine-query-editor-sql-server-management-studio.md)
