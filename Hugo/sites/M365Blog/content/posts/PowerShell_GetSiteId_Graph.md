---
title: "Retrieving SiteId from Microsoft Graph for Subsequent API Calls"
date: 2024-06-15T14:49:19+01:00
tags: ["PnP PowerShell", "Get Site Id","Microsoft Graph"]
featured_image: '/posts/images/PowerShell_GetSiteId_Graph/example.png'
omit_header_text: true
draft: false
---

# Retrieving SiteId from Microsoft Graph for Subsequent API Calls

This post offers an option to retrieve a SiteId from Microsoft Graph using PnP PowerShell. This can be particularly useful when making further API calls that require the SiteId.

```powershell
$siteurl = "https://contoso.sharepoint.com/sites/Company311"
Connect-PnPOnline -url $siteurl -interactive

# for the site url https://contoso-admin.sharepoint.com/teams/app-m365 
# Extract the domain and site name
$uri = New-Object System.Uri($siteurl)
$domain = $uri.Host
$siteName = $uri.AbsolutePath

# Construct the new URL
$RestMethodUrl = "v1.0/sites/$($domain):$siteName?$select=id"

$site = (Invoke-PnPGraphMethod -Url $RestMethodUrl -Method Get -ConsistencyLevelEventual)
$siteId = $site.id

write-host $siteId
```

The above script uses the PnP PowerShell module to connect to a SharePoint site and retrieve its SiteId using the Microsoft Graph API.

Here's a sample output:

```powershell
contoso.sharepoint.com,18440dad-4514-4946-bcb5-ba2f7e033ed1,dbfeb5ba-2bf1-4ff8-bf61-af1d984fceb1
```

This output represents the SiteId that you can use for subsequent API calls.

```powershell
$siteId = "contoso.sharepoint.com,18440dad-4514-4946-bcb5-ba2f7e033ed1,dbfeb5ba-2bf1-4ff8-bf61-af1d984fceb1"
$RestMethodUrl = "v1.0/sites/$siteId"
$siteDetails = (Invoke-PnPGraphMethod -Url $RestMethodUrl -Method Get)
write-host $siteDetails
```

![Sample Output](../images/PowerShell_GetSiteId_Graph/example.png)

In this example, the Invoke-PnPGraphMethod cmdlet is used to make a GET request to the /sites/{site-id} endpoint of the Microsoft Graph API. The {site-id} is replaced with the actual SiteId. The response, which contains the details of the SharePoint site, is then printed to the console.

## References

[How to find {sitesId} to pass to MS Graph API call?](https://pankajsurti.com/2021/09/27/how-to-find-sitesid-to-pass-to-ms-graph-api-call/)