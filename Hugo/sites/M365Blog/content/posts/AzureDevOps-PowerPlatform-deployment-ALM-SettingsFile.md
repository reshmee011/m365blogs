
---
title: "Power Platform managed solution deployment with connection reference with no Allow customisations"
date: 2024-06-03T16:54:24+01:00
tags: ["Power Platform","solution","managed","allow customizations","ALM"]
featured_image: '/posts/images/AzureDevOps-PowerPlatform-deployment-ALM-SettingsFile/settingsFile.png'
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


