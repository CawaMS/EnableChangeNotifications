# Enable Change Notifications Private Preview - Application Change Analysis

The following steps explain how to enable change notifications private preview on your Azure subscription and how to configure an Azure Monitor alert on specific changes.

## Pre-requisites
* You should have an Azure subscription. 
    * [Create your Azure free account today](https://azure.microsoft.com/en-us/free/)
* Email changeanalysisteam@microsoft.com with your subscription ID to enable notifications
* [Install Azure PowerShell Az module](https://docs.microsoft.com/en-us/powershell/azure/new-azureps-module-az?view=azps-3.8.0#upgrade-to-az)
* [Create a Log Analytics workspace in the Azure portal](https://docs.microsoft.com/en-us/azure/azure-monitor/learn/quick-create-workspace)

## Enable notifications
* Download this script //TODO
* Edit the following parameters:
    * Subscription ID
    * Workspace ID
    * Workspace resource ID
    * Location
    * ‘Include change details’ – anyone who has READ access to the workspace may potentially see sensitive information (the old/new value. i.e. connection string) for the workspace
    * Enable/Disable notifications
* Run the script

## Test the notification
* Make a change by adding a slot to your web app
* Go to the Log Analytics workspace. You should see MicrosoftChangeAnalysis_ChangeEvent_CL under Custom Logs. Run a query to see results
<TODO: screenshot>
* Go to Alerts and configure a custom log alert on change. Refer to [Create, view, and manage log alerts using Azure Monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-log)
    * Based on if configured ‘include change details’, user can either see the old/new value or click on deep link



