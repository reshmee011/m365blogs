---
title: "PowerShell_PnPUnifiedLog"
date: 2023-11-29T16:16:09Z
draft: true
---

Actions


FileModifiedExtended
FileAccessed
ListViewed
PageViewed
FileAccessedExtended
FileModified
FileRenamed
FileDownloaded
FilePreviewed
FileUploaded
ListItemViewed
SearchQueryPerformed
PagePrefetched
ListItemCreated
ListItemRecycled
FileSyncDownloadedFull
FileCheckedIn
ListItemUpdated
RemovedFromGroup
SiteCollectionAdminRemoved
SignInEvent
FileRecycled
SharingSet
AddedToGroup
CompanyLinkCreated
SharingInheritanceBroken
GroupAdded
PageViewedExtended
FileCheckedOut
FileVersionsAllDeleted
FolderCreated
SharingInvitationBlocked
FileSyncUploadedFull
ClientViewSignaled
FileSensitivityLabelChanged
FileMoved
FolderRenamed
CompanyLinkUsed
FileSensitivityLabelApplied
SecureLinkUsed
FileCheckOutDiscarded
ListColumnCreated
ListUpdated
SharingRevoked
FileCopied
AppCatalogSPCorporateCatalogAccessorBaseAdd
FolderModified
SPCorporateCatalogAppMetadataDeployWithFeatureDeployment
AppCatalogCreateSiteCollectionAppCatalog
FileDeleted
SecureLinkCreated
ListCreated
AddedToSecureLink
SecureLinkUpdated
FolderMoved
SiteColumnCreated
SiteDeleted
SiteLocksChanged
```

## Script

```
$SiteUrl = "https://contoso.sharepoint.com/teams/team-devtest/"
Connect-PnPOnline -url $SiteUrl -Interactive
$userId = "user@contoso.co.uk"
$days = 30
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
