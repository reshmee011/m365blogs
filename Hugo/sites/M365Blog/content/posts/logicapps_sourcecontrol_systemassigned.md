---
title: "Logic Apps System Assigned Source Control "
omit_header_text: true
draft: true
date: 2024-04-01T16:49:18Z
featured_image: '/posts/images/logicapps_sourcecontrol_systemassigned/SetInReview.png'
tags: ["Logic apps", "System Assigned", "Source Control"]
---

Logic Apps provide an alternative to power automate,

Difference between Logics Apps and Power Automate 

Ownership is different 

Cost is different – Premium actions are standard actions 

Scale  

History for more than 30 days, default is 90 days and can be extended 

![Long running flow Limits and configuration reference guide - Azure Logic Apps | Microsoft Learn] (https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-limits-and-config?tabs=consumption)

However Logic Apps do not have the notion of solution to allow deployment via pipeline in Azure DevOps.

There is Logic Apps visual studio code extension to allow export of the logic apps. It allows `Generate Azure DevOps pipeline definition for local project`.


![Azure Logic Apps Extension](../images/logicapps_sourcecontrol_systemassigned/AzureLogicApps_Extension.png)

However it does not add the system assigned property which needs to be added manually and the apiversion is defaulted with no option to amend the apiversion.

```json
  "apiVersion": "2019-05-01",
            "dependsOn": [],
            "location": "uksouth",
            "name": "[parameters('test_workflow')]",
            "identity": {
                "type": "SystemAssigned"
            },
```



## References
[Logic Apps and Source Control (with PowerShell)](https://sqlkover.com/logic-apps-and-source-control-with-powershell/)