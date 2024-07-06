---
title: "Guide to Preparing setting files for Power Platform managed solution deployment"
date: 2024-06-15T16:54:24+01:00
tags: ["Power Platform","solution","managed solution","setting file","ALM"]
omit_header_text: true
featured_image: '/posts/images/AzureDevOps-PowerPlatform-deployment-ALM-SettingsFile/CreateDeploymentSettings.png'
draft: false
---

# Guide to Preparing setting files for Power Platform managed solution deployment

This guide will walk you through the process of creating a settings file for Power Platform managed solution deployment. Refer to the posts for more details to set up Application Lifecycle Management (ALM) for power platform: [Power Platform ALM & Pipelines w/ Matt Devaney](https://www.youtube.com/watch?v=wQe7n62RRNU) and [Converting to Modern YAML Pipeline: Application Lifecycle Management in Azure DevOps for Power Platform](https://reshmee.netlify.app/posts/powerplatform-convert-classic-pipeline-to-modern-pipeline/) for detailed steps for ALM for power platform solutions.
The settings file will be created referring to an exported solution, one of several methods described in the official documentation using **Power Platform CLI (pac)**

Make sure you have the latest version of pac installed by executing `pac install latest` first.

## Export Solution

Start by exporting the solution as managed to a local location on your machine.

```dotnetcli
pac solution export --path E:\Settings\test_managed.zip --name test --managed
```

![Export solution](../images/AzureDevOps-PowerPlatform-deployment-ALM-SettingsFile/Export_Solution.png)

## Run Power Platform CLI (pac)

Next, open a Command Prompt or PowerShell in the folder containing the solution zip file. We will use the Power Platform CLI (pac) to create a settings file based on the exported solution. 

```PowerShell
# connect to the power platform environment
pac auth create --environment "dev (contoso.co.uk)"
```

![Successful connection](../images/AzureDevOps-PowerPlatform-deployment-ALM-SettingsFile/pac_environment_success.png)

```PowerShell
# ensure the correct environment is selected
pac auth list
# select the environment by specifying the index
pac auth select --index 1
```

![pac auth select](../images/AzureDevOps-PowerPlatform-deployment-ALM-SettingsFile/pac_auth_select.png)

```PowerShell
pac solution create-settings --solution-zip ./DevToolsTest_1_0_0_1.zip --settings-file ./settings-test.json
```

![Create Deployment settings](../images/AzureDevOps-PowerPlatform-deployment-ALM-SettingsFile/CreateDeploymentSettings.png)

The settings file generated will look something like this with two main sections for **EnvironmentVariables** and **ConnectionReferences**

```json
{
  "EnvironmentVariables": [

    {
      "SchemaName": "ppf_contosoSPSitePrefix",
      "Value": "",
      "DefaultValue": "Dev - ",
      "Name": {
        "Default": "contoso - SP Site Prefix",
        "ByLcid": {
          "1033": "contoso - SP Site Prefix"
        }
      }
    },
    {
      "SchemaName": "ppf_contosoSPSiteUrl",
      "Value": "",
      "Name": {
        "Default": "contoso - SP SiteUrl",
        "ByLcid": {
          "1033": "contoso - SP SiteUrl"
        }
      }
    }
  ],
  "ConnectionReferences": [
    {
      "LogicalName": "ppf_contoso",
      "ConnectionId": "",
      "ConnectorId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline"
    },
    {
      "LogicalName": "ppf_contosoApprovals",
      "ConnectionId": "",
      "ConnectorId": "/providers/Microsoft.PowerApps/apis/shared_approvals"
    }
  ]
}
```

Each environment variable and connection reference will need to be filled out according. A separate settings file for each target environment : test, UAT and Prod needs to be prepared.

```json
{
  "EnvironmentVariables": [
    {
      "SchemaName": "contoso_SPSitePrefix",
      "Value": "Test - ",
      "DefaultValue": "Test - ",
      "Name": {
        "Default": "test - SP Site Prefix",
        "ByLcid": {
          "1033": "test - SP Site Prefix"
        }
      }
    },
    {
      "SchemaName": "contoso_SPSiteUrl",
      "Value": "https://contoso.sharepoint.com/teams/t-app-test",
      "Name": {
        "Default": "test - SP SiteUrl",
        "ByLcid": {
          "1033": "test - SP SiteUrl"
        }
      }
    }
  ],
  "ConnectionReferences": [
    {
        "LogicalName": "contoso_test",
        "ConnectionId": "0ba02acefbfc464f870b6d421f633e82",
        "ConnectorId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline"
      },
      {
        "LogicalName": "ppf_testApprovals",
        "ConnectionId": "429957a759e3460badcd197b746afb4a",
        "ConnectorId": "/providers/Microsoft.PowerApps/apis/shared_approvals"
      }
  ]
}
```

An errors in the deploymemt settings file will result in an error.

## Environment variable missing value

In case the value of an environment variable is left empty, attempt to deploy the solution using the settings file will result in 

![Environment Variable can't be empty](../images/AzureDevOps-PowerPlatform-deployment-ALM-SettingsFile/EnvironmentVariableCantBeEmpty.png)

## Wrong connection reference

In case the connection referece connectionid or connectorid is incorrect the error **An error unexpected occurred.** can be thrown.

![An unexpected error occurred](../images/AzureDevOps-PowerPlatform-deployment-ALM-SettingsFile/AnexpectedError.png)

Ensure the connection has been created and correct id populated in the settings file. Also shared the connection with the identity used for deployment of the managed solution.

Navigate to the Power Platform environment, click on connections, select the connection (for example SharePoint) and copy the Id from the Url into the connectionid property

![Connection Id](../images/AzureDevOps-PowerPlatform-deployment-ALM-SettingsFile/connectionid.png)

## Successful deployment

If the environment variables and connection reference have been populated correctly, the managed solution will be successfuly deployed.

## References

[pac admin](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/admin)
