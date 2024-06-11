---
title: "Power Platform managed solution deployment using settings files"
date: 2024-06-03T16:54:24+01:00
tags: ["Power Platform","solution","managed","allow customizations","ALM"]
featured_image: '/posts/images/AzureDevOps-PowerPlatform-deployment-ALM-SettingsFile/settingsFile.png'
draft: true
---

# Power Platform managed solution deployment using settings files

The first step would be to create our Settings file. There are several ways of doing it (as described in the docs). We will do it based on a solution file.

Export Solution
We export the solution we created as part of the prerequisites unmanaged and save it somewhere on our machine.

## Run pac

Open a Command Prompt (or PowerShell) in the folder which contains the solution zip file.

The following command will create a settings file (called settings-test.json) based on your solution in the same folder. You have to replace the name of the zip file with yours.

Make sure you have the latest version of the pac installed by executing “pac install latest” first.

```PowerShell
pac auth create --environment "dev (contoso.co.uk)"
```

```PowerShell
pac auth list
```

```PowerShell
pac auth select --index 1
```


```PowerShell
pac solution create-settings --solution-zip ./DevToolsTest_1_0_0_1.zip --settings-file ./settings-test.json
```

In my case, the settings file looks like this

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



{
  "EnvironmentVariables": [
    {
      "SchemaName": "ppf_ConligoSPSitePrefix",
      "Value": "Test - ",
      "DefaultValue": "Test - ",
      "Name": {
        "Default": "Conligo - SP Site Prefix",
        "ByLcid": {
          "1033": "Conligo - SP Site Prefix"
        }
      }
    },
    {
      "SchemaName": "ppf_ConligoSPSiteUrl",
      "Value": "https://ppfonline.sharepoint.com/teams/t-app-conligo",
      "Name": {
        "Default": "Conligo - SP SiteUrl",
        "ByLcid": {
          "1033": "Conligo - SP SiteUrl"
        }
      }
    }
  ],
  "ConnectionReferences": [
    {
        "LogicalName": "ppf_Conligo",
        "ConnectionId": "0ba02acefbfc464f870b6d421f633e82",
        "ConnectorId": "/providers/Microsoft.PowerApps/apis/shared_sharepointonline"
      },
      {
        "LogicalName": "ppf_ConligoApprovals",
        "ConnectionId": "429957a759e3460badcd197b746afb4a",
        "ConnectorId": "/providers/Microsoft.PowerApps/apis/shared_approvals"
      }
  ]
}

What you can see is that there is an array of Environment Variables where you could add the desired value. The file also contains an array of Connection References, what missing here is the ID of the connection (in the target environment) you would like to associate.

For the ease of this post, I will hard code all of that. For sure you could dynamically set/fill this file as a part of your pipeline. What I found most practical is having a settings file for every target environment. Much like the app.config one knows from C# projects.

