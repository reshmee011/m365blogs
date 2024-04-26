---
title: "PowerShell Update Site Header Sitelogo Thumbnail"
date: 2024-02-11T06:39:48Z
draft: false
---

```PowerShell
$AdminCenterURL="https://contoso-admin.sharepoint.com"
$tenantUrl = "https://contoso.sharepoint.com"
$hubSiteUrl = "https://ppfonline.sharepoint.com"
Connect-PnPOnline -Url $AdminCenterURL -Interactive
 
$logoLocalPath = (Get-Location).Path + '\Logo\logo white 1280x1280.png'
 
$m365Sites = Get-PnPHubSiteChild -Identity $hubSiteUrl

Start-Transcript
 
$m365Sites | ForEach-Object {
    Connect-PnPOnline -Url $_ -Interactive
    $logoFile = '__rectSitelogo__contoso-logo.png'
  
    $logoRelativePath = $_.replace($tenantUrl, "") + "/SiteAssets/" + $logoFile
 
    $fileAslistItem =  Get-PnPFile -Url $logoRelativePath -AsListItem
 
    if(!$fileAslistItem){
      Add-PnPFile -Path $logoLocalPath -Folder 'SiteAssets' -NewFileName $logoFile | Out-Null
    }
 
    Set-PnPWebHeader -SiteLogoUrl $logoRelativePath
}
Stop-Transcript
```

[How to change the header banner image of a sharepoint site using Power Automate](https://powerusers.microsoft.com/t5/General-Power-Automate/How-to-change-the-header-banner-image-of-a-sharepoint-site-using/td-p/1675774)
[Enhancement: spo site set enhance logo options ](https://github.com/pnp/cli-microsoft365/issues/5495)
