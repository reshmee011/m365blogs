---
title: "Coauthoring in power apps issues"
date: 2023-07-09T07:29:39+01:00
tags: ["Power Apps", "coauthoring","Azure DevOps"]
draft: false
---

# Coauthoring with power apps issues

Coauthoring in power apps allows multiple power apps developers to work on the same canvas app. Despite being an experimental feature we decided to give a try to speed up the development process. We used the blog post [How To Setup Power Apps Co-Authoring - Azure Dev Ops Version] (https://www.matthewdevaney.com/how-to-setup-power-apps-co-authoring-azure-dev-ops-version/). The tutorial covers step-by-step instructions for enabling co-authoring for canvas apps in Power Apps using Azure DevOps repository. It highlights the benefits of multiple developers working together on the same app, the process of committing changes and checking for updates, and viewing the Power Apps source code in an Azure DevOps repository. 

We were wary of not doing changes on the same screen alluding to how coauthoring in applications like PowerPoint can go wrong if multiple users are editing the same slide overriding each other's work. We had some success with each one of us creating their own screen and adding some controls. However if multiple people tries to commit at the same, in the background smart merge automatically happens which could explain some of our woes noticed. 

![Smart Merge](../images/coauthoring-canvasapps/SmartMerge.png)

## Loss of implementation 
Renaming the parent control like a container resulted in all controls underneath go missing. Also while making changes to a gallery , all controls added to the gallery were gone. After a few times it happened we decided not to use co-authoring. Instead only one person will make changes at one time.

## Restore a previous version failed
Using git we attempted to restore a previous version to recover some implementation, however the canvas app did not seem to reflect the restored version and had to redo all implementation.

## Random error messages while working in coauthoring mode.
We were commiting our changes frequently at least every 15 mins and sometimes ended with being unable to resume editing or even opening the app because of server exception error. 
![Server Exception](../images/coauthoring-canvasapps/ServerException.png)

Fortunately reverting last commit helped to resolve above issue.

Sometimes clicking on the "Commit changes and check for git updates" resulted in the XMLHttpRequestError issue. Clicking again helped to resolve the issue.
![XMLHttpRequestError](../images/coauthoring-canvasapps/XMLHttpRequestError.png)

Other times the app would take a long time to load.
![Long Time to Load](../images/coauthoring-canvasapps/LongTimeToLoad.png)

We gave up in the end concluding though coauthoring would have been the utopia for speed development , it is not reliable to be used yet until there are further improvements in that area.
