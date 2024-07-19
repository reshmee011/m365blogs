---
title: "SharePoint Storage Monitoring using PowerShell"
date: 2024-07-19T09:53:05+01:00
tags: ["SharePoint", "Storage", "Monitoring","PowerShell"]
featured_image: '/posts/images/PowerShell-SharePoint-Storage-Reporting/Storage_Metric.png'
omit_header_text: true
draft: false
---

There is limited space allocated to the tenant. To ensure business continuity and smooth ongoing operation, it is imperative to keep an eye on its usage and take relevant actions suited to the circumstances.

## Space calculation
Total storage depends on number of licensed users
1 TB plus 10GB per license purchased

## Data storage

The data using up space are made up of
1.	Files, Meetings within SharePoint - Teams sites
2.	Others service like Loop workspace and SharePoint Embedded are part of the storage but it has not been implemented

OneDrive storage is not reflected and each user is allocated 1 TB by default.

This script written with the help of PnP PowerShell module will return the total storage usage. 

## Script to return big sites within tenant

```PowerShell
connect-pnpOnline -url https://contoso-admin.sharepoint.com/ -Interactive
$reportPath = "c:\temp\storage.csv"
Get-PnPTenantSite |  Sort-Object StorageUsageCurrent -Descending  | Select-Object Url, @{Name='StorageUsageCurrent (GB)'; Expression={$_.StorageUsageCurrent / 1024}}, @{Name='StorageQuota (GB)'; Expression={$_.StorageQuota / 1024}}, @{Name='% Used'; Expression={'{0:P2}' -f ($_.StorageUsageCurrent / $_.StorageQuota)}} | Select-Object -First 10| export-csv  $reportPath -notypeinformation
```

> Omit `Select-Object -First 10` if all sites need to be monitored. 


## Conclusion

PowerShell can help monitor storage within the tenant


