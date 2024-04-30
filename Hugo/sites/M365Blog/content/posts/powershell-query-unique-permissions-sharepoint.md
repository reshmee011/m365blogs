---
title: "PowerShell Script to Query Unique Permissions in SharePoint"
date: 2024-04-28T22:31:33+01:00
tags: ["SharePoint","Unique Permissions","SharingLinks","PowerShell","Sites", "Security","Copilot for M365", "Governance","CSOM"]
featured_image: '/posts/images/powershell-query-unique-permissions-sharepoint/report.png'
draft: false
---

# Query Unique Permissions in SharePoint using CSOM and PnP PowerShell

Managing permissions in SharePoint is a critical aspect of maintaining data security and compliance within organisations. However, as SharePoint environments grow in complexity, manually auditing and managing permissions becomes increasingly challenging. To address this challenge, PowerShell scripts can be leveraged to automate the auditing process, providing administrators with valuable insights into permission structures across SharePoint sites and libraries.

For Copilot for M365 implementations, ensuring there is no oversharing is a critical aspect of safeguarding sensitive information and maintaining regulatory compliance. By integrating the unique permissions audit process, administrators can preemptively address security vulnerabilities and uphold the integrity of M365 environments. Continuous monitoring and optimisation allows to harness the full potential of M365 collaboration tools while safeguarding against unauthorised access and data leaks.

## Introduction

In this blog post, we'll explore a PowerShell script designed to automate the auditing of SharePoint permissions. This script facilitates the retrieval of unique permission assignments and sharing links for sites, lists, and items within a SharePoint environment. By executing this script, administrators can generate comprehensive reports detailing permission assignments, enabling them to identify potential security risks and ensure compliance with organizational policies.

![ManageLinks](../images/powershell-query-unique-permissions-sharepoint/UniquePermissions.png)

I have not been able to determine why unique permissions are sometimes created without generating a sharing link. One of the scenerios I noticed is the sharing link is only created when "Copy Link" is clicked on.

![CopyLinks](../images/powershell-get-sharing-links-sharepoint/linkcopied.png)

## PowerShell script Overview

{{< gist reshmee011 c23123e5f1abedd1085876279ac17b7f >}}

### Script Overview

Let's break down the key components of the PowerShell script:

### Parameters

The script begins by defining parameters such as $tenantUrl, $dateTime, and $directorypath. These parameters allow users to specify the SharePoint site collection URL, generate unique file names for reports, and specify the directory path for exporting reports.

Also two additional parameters are available  $excludeLimitedAccess and $includeListsItems to change behaviour of the script as described in the comments

```powershell
    $excludeLimitedAccess = $true; # update to $false to include limited access permissions
    $includeListsItems = $true; # update to $false to exclude list items/files 
```

### Connection to SharePoint Online Site Collection

Using the Connect-PnPOnline cmdlet, the script establishes a connection to the SharePoint Online site collection. User is prompted to enter the URL of the site collection interactively.

### Retrieval of SharePoint Permissions

The script contains functions (PermissionObject, Extract-Guid, QueryUniquePermissionsByObject, and QueryUniquePermissions) responsible for querying and extracting unique permission assignments for sites, lists, and items within the SharePoint environment. These functions utilise various PnP PowerShell cmdlets to retrieve role assignments, member details, and permission levels.

### Exclusion of Certain Libraries

To streamline the auditing process, the script excludes specified libraries, such as system libraries, using the $ExcludedLibraries array. This ensures that only relevant data is included in the permission audit.

### Exporting the Report

Once the permission data is collected, the script exports it to a CSV file with a timestamped filename. The exported report includes details such as site URL, site title, object type (site, list, or item), relative URL, list title, member type, member name, member login name, parent group, and assigned roles.

### Conclusion

By automating the auditing of SharePoint permissions with PowerShell, organisations can streamline their security and compliance efforts. This script empowers administrators to gain comprehensive insights into permission structures across SharePoint sites and libraries, enabling them to proactively address security vulnerabilities and ensure adherence to regulatory requirements. As SharePoint environments continue to evolve, automation tools like PowerShell play a crucial role in simplifying administrative tasks and enhancing overall governance.
 
## Conclusion

This script provides a comprehensive way to query unique permissions in SharePoint. It can be used to audit permissions, identify security risks, and ensure compliance with organizational policies.

## References

[How to check links shared in SharePoint Online or OneDrive for Business?](https://erik365.blog/2023/03/16/how-to-check-links-shared-in-sharepoint-online-or-onedrive-for-business/#:~:text=In%20the%20SharePoint%20Online%20report,on%20your%20Microsoft%20365%20users)

[CSOM in PowerShell Query All Unique Permissions](https://reshmeeauckloo.wordpress.com/tag/query-unique-permissions-site-webs-and-lists-csom-powershell/)