---
title: "Sharepoint Restricted Search"
date: 2024-05-28T09:55:41+01:00
draft: true
---

### Remove certain SharePoint sites from search

Excluding certain SharePoint sites from search would mean the contents from the excluded sites won't be available to M365 search and Copilot and should be used as a last resort. 

Until the right controls are in place, sensitive sites may be excluded using [PowerShell Scripts for Restricted SharePoint Search](https://learn.microsoft.com/en-us/sharepoint/restricted-sharepoint-search-admin-scripts?wt.mc_id=MVP_308367).

1. Enable Restricted Search Mode

```PowerShell
Set-SPOTenantRestrictedSearchMode -Mode Enabled 
```

2. Add the sites to the allowed list via csv file or in string

```Powershell
Add-SPOTenantRestrictedSearchAllowedList  -SitesListFileUrl C:\Users\admin\Downloads\UrlList.csv
```

3. Retrieves existing list of URLs in the allowed list

```powershell
 Get-SPOTenantRestrictedSearchAllowedList
```

4. Removes sites from the allow list

```powershell
Remove-SPOTenantRestrictedSearchAllowedList -SitesList <List[string]> [<CommonParameters>]
