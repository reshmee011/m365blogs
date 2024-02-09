---
title: "AzureDevOps Powerplatform Managedidentity"
date: 2024-02-08T16:46:04Z
draft: true
---

# Using managed identify in Azure Devops for Power Platform deployment

I was [Unable to find the powerplatform in Azure devops](https://stackoverflow.com/questions/73568367/unable-to-find-the-powerplatform-in-azure-devops)
and the solution is to install the [Extension: Power Platform Build Tools (2.0.5)](https://marketplace.visualstudio.com/items?itemName=microsoft-IsvExpTools.PowerPlatform-BuildTools&wt.mc_id=MVP_308367) to your Organization.

Then you will get the Power Platform service connection.


## References

[Unable to find the powerplatform in Azure devops](https://stackoverflow.com/questions/73568367/unable-to-find-the-powerplatform-in-azure-devops)
[Manage service connections](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml&wt.mc_id=MVP_308367)
[Managed identity and service principal support for Azure DevOps now in General Availability (GA)](https://devblogs.microsoft.com/devops/managed-identity-and-service-principal-support-for-azure-devops-now-in-general-availability-ga/)