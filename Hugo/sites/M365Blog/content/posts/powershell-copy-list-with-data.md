---
title: "Copy SharePoint list structure with data - ALM"
date: 2024-08-03T20:45:31+01:00
tags: ["SharePoint", "PowerShell","Copy Structure with data"]
omit_header_text: true
draft: false
---

If you have built Power Platform solutions using SharePoint as a datasource, you may want to export the list structure along with its data from the source environment (e.g., DEV) and deploy it across different environments such as Test, UAT, and PROD. Fortunately, the PnP Provisioning from PnP PowerShell provides this capability. Power Platform solutions can be deployed using pipelines after the SharePoint structure is deployed.

## Export Structure and Data from SharePoint Lists

PnP PowerShell makes this process straightforward with `Get-PnPSiteTemplate`:

```powershell
#Connect to the Source SharePoint Site
Connect-PnPOnline -url https://contoso.sharepoint.com/sites/d-LearningCatalog -Interactive
Export the Site Template:
Get-PnPSiteTemplate -Out "C:\temp\Assessment.json"
#Add Data Rows to the Site Template
Add-PnPDataRowsToSiteTemplate -Path "C:\temp\Assessment.json" -List 'Skills'
Add-PnPDataRowsToSiteTemplate -Path "C:\temp\Assessment.json" -List 'Roles' 
Add-PnPDataRowsToSiteTemplate -Path "C:\temp\Assessment.json" -List 'Learning Catalog' 
Add-PnPDataRowsToSiteTemplate -Path "C:\temp\Assessment.json" -List 'Skills Score Targets' 
```

## Import Structure and Data from SharePoint Lists

With PnP PowerShell, the exported structure can be applied using `Invoke-PnpSiteTemplate`

```powershell
Disconnect-PnPOnline
#Connect to the Target SharePoint Site
Connect-PnPOnline -url https://contoso.sharepoint.com/sites/u-LearningCatalog -Interactive
#Invoke the Site Template to Import Structure and Data:
Invoke-PnpSiteTemplate -Path "C:\temp\Assessment.json"
```