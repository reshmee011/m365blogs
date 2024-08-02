---
title: "Delete M365 connected Team Site using PowerShell"
date: 2024-07-31T09:53:05+01:00
tags: ["PowerShell", "M365 Group", "Teams Site"]
featured_image: '/posts/images/powershell-delete-m365-connected-teamsite/Delete_Button_Disabled.png'
omit_header_text: true
draft: false
---

If a M365 group connected Team site is deleted from SharePoint Admin Centre, from the UI it is not possible to delete the site from the deleted sites to be able to resuse the URL or name of the deleted site. 

The end user is presented with the following prompt
![Delete Prompt for site deletion](../images/powershell-delete-m365-connected-teamsite/Delete_TestDT.png)

>This site belongs to a Microsoft 365 group. Deleting the site will delete the group and all its resources, including the Outlook mailbox and calendar, and any Teams channels. You will have 30 days to restore the group.

After deletion the site is available for 93 days for restore with inability to delete within 30 days of deletion or until after the M365 Group is deleted automatically after 30 days or through a script.
![Delete Button Disabled for the deleted site](../images/powershell-delete-m365-connected-teamsite/Delete_Button_Disabled.png)

The corresponding M365 Group from the deleted M365 Groups can't be deleted with no delete button
![Delete Button Disabled for the M365 Group](../images/powershell-delete-m365-connected-teamsite/UnableToDeleteM365GroupFromUI.png)

The alternative way not to wait for the 30 days period until the M365 Group is deleted in the background is to run a PowerShell script to perform the deletion of the site and M365 group

```PowerShell
Try {
    $AdminCenterURL="https://contoso-admin.sharepoint.com/"
    $SiteURL = "https://contoso.sharepoint.com/sites/test"
    #Connect to SharePoint admin site
    Connect-PnPOnline -Url $AdminCenterURL -Interactive
    #Get the SharePoint Online site
    $Site = Get-PnPTenantSite -Identity $SiteURL -errorAction Ignore
 
    #Delete the Office 365 Group
    Remove-PnPMicrosoft365Group -Identity $Site.GroupId -errorAction Ignore
    #permanently remove the m365 group
    Remove-PnPDeletedMicrosoft365Group -Identity $Site.GroupId -errorAction Ignore
 
    #Permanently Delete the SharePoint site
    $dleted = $false;
    do {
        Start-Sleep -Seconds 60
        try {
           Remove-PnPTenantSite -Url $SiteUrl -Force -SkipRecycleBin
        $dleted = $true;      
        }
        catch {
           Write-Host "Delete site"
        }
 
        try {
            Remove-PnPTenantDeletedSite -Url $SiteUrl -Force -SkipRecycleBin
         $dleted = $true;      
         }
         catch {
            Write-Host "Permanently delete site"
         }
 
    } while (!$dleted)#>
}
Catch {
     write-host "Error: $($_.Exception.Message)" -foregroundcolor Red
}
```