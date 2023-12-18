---
title: "SharePoint Audit Logs"
date: 2023-12-14T16:16:09Z
draft: true
---

# Get SharePoint Audit logs for the last 7 days

The following methods can help to retrieve audit logs.

* The (Office 365 Management Activity API reference)[https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference]
* The (audit log search tool)[https://learn.microsoft.com/en-us/purview/audit-new-search] in the Microsoft Purview compliance portal
* The (Search-UnifiedAuditLog cmdlet)[https://learn.microsoft.com/en-us/powershell/module/exchange/search-unifiedauditlog] in Exchange Online PowerShell

This post focuses on the first option using (Office 365 Management Activity API reference)[https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference] with the help of **Get-PnPUnifiedAuditLog** PnP PowerShell. It might be useful to retrieve audit logs regularly because it can provide large organisations with the scalability and performance to retrieve millions of audit records on an ongoing basis.The two other methods have their merits too, see post (Use a PowerShell script to search the audit log)[https://learn.microsoft.com/en-us/purview/audit-log-search-script] for more details.

## Actions

Possible activities retrieved by the cmdlet **Get-PnPUnifiedAuditLog** for SharePoint 

* FileM*odifiedExtended
* FileAccessed
* ListViewed
* PageViewed
* FileAccessedExtended
* FileModified
* FileRenamed
* FileDownloaded
* FilePreviewed
* FileUploaded
* ListItemViewed
* SearchQueryPerformed
* PagePrefetched
* ListItemCreated
* ListItemRecycled
* FileSyncDownloadedFull
* FileCheckedIn
* ListItemUpdated
* RemovedFromGroup
* SiteCollectionAdminRemoved
* SignInEvent
* FileRecycled
* SharingSet
* AddedToGroup
* CompanyLinkCreated
* SharingInheritanceBroken
* GroupAdded
* PageViewedExtended
* FileCheckedOut
* FileVersionsAllDeleted
* FolderCreated
* SharingInvitationBlocked
* FileSyncUploadedFull
* ClientViewSignaled
* FileSensitivityLabelChanged
* FileMoved
* FolderRenamed
* CompanyLinkUsed
* FileSensitivityLabelApplied
* SecureLinkUsed
* FileCheckOutDiscarded
* ListColumnCreated
* ListUpdated
* SharingRevoked
* FileCopied
* AppCatalogSPCorporateCatalogAccessorBaseAdd
* FolderModified
* SPCorporateCatalogAppMetadataDeployWithFeatureDeployment
* AppCatalogCreateSiteCollectionAppCatalog
* FileDeleted
* SecureLinkCreated
* ListCreated
* AddedToSecureLink
* SecureLinkUpdated
* FolderMoved
* SiteColumnCreated
* SiteDeleted
* SiteLocksChanged

, , SharePoint, , DLP. 
Possible activities retrieved by the cmdlet **Get-PnPUnifiedAuditLog** for AzureActiveDirectory

Possible activities retrieved by the cmdlet **Get-PnPUnifiedAuditLog** for Exchange

Possible activities retrieved by the cmdlet **Get-PnPUnifiedAuditLog** for DLP

Possible activities retrieved by the cmdlet **Get-PnPUnifiedAuditLog** for General

## Script


```PowerShell
$SiteUrl = "https://contoso.sharepoint.com/teams/team-devtest/"
$userId = "test@contoso.sharepoint.com"
Connect-PnPOnline -url $SiteUrl -Interactive
$days = 7
$endDay = 0
$Operations = @()
 
# Generate a unique log file name using today's date
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "logReport-" + $dateTime + ".csv"
$OutPutView = $directorypath + "\Logs\"+ $fileName
 
$logCollection = @()
while($days -ge $endDay){
if($days -eq 0)
{
 $activities =  Get-PnPUnifiedAuditLog -ContentType SharePoint -ErrorAction Ignore
}else {
    $activities = Get-PnPUnifiedAuditLog -ContentType SharePoint -ErrorAction Ignore  -StartTime (Get-date).adddays(-$days) -EndTime (Get-date).adddays(-($days-1))
 }
 
 $activities| ForEach-Object {
    $activity = $_
if($activity.UserId -and $activity.SiteUrl){
   if($activity.UserId.ToLower() -eq $userId  -and $activity.SiteUrl.ToLower() -eq $SiteUrl)    
 {  
    if($Operations -notcontains $activity.Operation){
        $Operations+= $activity.Operation
        write-host $activity.Operation
    }
    $ExportVw = New-Object PSObject
    $ExportVw | Add-Member -MemberType NoteProperty -name "ObjectId" -value $activity.ObjectId
    $ExportVw | Add-Member -MemberType NoteProperty -name "SiteUrl" -value $activity.SiteUrl
    $ExportVw | Add-Member -MemberType NoteProperty -name "Operation" -value $activity.Operation
    $ExportVw | Add-Member -MemberType NoteProperty -name "UserId" -value $activity.UserId
    $ExportVw | Add-Member -MemberType NoteProperty -name "Date" -value $activity.CreationTime
    $logCollection += $ExportVw
 }
}
}
 $days = $days - 1
}
$logCollection | sort-object "ObjectId" |Export-CSV $OutPutView -Force -NoTypeInformation

```

The selected PowerShell script uses the Get-PnPUnifiedAuditLog cmdlet from the PnP (Patterns and Practices) PowerShell library. This cmdlet is used to retrieve audit log entries from the Office 365 Unified Audit Log.

Here's a breakdown of the script:

It first connects to a SharePoint site using Connect-PnPOnline.

It sets up a loop to retrieve audit logs for the past 7 days.

Inside the loop, it calls Get-PnPUnifiedAuditLog to retrieve SharePoint content type audit logs for each day. If it's the current day, it retrieves the logs without specifying the start and end time. For previous days, it specifies the start and end time.

For each retrieved activity, it checks if the activity's UserId and SiteUrl match the specified user ID and site URL. If they do, it creates a new PSObject (PowerShell Object) with properties like ObjectId, SiteUrl, Operation, UserId, and Date, and adds this object to the log collection.

After all activities are processed, it decreases the days variable by 1 to move to the previous day.

After the loop finishes, it sorts the log collection by ObjectId and exports it to a CSV file.

The Get-PnPUnifiedAuditLog cmdlet is a powerful tool for auditing SharePoint activities as it provides detailed information about each activity, including the user who performed the activity, the object affected, the operation performed, and the time of the activity.

I have not been able to retrieve audit data older than 7 daysfrom the tenant I used the script. 

## Conclusion

This blog post will provide a comprehensive guide to using PnP PowerShell for retrieving and working with SharePoint Audit Logs, making it a valuable resource for SharePoint administrators and developers. Other altenatives is to the **Search-UnifiedAuditLog** as described within (How to Search the Microsoft 365 Audit Log for Events)[https://office365itpros.com/2020/10/08/search-microsoft-365-audit-log/].


## References

(Report of SharePoint Files Incidents - PowerShell Script)[https://michalkornet.com/2023/09/19/Report-of-SharePoint-Files-Incidents.html]
(Enumerate activities (preview))[https://learn.microsoft.com/en-us/onedrive/developer/rest-api/api/activities_list?view=odsp-graph-online]
(ItemActivity resource type)[https://learn.microsoft.com/en-us/onedrive/developer/rest-api/resources/itemactivity?view=odsp-graph-online]
(Get-PnPUnifiedAuditLog)[https://pnp.github.io/powershell/cmdlets/Get-PnPUnifiedAuditLog]
(Use a PowerShell script to search the audit log)[https://learn.microsoft.com/en-us/purview/audit-log-search-script]
