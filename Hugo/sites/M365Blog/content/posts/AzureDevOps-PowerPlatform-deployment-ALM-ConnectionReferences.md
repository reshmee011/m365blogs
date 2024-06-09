---
title: "Power Platform Managed Solution Deployment with Connection References - Allow customisations"
date: 2024-06-07T16:54:24+01:00
tags: ["Power Platform","Solution","Managed Solution","Customizations","ALM"]
featured_image: '/posts/images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/ConnectionReference_cannot_be_Updated.png'
draft: false
---

# Power Platform Managed Solution Deployment with Connection References - Allow customisations

This article explores the challenges and solutions associated with Power Platform managed solution deployment, specifically focusing on connection references and the "Allow Customisations" property.

Refer to the posts for more details to set up Application Lifecycle Management (ALM) for power platform: [Power Platform ALM & Pipelines w/ Matt Devaney](https://www.youtube.com/watch?v=wQe7n62RRNU) and [Converting to Modern YAML Pipeline: Application Lifecycle Management in Azure DevOps for Power Platform](https://reshmee.netlify.app/posts/powerplatform-convert-classic-pipeline-to-modern-pipeline/)

## Understanding Solutions in Power Platform

A solution in Power Platform is a container that assists with Application Lifecycle Management (ALM). It comes in two forms:
- **Unmanaged**: Used for development purposes and customisations.
- **Managed**: Deployed into non-development environments to prevent customisations.

A solution can include various artefacts, from power apps to cloud flows, and supported artefacts like environment variables and connection references.

It's advisable not to make changes directly in non development development due to the risks of introducing bugs or security loopholes. Always implement changes in a development environment using an unmanaged solution before deploying it as managed solution to other environments.

![Changes can create unmanaged layer](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/AllowCustomisations_objects.png)

## The Role of "Allow Customisations"

![Allow Customisations](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/AllCustomisation.png)

Turning off the "Allow Customisations" property on all artefacts is a recommended practice. However, this can lead to issues when deploying the managed solution to a different environment due to connection references.

## Connection References Explained

Power Automate Flow actions that require authentication must use either a connection or a connection reference. Connections enable Power Automate to interact with other online services (SharePoint, Outlook, Dataverse, etc). Connection references, on the other hand, hold a reference to a shared connection, reducing maintenance efforts as changes can be made in one place, updating all connected flows.

![Add connection reference](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/AddConnectionReference.png)

The advantage of connection references in a managed solution is that they can be updated without creating an unmanaged layer. This is unlike editing flows or power apps from a managed solution, which creates an unmanaged layer. To avoid introducing bugs, flows or power apps should not be edited in production and test environments. Setting the "Allow Customisations" flag to off can help achieve this.

Keeping the "Allow Customizations"  flag for connection references can help resolve issues encountered during pipeline deployment.

![ConnectionReference_cannot_be_Updated](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/ConnectionReference_cannot_be_Updated.png)

**The reason given was: The evaluation of the current component(name=connectionreference, id=f0cdf68a-45a6-4cc0-bd29-4c980d87c208) in the current operation (Update) failed during managed property evaluation of condition: Managed Property Name: iscustomizableanddeletable; Component Name: connectionreference; Attribute Name: iscustomizable; '**

To view a list of connection references to find out which connection reference it is complaining about, visit https://<url>.crm11.dynamics.com/api/data/v9.0/connectionreferences and note down the connection reference name to check whether the "Allow customizations" flag is set to off and is the cause of the issue during deployment.

![List of connection references from environment](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/listofconnectionreferencesfromenvironment.png)

For more details on editing connection references using the default solution, refer to the blog post [Changing Connections in Connection References on a Managed Solution](https://ashiqf.com/2023/01/31/changing-connections-in-connection-references-on-a-managed-solution/).

![Save connection references](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/SaveChangesToConnectionReferences.png)

Note: Attempting to update a connection reference within a default solution when "Allow customisations" is set to off will produce an error message.

![Error updating connection reference](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/Error_Updating_ConnectionReference.png)

## Wrapping Up

In conclusion, the management of connection references within Power Platform managed solutions can be a complex task, but with the right understanding and approach, it can be effectively handled. Remember, while it's recommended to turn off the "Allow Customisations" property for most artefacts, keeping it on for connection references can help avoid deployment issues. 

Always be mindful of the implications of your settings and strive to maintain a clean, manageable environment. This will not only make your deployment process smoother but also enhance the maintainability of your solutions in the long run.
