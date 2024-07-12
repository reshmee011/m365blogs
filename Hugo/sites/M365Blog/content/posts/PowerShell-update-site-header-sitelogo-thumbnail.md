---
title: "PowerShell Update Site Header Sitelogo Thumbnail"
date: 2024-02-11T06:39:48Z
tags: ["PowerShell","Site Logo","Thumbnail","SharePoint"]
featured_image: '/posts/images/PowerShell-update-site-header-sitelogo-thumbnail/example.png'
omit_header_text: true
draft: true
---

```PowerShell
$AdminCenterURL="https://contoso-admin.sharepoint.com"
$tenantUrl = "https://contoso.sharepoint.com"
$hubSiteUrl = "https://contoso.sharepoint.com"

Connect-PnPOnline -Url $AdminCenterURL -Interactive
 
$logoLocalPath = (Get-Location).Path + '\Logo\logo white 1280x1280.png'
$thumbnailLocalPath = (Get-Location).Path + '\Logo\thumbnail white 1280x1280.png'
 
$m365Sites = Get-PnPHubSiteChild -Identity $hubSiteUrl

Start-Transcript
 
$m365Sites | ForEach-Object {
    Connect-PnPOnline -Url $_ -Interactive
    $logoFile = '__rectSitelogo__contoso-logo.png'
    $thumbnailFile = 'thumbnail.png'

    $logoRelativePath = $_.replace($tenantUrl, "") + "/SiteAssets/" + $logoFile
    $thumbnailRelativePath = $_.replace($tenantUrl, "") + "/SiteAssets/" + $logoFile

    $logoAslistItem =  Get-PnPFile -Url $logoRelativePath -AsListItem
    $thumbnailAslistItem =  Get-PnPFile -Url $thumbnailRelativePath -AsListItem
 
    if(!$logoAslistItem){
      Add-PnPFile -Path $logoLocalPath -Folder 'SiteAssets' -NewFileName $logoFile | Out-Null
    }
    
    if(!$thumbnailAslistItem){
      Add-PnPFile -Path $thumbnailLocalPath -Folder 'SiteAssets' -NewFileName $thumbnailFile | Out-Null
    }
 
    Set-PnPWebHeader -SiteLogoUrl $logoRelativePath -SiteThumbnailUrl $thumbnailRelativePath
}
Stop-Transcript
```

[How to change the header banner image of a sharepoint site using Power Automate](https://powerusers.microsoft.com/t5/General-Power-Automate/How-to-change-the-header-banner-image-of-a-sharepoint-site-using/td-p/1675774)
[Enhancement: spo site set enhance logo options ](https://github.com/pnp/cli-microsoft365/issues/5495)
