---
title: "MySQL on-premises to Azure Database for MySQL migration guide Post Migration Management"
description: "Once the migration has been successfully completed, the next phase it to manage the new cloud-based data workload resources."
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags 
ms.author: arthiaga
ms.reviewer: maghan
ms.custom:
ms.date: 05/26/2021
---

# MySQL on-premises to Azure Database for MySQL migration guide Post Migration Management

### Monitoring and Alerts

Once the migration has been successfully completed, the next phase it to manage the new cloud-based data workload resources. Management operations include both control plane and data plane activities. Control plane activities are those related to the Azure resources versus data plane which is **inside** the Azure resource (in this case MySQL).

Azure Database for MySQL provides for the ability to monitor both of these types of operational activities using Azure-based tools such as [Azure Monitor,](/azure/azure-monitor/overview) [Log Analytics](/azure/azure-monitor/platform/design-logs-deployment) and [Azure Sentinel.](/azure/sentinel/overview) In addition to the Azure-based tools, security information and event management (SIEM) systems can be configured to consume these logs as well.

Whichever tool is used to monitor the new cloud-based workloads, alerts will need to be created to warn Azure and database administrators of any suspicious activity. If a particular alert event has a well-defined remediation path, alerts can fire automated [Azure run books](/azure/automation/automation-quickstart-create-runbook) to address the event.

The first step to creating a fully monitored environment is to enable MySQL log data to flow into Azure Monitor. Reference [Configure and access audit logs for Azure Database for MySQL in the Azure portal](/azure/mysql/howto-configure-audit-logs-portal) for more information.

Once log data is flowing, use the [Kusto Query Language (KQL)](/azure/data-explorer/kusto/query/) query language to query the various log information. Administrators unfamiliar with KQL can find a SQL to KQL cheat sheet [here](/azure/data-explorer/kusto/query/sqlcheatsheet) or the [Get started with log queries in Azure Monitor](/azure/azure-monitor/log-query/get-started-queries) page.

For example, to get the memory usage of the Azure Database for MySQL:

```
AzureMetrics
| where TimeGenerated \> ago(15m)
| limit 10
| where ResourceProvider == "MICROSOFT.DBFORMYSQL"
| where MetricName == "memory\_percent"
| project TimeGenerated, Total, Maximum, Minimum, TimeGrain, UnitName 
| top 1 by TimeGenerated
```
To get the CPU usage:

```
AzureMetrics
| where TimeGenerated \> ago(15m)
| limit 10
| where ResourceProvider == "MICROSOFT.DBFORMYSQL"
| where MetricName == "cpu\_percent"
| project TimeGenerated, Total, Maximum, Minimum, TimeGrain, UnitName 
| top 1 by TimeGenerated
```
Once you have created the KQL query, you will then create [log alerts](/azure/azure-monitor/platform/alerts-unified-log) based of these queries.

### Server Parameters

As part of the migration, it is likely the on-premises [server parameters](/azure/mysql/concepts-server-parameters) were modified to support a fast egress. Also, modifications were made to the Azure Database for MySQL parameters to support a fast ingress. The Azure server parameters should be set back to their original on-premises workload optimized values after the migration.

However, be sure to review and make server parameters changes that are appropriate for the workload and the environment. Some values that were great for an on-premises environment, may not be optimal for a cloud-based environment. Additionally, when planning to migrate the current on-premises parameters to Azure, verify that they can in fact be set.

Some parameters are not allowed to be modified in Azure Database for MySQL.

### PowerShell Module

The Azure Portal and Windows PowerShell can be used for managing the Azure Database for MySQL. To get started with PowerShell, install the Azure PowerShell cmdlets for MySQL with the following PowerShell command:

`Install-Module -Name Az.MySql`

After the modules are installed, reference tutorials like the following to learn ways you can take advantage of scripting your management activities:

  - [Tutorial: Design an Azure Database for MySQL using PowerShell ](/azure/mysql/tutorial-design-database-using-powershell)

  - [How to back up and restore an Azure Database for MySQL server using PowerShell ](/azure/mysql/howto-restore-server-powershell)

  - [Configure server parameters in Azure Database for MySQL using PowerShell ](/azure/mysql/howto-configure-server-parameters-using-powershell)

  - [Auto grow storage in Azure Database for MySQL server using PowerShell ](/azure/mysql/howto-auto-grow-storage-powershell)

  - [How to create and manage read replicas in Azure Database for MySQL using PowerShell ](/azure/mysql/howto-read-replicas-powershell)

  - [Restart Azure Database for MySQL server using PowerShell ](/azure/mysql/howto-restart-server-powershell)

### Azure Database for MySQL Upgrade Process

Since Azure Database for MySQL is a PaaS offering, administrators are not responsible for the management of the updates on the operating system or the MySQL software. However, it is important to be aware the upgrade process can be random and when being deployed, will stop the MySQL server workloads. Plan for these downtimes by rerouting the workloads to a read replica in the event the particular instance goes into maintenance mode.

> [!NOTE]
> This style of failover architecture may require changes to the applications data layer to support this type of failover scenario. If the read replica is maintained as a read replica and is not promoted, the application will only be able to read data and it may fail when any operation attempts to write information to the database.

The [Planned maintenance notification](/azure/mysql/concepts-monitoring#planned-maintenance-notification) feature will inform resource owners up to 72 hours in advance of installation of an update or critical security patch. Database administrators may need to notify application users of planned and unplanned maintenance.

> [!NOTE]
> Azure Database for MySQL maintenance notifications are incredibly important. The database maintenance can take your database and connected applications down for a period of time.

### WWI Scenario

WWI decided to utilize the Azure Activity logs and enable MySQL logging to flow to a [Log Analytics workspace.](/azure/azure-monitor/platform/design-logs-deployment) This workspace is configured to be a part of [Azure Sentinel](/azure/sentinel/) such that any [Threat Analytics](/azure/mysql/concepts-data-access-and-security-threat-protection) events would be surfaced, and incidents created.

The MySQL DBAs installed the Azure Database for [MySQL Azure PowerShell cmdlets](/azure/mysql/quickstart-create-mysql-server-database-using-azure-powershell) to make managing the MySQL Server automated versus having to log to the Azure Portal each time.

### Management Checklist

  - Create resource alerts for common things like CPU and Memory.

  - Ensure the server parameters are configured for the target data workload after migration.

  - Script common administrative tasks.

  - Set up notifications for maintenance events such as upgrades and patches. Notify users as necessary.  


> [!div class="nextstepaction"]  
> [Optimization](./optimization.md)