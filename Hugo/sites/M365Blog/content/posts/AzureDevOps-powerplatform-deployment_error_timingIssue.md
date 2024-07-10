---
title: "AzureDevOps Powerplatform Deployment error due to timing issue"
tags: ["Azure","DevOps","YAML","Power Platform","Import","Timing Issue"]
date: 2024-07-01T06:59:50+01:00
featured_image: '/posts/images/AzureDevOps-powerplatform-deployment_error_timingIssue/ChangeAsyncTime_to_120.PNG'
omit_header_text: true
draft: false
---

If it takes too long to deploy the solution, time out issues might occur. In this instance consider checking and updating the MaxAsyncWaitTime. Its value has been increased to 120 which means the deployment process will wait up to 120 seconds (or 2 minutes) for asynchronous operations such as import power platform solution to complete fixing the timing out issue.

[Timing out issue](../images/AzureDevOps-powerplatform-deployment_error_timingIssue/PAppsTimeout.PNG)

The fix is to amend the MaxAsyncWaitTime to 120

```yml
          - task: PowerPlatformImportSolution@2
            inputs:
              authenticationType: PowerPlatformSPN
              PowerPlatformSPN: 'powerplatform-t-connection'
              Environment: 'https://ppf-st.crm11.dynamics.com'
              SolutionInputFile: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/$(SolutionName)_Managed.zip'
              UseDeploymentSettingsFile: true
              DeploymentSettingsFile: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/settings-test.json'
              ConvertToManaged: true
              PublishCustomizationChanges: true
              MaxAsyncWaitTime: "120"
```

[Change Async Time to 120](../images/AzureDevOps-powerplatform-deployment_error_timingIssue/ChangeAsyncTime_to_120.PNG)