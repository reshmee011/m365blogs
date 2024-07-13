---
title: "Updating SharePoint Site Logo and Thumbnail with PowerShell"
date: 2024-07-12T06:39:48Z
tags: ["PowerShell","Site Logo","Thumbnail","Branding","SharePoint"]
featured_image: '/posts/images/PowerShell-update-site-header-sitelogo-thumbnail/sitelogothumbnail.png'
omit_header_text: true
draft: false
---

# Update SharePoint Site Logo and Thumbnail with PowerShell 

In SharePoint Online sites, the distinction between the `Site Logo` and `Site Thumbnail` is crucial. The site logo appears in the site header, while the site thumbnail is used in search results, site cards, file copying/moving, and other areas.

Both site logo and thumbnail are part of SharePoint branding. This post covers how to update the site logo and thumbnail across multiple SharePoint sites within a hub using PowerShell.

I used the script to update site logo and thumbnail for associated sites within a hub site which could be an intranet if there are no valid site logo and thumbnail already on the site.

## PowerShell Script

This scripts updates the site logo and thumbnail for all sites associated with a specific hub site.

Please update the variables 
 * $AdminCenterURL= sharepoint centre url
 * $tenantUrl = tenant url
 * $hubSiteUrl = hub site url
 * $logoLocalPath : path of the logo file
 * $thumbnailLocalPath = path of the thumbnail file

```PowerShell
$AdminCenterURL="https://contoso-admin.sharepoint.com"
$tenantUrl = "https://contoso.sharepoint.com"
$hubSiteUrl = "https://contoso.sharepoint.com"

$logoLocalPath = (Get-Location).Path + '\Logo\logo white 1280x1280.png'
$thumbnailLocalPath = (Get-Location).Path + '\Logo\thumbnail white 1280x1280.png'

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

After running the script

![Update Site Logo and Thumbnail](../images/PowerShell-update-site-header-sitelogo-thumbnail/sitelogothumbnail.png)

## Conclusion

Updating the site logo and thumbnail for each SharePoint site manually can be a tedious and time-consuming task, especially for organisations with a large number of sites as part of rebranding or branding exercise. 

This PowerShell script helps to automate updating site logo and site thumbnail for consistent branding across a SharePoint hub site.

## References

[How to change the header banner image of a sharepoint site using Power Automate](https://powerusers.microsoft.com/t5/General-Power-Automate/How-to-change-the-header-banner-image-of-a-sharepoint-site-using/td-p/1675774)

[Enhancement: spo site set enhance logo options ](https://github.com/pnp/cli-microsoft365/issues/5495)
