
---
title: "Power Platform managed solution deployment with connection reference with no Allow customisations"
date: 2024-06-03T16:54:24+01:00
tags: ["Power Platform","solution","managed","allow customizations","ALM"]
featured_image: '/posts/images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/ConnectionReference_cannot_be_Updated.png'
draft: true
---

# Power Platform managed solution deployment with connection reference with no Allow customisations

This article will focus on connection references and issue you may face if the property "Allow customisations" is turned off trying to follow best practices to avoid unamanged layer.

## Solution

A solution helps with ALM. It has two forms :
- Unmanaged to use in for development purposes for customisations
- Managed to deploy into other non dev environments to avoid customisations

A solution can have different artefacts from power apps to cloud flows and supported artefacts like environment variables and connection references. 

## Allow customisations

![Allow Customisations](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/AllCustomisation.png)

It is a good practice to turn off the "Allow Customisations" on all artefacts. However I was facing issues deploying the managed solution to a different environment due to the connection references 

![ConnectionReference_cannot_be_Updated](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/ConnectionReference_cannot_be_Updated.png)

```text
The reason given was: The evaluation of the current component(name=connectionreference, id=f0cdf68a-45a6-4cc0-bd29-4c980d87c208) in the current operation (Update) failed during managed property evaluation of condition: Managed Property Name: iscustomizableanddeletable; Component Name: connectionreference; Attribute Name: iscustomizable; '
```

## Connection References

Power Automate Flow actions requiring authentication must use either a connection or a connection reference. Connections allow Power Automate to interact with other online services (SharePoint, Outlook, Dataverse, etc). 
Connections references hold a reference to a shared connection instead of connecting directly themselves. It is a good practice to always use connection references to reduce maintenance efforts. These can be changed in one place and will update all connected flows and is easily to fix broken connections. 

![Add connection reference](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/AddConnectionReference.png)

The good news is that connection references in a managed solution can be updated without creating an unmanaged layer while editing flows or power apps from managed solution will create an unamanged layer. Flows or power apps should be not edited in production and test environments to avoid introducing bugs and setting the flag "Allow customisations" off will help to achieve the aim.
 
It is fine to keep the flag “Allow Customizations” for connection references and it helps fix the issue I was having during pipeline deployment.

To view list of connection references, go to https://<url>.crm11.dynamics.com/api/data/v9.0/connectionreferences 

![List of connection references from environment](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/listofconnectionreferencesfromenvironment.png)

To edit connection references, use the default solution, please see the blog post for more details.

View [Changing Connections in Connection References on a Managed Solution](https://ashiqf.com/2023/01/31/changing-connections-in-connection-references-on-a-managed-solution/) for more details.

Attempting to update a connection reference within a **default** solution when "Allow customisations" is set to off will produce the following error message.

**There was a problem updating your connection reference. The evaluation of the current component(name=connectionreference, id=8a19a222-a93c-4d00-b98d-d730f2aaf61e) in the current operation (Update) failed during managed property evaluation of condition: Managed Property Name: iscustomizableanddeletable; Component Name: connectionreference; Attribute Name: iscustomizable;** 

![Error updating connection reference](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/Error_Updating_ConnectionReference.png)



