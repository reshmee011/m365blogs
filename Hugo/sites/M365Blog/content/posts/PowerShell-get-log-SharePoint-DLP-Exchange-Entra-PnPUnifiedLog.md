---
title: "Unveiling Audit Logs with PnP PowerShell"
date: 2023-12-14T16:16:09Z
tags: ["SharePoint","PnP","PowerShell", "Audit Logs","DLP", "Exchange", "AzureDirectory"]
featured_image: '/posts/images/PowerShell_PnPUnifiedLog/Sample.png'
draft: false
---

# Unveiling Audit Logs with PnP PowerShell for the last 7 days 

Understanding and tracking activities within your M365 environment is crucial for maintaining security and compliance. Audit Logs offer a wealth of information, and in this post, we'll delve into the methods to retrieve and analyze them. Specifically, we'll focus on leveraging the Office 365 Management Activity API reference through the Get-PnPUnifiedAuditLog PnP PowerShell cmdlet. It is a great alternative if you are only a SharePoint Administrator with no global admin or Purview Audit logs access.
The scenerio I had to use it was to trace a non existent file the end user definitely thought have saved somewhere in M365, hence the filter of activities by user id. Unfortunately the file could not be found at all even rename or move of file.

## Retrieving Audit Logs

There are various methods to retrieve SharePoint Audit Logs:

