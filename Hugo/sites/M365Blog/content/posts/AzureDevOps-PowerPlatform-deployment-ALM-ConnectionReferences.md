
---
title: "Power Platform managed solution deployment with connection reference with no Allow customisations"
date: 2024-06-03T16:54:24+01:00
tags: ["Power Platform","solution","managed","allow customizations","ALM"]
featured_image: '/posts/images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/ConnectionReference_cannot_be_Updated.png'
draft: true
---

# Power Platform managed solution deployment with connection reference with no Allow customisations

A solution helps with ALM. It has two forms :
- Unmanaged to use in for development purposes for customisations
- Managed to deploy into other non dev environments to avoid customisations

## Solution
A solution can have different artefacts from power apps to cloud flows and supported artefacts like environment variables and connection references. This article will focus on connection references and issue you may face if the property "Allow customisations" is turned off trying to follow best practices to avoid unamanged layer.

## Connection References
Power Automate Flow actions requiring authentication must use either a connection or a connection reference. Connections allow Power Automate to interact with other online services (SharePoint, Outlook, Dataverse, etc). 
Connections references hold a reference to a shared connection instead of connecting directly themselves. Always use connection references to reduce maintenance efforts. Connection references can change in one place and will update all connected flows and is easily to fix broken connections. 
Also connection references in a managed solution can be updated without creating an unmanaged layer. Never directly edit flows in production and test environments to avoid introducing bugs.
 

It is fine to keep the flag “Allow Customizations” for connection references 
 
![Add connection reference](../images/AzureDevOps-PowerPlatform-deployment-ALM-ConnectionReferences/AddConnectionReference.png)



The reason given was: The evaluation of the current component(name=connectionreference, id=f0cdf68a-45a6-4cc0-bd29-4c980d87c208) in the current operation (Update) failed during managed property evaluation of condition: Managed Property Name: iscustomizableanddeletable; Component Name: connectionreference; Attribute Name: iscustomizable; '