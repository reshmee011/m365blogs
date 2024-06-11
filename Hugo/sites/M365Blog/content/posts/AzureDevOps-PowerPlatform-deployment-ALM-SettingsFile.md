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
pac solution create-settings --solution-zip ./DevToolsTest_1_0_0_1.zip --settings-file ./settings-test.json
In my case, the settings file looks like this

{
"EnvironmentVariables": [
{
"SchemaName": "bebe_EnvVarTest",
"Value": ""
}
],
"ConnectionReferences": [
{
"LogicalName": "bebe_sharedcommondataserviceforapps_e904a",
"ConnectionId": "",
"ConnectorId": "/providers/Microsoft.PowerApps/apis/shared_commondataserviceforapps"
}
]
}
What you can see is that there is an array of Environment Variables where you could add the desired value. The file also contains an array of Connection References, what missing here is the ID of the connection (in the target environment) you would like to associate.

For the ease of this post, I will hard code all of that. For sure you could dynamically set/fill this file as a part of your pipeline. What I found most practical is having a settings file for every target environment. Much like the app.config one knows from C# projects.