* The (Office 365 Management Activity API reference)[https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference]
* The (audit log search tool)[https://learn.microsoft.com/en-us/purview/audit-new-search] in the Microsoft Purview compliance portal, provides another avenue for extracting audit logs.
* The (Search-UnifiedAuditLog cmdlet)[https://learn.microsoft.com/en-us/powershell/module/exchange/search-unifiedauditlog] in Exchange Online PowerShell provides another option. 

This post focuses on utilizing the (Office 365 Management Activity API reference)[https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference] reference with the Get-PnPUnifiedAuditLog PnP PowerShell cmdlet. Regularly retrieving audit logs is essential for large organizations, ensuring scalability and performance in handling millions of audit records continuously. For alternative methods, refer to the post (Use a PowerShell script to search the audit log)[https://learn.microsoft.com/en-us/purview/audit-log-search-script] for more details.

## SharePoint Audit Actions

The Get-PnPUnifiedAuditLog cmdlet provides detailed insights into various SharePoint activities, including but not limited to:

* AddedToGroup
* AddedToSecureLink
* AppCatalogCreateSiteCollectionAppCatalog
* AppCatalogSPCorporateCatalogAccessorBaseAdd
* ClientViewSignaled
* CompanyLinkCreated
* CompanyLinkUsed
* FileAccessed
* FileAccessedExtended
* FileCheckOutDiscarded
* FileCheckedIn
* FileCheckedOut
* FileCopied
* FileDeleted
* FileDownloaded
* FileModified
* FileModifiedExtended
* FileMoved
* FilePreviewed
* FileRecycled
* FileRenamed
* FileSensitivityLabelApplied
* FileSensitivityLabelChanged
* FileSyncDownloadedFull
* FileSyncUploadedFull
* FileUploaded
* FileVersionsAllDeleted
* FolderCreated
* FolderModified
* FolderMoved
* FolderRenamed
* GroupAdded
* ListColumnCreated
* ListCreated
* ListItemCreated
* ListItemRecycled
* ListItemUpdated
* ListItemViewed
* ListUpdated
* ListViewed
* PagePrefetched
* PageViewed
* PageViewedExtended
* RemovedFromGroup
* SecureLinkCreated
* SecureLinkUpdated
* SecureLinkUsed
* SharingInheritanceBroken
* SharingInvitationBlocked
* SharingRevoked
* SharingSet
* SignInEvent
* SiteCollectionAdminRemoved
* SiteColumnCreated
* SiteDeleted
* SiteLocksChanged
* SPCorporateCatalogAppMetadataDeployWithFeatureDeployment
* SearchQueryPerformed

## Azure Active Directory Audit Actions

For Azure Active Directory, potential actions include:

* Add app role assignment grant to user.
* Add device.
* Add group.
* Add member to group.
* Add member to role.
* Add owner to group.
* Add registered owner to device.
* Add registered users to device.
* Add service principal.
* Add service principal credentials.
* Change user license.
* Change user password.
* Consent to application.
* Delete device.
* Device no longer compliant.
* Device no longer managed.
* Remove member from group.
* Remove member from role.
* Remove service principal.
* Remove service principal credentials.
* Reset user password.
* Set Company Information.
* Update agreement.
* Update device.
* Update group.
* Update service principal.
* Update StsRefreshTokenValidFrom Timestamp.
* Update user.
* UserLoggedIn
* UserLoginFailed

## Exchange Audit Actions

Possible activities retrieved by the cmdlet **Get-PnPUnifiedAuditLog** for Exchange

* Add-MailboxPermission
* Create
* Enable-AddressListPaging
* FolderBind
* HardDelete
* Install-AdminAuditLogConfig
* Install-DataClassificationConfig
* Install-DefaultSharingPolicy
* Install-ResourceConfig
* MailItemsAccessed
* MipLabel
* ModifyFolderPermissions
* Move
* MoveToDeletedItems
* New-ExchangeAssistanceConfig
* New-Mailbox
* Send
* SendAs
* SendOnBehalf
* Set-AdminAuditLogConfig
* Set-ExchangeAssistanceConfig
* Set-Mailbox
* Set-MailboxPlan
* Set-OwaMailboxPolicy
* Set-RecipientEnforcementProvisioningPolicy
* Set-TenantObjectVersion
* Set-TransportConfig
* SoftDelete
* Update
* UpdateInboxRules

## Data Loss Prevention (DLP) Audit Actions

Possible activities retrieved by the cmdlet **Get-PnPUnifiedAuditLog** for DLP:

* AddedToGroup
* AddedToSecureLink
* ClientViewSignaled
* CompanyLinkCreated
* CompanyLinkUsed
* CommentCreated
* FileAccessed
* FileAccessedExtended
* FileCheckOutDiscarded
* FileCheckedIn
* FileCheckedOut
* FileCopied
* FileDownloaded
* FileModified
* FileModifiedExtended
* FileMoved
* FilePreviewed
* FileRecycled
* FileRenamed
* FileSensitivityLabelApplied
* FileSyncDownloadedFull
* FileSyncUploadedFull
* FileUploaded
* FileVersionsAllDeleted
* FolderCreated
* FolderModified
* FolderMoved
* FolderRecycled
* GroupAdded
* ListColumnCreated
* ListColumnDeleted
* ListItemCreated
* ListItemDeleted
* ListItemRecycled
* ListItemUpdated
* ListItemViewed
* ListViewUpdated
* ListViewed
* ManagedSyncClientAllowed
* PagePrefetched
* PageViewed
* PageViewedExtended
* SearchQueryPerformed
* SecureLinkCreated
* SecureLinkUpdated
* SecureLinkUsed
* SharingInheritanceBroken
* SharingInvitationBlocked
* SharingSet
* SignInEvent
* SiteColumnCreated

## General 

Possible activities retrieved by the cmdlet **Get-PnPUnifiedAuditLog** for General:

* SensitivityLabelApplied      
* SensitivityLabeledFileOpened 
* SensitivityLabeledFileRenamed
* CrmDefaultActivity
* Search
* ReactedToMessage
* MessageCreatedHasLink        
* MessageReadReceiptReceived   
* ThreadViewed
* ViewReport
* ViewResponses
* ViewRuntimeForm
* MDCAssessments
* TeamsSessionStarted
* CreateResponse
* LaunchPowerApp
* MemberAdded
* SensitivityLabelUpdated
* MessageUpdated
* TIMailData
* ViewDashboard
* AddTile
* CreateDashboard
* GetSnapshots
* DeleteDashboard
* GenerateCustomVisualAADAccessToken
* AnalyzedByExternalApplication
* ViewedApprovalRequest
* ApprovedRequest
* TeamsUserSignedOut
* ExportReport
* MessageSent
* PlanListRead
* MessageCreation
* RespondedWithCustomResponse
* MarkedMessageChanged
* Import
* MDCRegulatoryComplianceAssessments
* RefreshDataset
* EditDataset
* Validate

## PnP PowerShell Script 

```PowerShell
$SiteUrl = "https://contoso-admin.sharepoint.com"
Connect-PnPOnline -url $SiteUrl -Interactive
$userId = "testusero@contoso.co.uk"
$days = 3
$endDay = 0
$Operations = @()
 
# Generate a unique log file name using today's date
$dateTime = (Get-Date).toString("dd-MM-yyyy_HHmm")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "logReport-" + $dateTime + ".csv"
$OutPutView = $directorypath + "\Logs\"+ $fileName
 
$logCollection = @()
while($days -ge $endDay){
if($days -eq 0)
{
 $activities =  Get-PnPUnifiedAuditLog -ContentType SharePoint -ErrorAction Ignore
 $activities +=  Get-PnPUnifiedAuditLog -ContentType AzureActiveDirectory -ErrorAction Ignore
 $activities +=  Get-PnPUnifiedAuditLog -ContentType DLP -ErrorAction Ignore
 $activities +=  Get-PnPUnifiedAuditLog -ContentType Exchange -ErrorAction Ignore
 $activities +=  Get-PnPUnifiedAuditLog -ContentType General -ErrorAction Ignore
 
}else {
    $activities = Get-PnPUnifiedAuditLog -ContentType AzureActiveDirectory -ErrorAction Ignore  -StartTime (Get-date).adddays(-$days) -EndTime (Get-date).adddays(-($days-1))
    $activities += Get-PnPUnifiedAuditLog -ContentType SharePoint -ErrorAction Ignore  -StartTime (Get-date).adddays(-$days) -EndTime (Get-date).adddays(-($days-1))
    $activities += Get-PnPUnifiedAuditLog -ContentType DLP -ErrorAction Ignore  -StartTime (Get-date).adddays(-$days) -EndTime (Get-date).adddays(-($days-1))
    $activities += Get-PnPUnifiedAuditLog -ContentType Exchange -ErrorAction Ignore  -StartTime (Get-date).adddays(-$days) -EndTime (Get-date).adddays(-($days-1))
    $activities += Get-PnPUnifiedAuditLog -ContentType General -ErrorAction Ignore  -StartTime (Get-date).adddays(-$days) -EndTime (Get-date).adddays(-($days-1))
 }
 
 $activities| ForEach-Object {
    $activity = $_
    if($Operations -notcontains $activity.Operation){
      $Operations+= $activity.Operation
      write-host $activity.Operation
  }
    if($activity.UserId ){#-and $activity.SiteUrl
       if($activity.UserId.ToLower() -eq $userId  )    #-and $activity.SiteUrl.ToLower() -eq $SiteUrl
         {      
            $logCollection += $activity
         }
      }
   }
   $days = $days - 1
}
$logCollection | sort-object "Operation" |Export-CSV $OutPutView -Force -NoTypeInformation

```

The selected PowerShell script uses the Get-PnPUnifiedAuditLog cmdlet from the PnP (Patterns and Practices) PowerShell library. This cmdlet is used to retrieve audit log entries from the Office 365 Unified Audit Log for the past 7 days for the different content types: SharePoint, DLP, Exchange, AzureActiveDirectory and General.

The logs are filtered based on a specific user ID and sorted by operation type. The script also generates a unique log file name using the current date and time. 

I have not been able to retrieve audit data older than 7 days from the tenant using the script and I have not found any documentation on this. 

## Conclusion

This blog post will provide a comprehensive guide to using PnP PowerShell for retrieving and working with SharePoint Audit Logs, making it a valuable resource for SharePoint administrators and developers. Other altenatives is to the **Search-UnifiedAuditLog** as described hin (How to Search the Microsoft 365 Audit Log for Events)[https://office365itpros.com/2020/10/08/search-microsoft-365-audit-log/].


## References

(Report of SharePoint Files Incidents - PowerShell Script)[https://michalkornet.com/2023/09/19/Report-of-SharePoint-Files-Incidents.html]

(Enumerate activities (preview))[https://learn.microsoft.com/en-us/onedrive/developer/rest-api/api/activities_list?view=odsp-graph-online]

(ItemActivity resource type)[https://learn.microsoft.com/en-us/onedrive/developer/rest-api/resources/itemactivity?view=odsp-graph-online]

(Get-PnPUnifiedAuditLog)[https://pnp.github.io/powershell/cmdlets/Get-PnPUnifiedAuditLog]

(Use a PowerShell script to search the audit log)[https://learn.microsoft.com/en-us/purview/audit-log-search-script]
