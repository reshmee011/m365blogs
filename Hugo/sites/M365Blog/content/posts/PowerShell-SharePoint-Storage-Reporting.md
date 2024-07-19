---
title: "SharePoint Storage Monitoring"
date: 2024-07-19T09:53:05+01:00
tags: ["SharePoint", "Storage", "Monitoring"]
featured_image: '/posts/images/PowerShell-SharePoint-Storage-Reporting/Storage_Metric.png'
omit_header_text: true
draft: false
---

Objectives
1.	Make the most of storage included in license
2.	Keep business running smoothly
3.	Prevent unwanted costs
Storage Calculation
Total storage depends on number of licensed users
1 TB plus 10GB per license purchased
We purchased 2TB for one 1 year till March 2025 to increase capacity from 8 TB to 10 TB.

What data is stored
1.	Files, Meetings within SharePoint - Teams sites
2.	Others service like Loop workspace and SharePoint Embedded are part of the storage but it has not been implemented at PPF.

OneDrive storage is not reflected


## 10 Biggest sites within tenant

```PowerShell
connect-pnpOnline -url https://contoso-admin.sharepoint.com/ -Interactive
$reportPath = "c:\temp\storage.csv"
Get-PnPTenantSite |  Sort-Object StorageUsageCurrent -Descending  | Select-Object Url, @{Name='StorageUsageCurrent (GB)'; Expression={$_.StorageUsageCurrent / 1024}}, @{Name='StorageQuota (GB)'; Expression={$_.StorageQuota / 1024}}, @{Name='% Used'; Expression={'{0:P2}' -f ($_.StorageUsageCurrent / $_.StorageQuota)}} | Select-Object -First 10| export-csv  $reportPath -notypeinformation
```

> Omit `Select-Object -First 10` if all sites need to be monitored. 


## Conclusion



