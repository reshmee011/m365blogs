---
title: 'Create default groups using CSOM and PowerShell after creating site collections via script'
date: Mon, 03 Oct 2016 16:13:28 +0000
draft: false
tags: ['CSOM', 'Default Groups Visitors Members and Owner', 'PowerShell', 'Security', 'SharePoint 2013', 'SharePoint Online']
---

Whenever a site collection is created using PowerShell, the default security groups (visitors, owners and members) are not provisioned. When site collection is created via browser an additional method call is made to create the security groups which is not called while using PowerShell.

The following scripts will help create the default groups.

https://gist.github.com/reshmee011/7c4ac90cb3038d16fa1448f4c5678896 Call method CreateWebDefaultGroup  passing the parameters context(clientcontext) and  UniquePermissions(bool) `CreateWebDefaultGroup -Context $ctx -UniquePermissions $false`