---
title: "Optimizing PowerShell Scripts to check for unique permissions in SharePoint: REST API vs. Get-PnPListItem"
date: 2024-08-04T08:42:13+01:00
tags: ["SharePoint", "PowerShell","Unique Permissions","PnP PowerShell","REST","HasUniqueRoleAssignments"]
featured_image: '/posts/images/powershell-Get-HasUniqueRoleAssignments/output.png'
omit_header_text: true
draft: false
---

When working with large SharePoint sites, checking for unique permissions can be a time-consuming task. This blog post explores methods to optimize PowerShell scripts for fetching property `HasUniqueRoleAssignments` to determine unique permissions, including using PnP PowerShell and the SharePoint REST API. We compare their performance and highlight the advantages and limitations of each approach.

## Using PnP PowerShell

PnP PowerShell provides an efficient way to interact with SharePoint Online and retrieve list items to check for unique permissions. Here's a script that demonstrates how to use PnP PowerShell to achieve this:

{{< gist reshmee011 f3d67227ea8c94db4af7836dc663cd05 >}}

For a site with 1600 items, this script takes approximately 200 seconds. For larger sites with over 100,000 items, it takes over 2 hours, which is not optimal.

![output of PnP](../images/powershell-Get-HasUniqueRoleAssignments/Output_PnP.png)

### Using parameter Fields `HasUniqueRoleAssignments` with Get-PnPListItem

Specifying the Fields parameter to return values field `HasUniqueRoleAssignments` does not load the values of properties `HasUniqueRoleAssignments` without specifying the Id parameter.

 ```powerShell
 $ListItems = Get-PnPListItem -List $list -PageSize 2000 -Fields "HasUniqueRoleAssignments"
 ```

### Attempts to Optimize with CAML Query

Using CAML query to specify fields did not significantly improve performance and encountered list view threshold errors for lists with more than 5000 items.

```powershell
$ListItems = Get-PnPListItem -List $list -Query "<View><ViewFields><FieldRef Name='HasUniqueRoleAssignments'/><FieldRef Name='FileRef'/><FieldRef Name='FileSystemObjectType'/><FieldRef Name='FileLeafRef'/></ViewFields><Query></Query></View>"
```

### Using REST API

An alternative approach involves using the SharePoint REST API to query the `HasUniqueRoleAssignments` property. 

#### REST API Endpoints

The following endpoints can be used to retrieve the `HasUniqueRoleAssignments` property:

```md
/_api/web/HasUniqueRoleAssignments  
/_api/web/lists/getbytitle('list title')/HasUniqueRoleAssignments  
/_api/web/lists/getbytitle('list title')/items(id)/HasUniqueRoleAssignments  
```

The query from the browser returns the `HasUniqueRoleAssignments` value.

```md
https://reshmeeauckloo.sharepoint.com/sites/Company311/_api/web/lists/getbytitle('Documents')/items(5)?$Select=ID,HasUniqueRoleAssignments
```

![HasUniqueRoleAssignments value returned from REST](../images/powershell-Get-HasUniqueRoleAssignments/REST_API.png)

#### Example REST API Script

Here's a PowerShell script leveraging the REST API to retrieve unique permissions:

{{< gist reshmee011 f2a0ea5d58b01d5b14571104d6f30800 >}}

For a site with 1600 items, this script takes only 8 seconds. 

![Output REST](../images/powershell-Get-HasUniqueRoleAssignments/Output_REST.png)

For larger sites with over 100,000 items, it completes in under 11 minutes, significantly faster than the PnP PowerShell approach with retry mechanism under the hood.

![REST Unique Permissions with retired](../images/powershell-Get-HasUniqueRoleAssignments/RESTAPI_UniquePermissions.png)

![Graph does not have property HasUniqueRoleAssignments](../images/powershell-Get-HasUniqueRoleAssignments/MSGraph_HasUniqueRoleAssignments_Does_NotExist.png)

## Microsoft Graph does not return the property HasUniqueRoleAssignments

Microsoft Graph does not return the HasUniqueRoleAssignments property, limiting its utility for this specific task.

![Graph does not have property HasUniqueRoleAssignments](../images/powershell-Get-HasUniqueRoleAssignments/MSGraph_HasUniqueRoleAssignments_Does_NotExist.png)


## Performance comparison

|**Endpoint Type**|**Number of items**|**Timing**|
|---|---|---|
|REST API|1600|8 secs|
|PnP PowerShell without any batch|1600|200 secs|
|REST API|100,000|11 mins|
|PnP PowerShell without any batch|100,000|2 hours|
|Microsoft Graph|`N/A`|`N/A`|

## Conclusion

Using the SharePoint REST API to check for unique permissions is significantly faster and more efficient than using PnP PowerShell, especially for large sites. This approach can save considerable time.

## References

[Invoke-PnPSPRestMethod](https://pnp.github.io/powershell/cmdlets/Invoke-PnPSPRestMethod.html)

[SharePoint REST API to retrieve ACLs of a SharePoint object](https://learn.microsoft.com/en-us/answers/questions/208656/sharepoint-rest-api-to-retrieve-acls-of-a-sharepoi)

[Get list of SharePoint files with unique permissions](https://community.powerplatform.com/forums/thread/details/?threadid=ee28b49a-4c5e-4882-b370-32ff84795ffd)