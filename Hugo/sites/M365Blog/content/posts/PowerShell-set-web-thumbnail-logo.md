---
title: "PowerShell: Set and Remove SharePoint Site Thumbnail Logo" 
date: 2024-02-12T21:21:02Z
tags: ["SharePoint","PnP","PowerShell","CLI for M365","M365","Logo","Thumbnail"]
featured_image: '/posts/images/PowerShell-set-web-thumbnail-logo/thumbnail_logo.png'
draft: false
---

# PowerShell: Set and Remove SharePoint Site Thumbnail Logo 

For many organizations, maintaining a consistent brand identity across SharePoint sites is crucial. The distinction between "Site Logo" and "Site Logo Thumbnail" is essential, as they serve different purposes across SharePoint and the Microsoft 365 ecosystem. The site logo appears in the site header, while the site logo thumbnail is used in search results, site cards, when copying/moving files, and other critical areas. It plays a vital role in helping end users differentiate between various SharePoint Online (SPO) sites and Teams. For more information, you can refer to [How to change a logo on a SharePoint Site](https://sharepointmaven.com/how-to-change-a-logo-on-a-sharepoint-site/).

This article covers how to help with updating site logo and thumbnail in a scalable and repeatable manner during site provisioning using PnP PowerShell , REST and CLI for Microsoft 365.

## PnP PowerShell : set by providing the absolute or relative URL to a file on the current site collection url 


```powershell
# Set thumbnail logo and site logo for a SharePoint site using PnP PowerShell
# Connect to the SharePoint site
Connect-PnPOnline -Url "https://yoursharepointsiteurl.com" -Interactive
# Set the Site Thumbnail URL
Set-PnPWebHeader -SiteThumbnailUrl "/sites/11/SiteAssets/thumbnail.jpg" -SiteLogoUrl "/sites/311/SiteAssets/logo.jpg"
```

## CLI for M365 : set by providing the absolute or relative URL to a file on the current site collection url 

```powershell
$SiteUrl = "https://yoursharepointsiteurl/"
Write-Host "Ensure logged in"
$m365Status = m365 status --output text
if ($m365Status -eq "Logged Out") {
  Write-Host "Logging in the User!"
  m365 login --authType browser
}

m365 spo site set --url $SiteUrl --siteThumbnailUrl "/sites/11/SiteAssets/thumbnail.jpg" --siteLogoUrl "/sites/311/SiteAssets/logo.jpg" 
```


## Remove Site Thumbnail URL and Site Logo URL using empty string

## PnP PowerShell

```powershell
# Set thumbnail logo and site logo for a SharePoint site using PnP PowerShell
# Connect to the SharePoint site
Connect-PnPOnline -Url "https://yoursharepointsiteurl.com" -Interactive
# Set the Site Thumbnail URL
Set-PnPWebHeader -SiteThumbnailUrl "" -SiteLogoUrl ""
```
## CLI for M365

```powershell
$SiteUrl = "https://yoursharepointsiteurl/"
Write-Host "Ensure logged in"
$m365Status = m365 status --output text
if ($m365Status -eq "Logged Out") {
  Write-Host "Logging in the User!"
  m365 login --authType browser
}

m365 spo site set --url $SiteUrl --siteThumbnailUrl "" --siteLogoUrl "" 
```

## REST API 

The endpoint siteurl/_api/siteiconmanager/setsitelogo can be used to update the thumbnail and logo and the differentiation is the aspect property passed as data. 

The thumbnail

```json
 data: {
    aspect: 0,
    relativeLogoUrl: logoUrl,
    type: 0
  }
```

The logo 

```json
 data: {
    aspect: 1,
    relativeLogoUrl: logoUrl,
    type: 0
  }
```

## References

[How to change a logo on a SharePoint Site](https://sharepointmaven.com/how-to-change-a-logo-on-a-sharepoint-site/)

[Change a site's title, description, logo, and site information settings](https://support.microsoft.com/en-gb/office/change-a-site-s-title-description-logo-and-site-information-settings-8376034d-d0c7-446e-9178-6ab51c58df42?wt.mc_id=MVP_308367)

[spo site set](https://pnp.github.io/cli-microsoft365/cmd/spo/site/site-set?wt.mc_id=MVP_308367)

[Set-PnPWebHeader](https://pnp.github.io/powershell/cmdlets/Set-PnPWebHeader.html?wt.mc_id=MVP_308367)



