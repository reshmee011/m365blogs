#Parameters
$SiteURL = "https://contoso.sharepoint.com/teams/Committees"
$FolderSiteRelativeURL = "Shared Documents/Committee Governance/Board/Forward Plan/HR matters"
#Connect to PnP Online
Connect-PnPOnline -Url $SiteURL -Interactive
#Get the folder
$Folder = Get-PnPFolder -Url $FolderSiteRelativeURL -Includes ListItemAllFields
#Get the total Size of the folder - with versions
Write-host "Size of the Folder:" $([Math]::Round(($Folder.ListItemAllFields.FieldValues.SMTotalSize.LookupId/1KB),2))
 
 
#Get the folder size from storage metrics
$FolderSize= Get-PnPFolderStorageMetric -FolderSiteRelativeUrl $FolderSiteRelativeURL| Select -ExpandProperty TotalSize
$FolderSize = [Math]::Round($FolderSize/1MB, 2)
$FolderItemCount = Get-PnPFolderStorageMetric -FolderSiteRelativeUrl $FolderSiteRelativeURL| Select -ExpandProperty TotalFileCount
Write-host "Total size of the Folder: $FolderSize MB"
Write-host "Total File count: $FolderItemCount"
