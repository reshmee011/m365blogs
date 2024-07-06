---
title: "PowerShell Storage Nearing Exceeding Quota"
date: 2024-04-21T17:32:36Z
tags: ["psm1","PowerShell","PnP","PowerShell Script Module"]
featured_image: '/posts/images/PowerShell-editing-psm1-reload/script.png'
omit_header_text: true
draft: true
---

# PowerShell Storage Nearing Exceeding Quota

```PowerShell
$AdminCenterURL="https://contoso-admin.sharepoint.com/"# Connect to SharePoint Online admin center
Connect-PnPOnline -Url $AdminCenterURL -Interactive
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "SiteStorageReport-" + $dateTime + ".csv"
$OutPutView = $directorypath + "\Logs\"+ $fileName
# Array to Hold Result - PSObjects
$siteCollection = @()
# 
$m365Sites = Get-PnPTenantSite -Detailed -IncludeOneDriveSites  | Where-Object { $_.Template -ne 'RedirectSite#0' } #($_.Url -like '*-my.sharepoint.com/personal/*') -and
#$m365Groups = Get-PnPMicrosoft365Group | where-object {$_.DisplayName -eq "Dev Mars"}
$m365Sites | ForEach-Object {
   $PercentUsed=[Math]::Round(($_.StorageUsageCurrent/$_.StorageQuota *100),3)
   if( $PercentUsed -gt 80){
   $oneDrive = [PSCustomObject]@{
    SiteName = $_.Title
    LastContentModifiedDate=$_.LastContentModifiedDate

   Template=$_.Template
   SensitivityLabel=$_.SensitivityLabel
   "Storage Maximum Level"=$_.StorageMaximumLevel
   StorageUsageCurrentMB=$_.StorageUsageCurrent
   StorageQuota=$_.StorageQuota
   "StorageQuotaWarning Level"=$_.StorageQuotaWarningLevel
   SharingCapability=$_.SharingCapability
   PercentUsed=$PercentUsed
   }
   $oneDriveCollection+=$oneDrive
 }
}
# Export the result array to CSV file
$siteCollection | sort-object "Group Name" |Export-CSV $OutPutView -Force -NoTypeInformation

# Disconnect SharePoint online connection

Disconnect-PnPOnline
```
