---
title: "Unveiling Audit Logs with PnP and Cli for M365 PowerShell"
date: 2024-01-31T16:16:09Z
tags: ["SharePoint","PnP","PowerShell","CLI for M365" ,"Audit Logs","DLP", "Exchange", "AzureDirectory"]
featured_image: '/posts/images/PowerShell_PnPUnifiedLog/Sample.png'
draft: false
---

# Unveiling Audit Logs with PnP PowerShell for the last 7 days 

Understanding and tracking activities within your M365 environment is crucial for maintaining security and compliance. Audit Logs offer a wealth of information, and in this post, we'll delve into the methods to retrieve and analyze them. Specifically, we'll focus on leveraging the Office 365 Management Activity API reference through the Get-PnPUnifiedAuditLog PnP PowerShell cmdlet. It is a great alternative if you are only a SharePoint Administrator with no global admin or Purview Audit logs access.
The scenerio I had to use it was to trace a non existent file the end user definitely thought have saved somewhere in M365, hence the filter of activities by user id. Unfortunately the file could not be found at all even rename or move of file.

## Retrieving Audit Logs

There are various methods to retrieve SharePoint Audit Logs:

* The [Office 365 Management Activity API reference](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference?wt.mc_id=MVP_308367)
* The [audit log search tool](https://learn.microsoft.com/en-us/purview/audit-new-search?wt.mc_id=MVP_308367) in the Microsoft Purview compliance portal, provides another avenue for extracting audit logs.
* The [Search-UnifiedAuditLog cmdlet](https://learn.microsoft.com/en-us/powershell/module/exchange/search-unifiedauditlog?wt.mc_id=MVP_308367) in Exchange Online PowerShell provides another option. 

This post focuses on utilizing the [Office 365 Management Activity API reference](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference?wt.mc_id=MVP_308367) reference with the Get-PnPUnifiedAuditLog PnP PowerShell cmdlet. Regularly retrieving audit logs is essential for large organizations, ensuring scalability and performance in handling millions of audit records continuously. For alternative methods, refer to the post [Use a PowerShell script to search the audit log](https://learn.microsoft.com/en-us/purview/audit-log-search-script?wt.mc_id=MVP_308367) for more details.

## PnP PowerShell Script 

```PowerShell
$SiteUrl = "https://contoso-admin.sharepoint.com"
Connect-PnPOnline -url $SiteUrl -Interactive
$userId = "testusero@contoso.co.uk"
$days = 7
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
 $activities +=  Get-PnPUnifiedAuditLog -ContentType SharePoint -ErrorAction Ignore
 $activities +=  Get-PnPUnifiedAuditLog -ContentType AzureActiveDirectory -ErrorAction Ignore
 $activities +=  Get-PnPUnifiedAuditLog -ContentType DLP -ErrorAction Ignore
 $activities +=  Get-PnPUnifiedAuditLog -ContentType Exchange -ErrorAction Ignore
 $activities +=  Get-PnPUnifiedAuditLog -ContentType General -ErrorAction Ignore
 
}else {
    $activities += Get-PnPUnifiedAuditLog -ContentType AzureActiveDirectory -ErrorAction Ignore  -StartTime (Get-date).adddays(-$days) -EndTime (Get-date).adddays(-($days-1))
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

The selected PowerShell script uses the [Get-PnPUnifiedAuditLog](https://pnp.github.io/powershell/cmdlets/Get-PnPUnifiedAuditLog.html) . This cmdlet is used to retrieve audit log entries from the Office 365 Unified Audit Log for the past 7 days for the different content types: SharePoint, DLP, Exchange, AzureActiveDirectory and General.

The logs are filtered based on a specific user ID and sorted by operation type. The script also generates a unique log file name using the current date and time. 
The next section is the same script using CLI for M365 

## CLI for M365 PowerShell Script 

```PowerShell
$SiteUrl = "https://contoso.sharepoint.com/sites/test"
$userId = "testusero@contoso.co.uk" 
Write-Host "Ensure logged in"
$m365Status = m365 status --output text
if ($m365Status -eq "Logged Out") {
  Write-Host "Logging in the User!"
  m365 login --authType browser
}
$days = 7
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
    $activities +=  m365 purview auditlog list --contentType SharePoint --output 'json' | ConvertFrom-Json
    $activities +=  m365 purview auditlog list --contentType AzureActiveDirectory --output 'json' | ConvertFrom-Json #-ErrorAction Ignore
    $activities +=  m365 purview auditlog list --contentType DLP --output 'json' | ConvertFrom-Json
    $activities +=  m365 purview auditlog list --contentType Exchange --output 'json' | ConvertFrom-Json
    $activities +=  m365 purview auditlog list --contentType General --output 'json' | ConvertFrom-Json
 
}else {
   $activities += m365 purview auditlog list --contentType SharePoint --startTime ((Get-date).adddays(-$days) | Get-Date -uFormat '%Y-%m-%d') --endTime ((Get-date).adddays(-($days-1)) | Get-Date -uFormat '%Y-%m-%d') --output 'json' | ConvertFrom-Json
   $activities += m365 purview auditlog list --contentType AzureActiveDirectory --startTime ((Get-date).adddays(-$days) | Get-Date -uFormat '%Y-%m-%d') --endTime ((Get-date).adddays(-($days-1)) | Get-Date -uFormat '%Y-%m-%d') --output 'json'| ConvertFrom-Json
   $activities += m365 purview auditlog list --contentType DLP -ErrorAction Ignore --startTime ((Get-date).adddays(-$days) | Get-Date -uFormat '%Y-%m-%d') --endTime ((Get-date).adddays(-($days-1)) | Get-Date -uFormat '%Y-%m-%d') --output 'json' | ConvertFrom-Json
   $activities += m365 purview auditlog list --contentType Exchange -ErrorAction Ignore --startTime ((Get-date).adddays(-$days) | Get-Date -uFormat '%Y-%m-%d') --endTime ((Get-date).adddays(-($days-1)) | Get-Date -uFormat '%Y-%m-%d') --output 'json' | ConvertFrom-Json
   $activities += m365 purview auditlog list --contentType General -ErrorAction Ignore  --startTime ((Get-date).adddays(-$days) | Get-Date -uFormat '%Y-%m-%d') --endTime ((Get-date).adddays(-($days-1)) | Get-Date -uFormat '%Y-%m-%d') --output 'json' | ConvertFrom-Json
 }
 
if($activity.SiteUrl ){#-and $activity.SiteUrl
   if($activity.SiteUrl.ToLower() -eq $SiteUrl)    #-$activity.UserId.ToLower() -eq $userId
 {  
    $logCollection += $activity
 }
```
Caution

Due to the amount of data retrieved, I faced memory limit reached issue

```PowerShell
[17820:000002897A066730]   125051 ms: Mark-sweep 3795.4 (4143.6) -> 3793.7 (4142.9) MB, 2069.6 / 0.0 ms  (average mu = 0.627, current mu = 0.035) allocation failure; scavenge might not succeed
[17820:000002897A066730]   126951 ms: Mark-sweep 3809.4 (4142.9) -> 3807.8 (4174.9) MB, 1870.6 / 0.0 ms  (average mu = 0.424, current mu = 0.015) allocation failure; scavenge might not succeed
 
 
<--- JS stacktrace --->
 
FATAL ERROR: Reached heap limit Allocation failed - JavaScript heap out of memory
1: 00007FF61A2E806F node_api_throw_syntax_error+175583
2: 00007FF61A26A976 v8::internal::wasm::WasmCode::safepoint_table_offset+59766
3: 00007FF61A26C660 v8::internal::wasm::WasmCode::safepoint_table_offset+67168
4: 00007FF61AD173A4 v8::Isolate::ReportExternalAllocationLimitReached+116
5: 00007FF61AD02732 v8::Isolate::Exit+674
```

Also has issues retrieving some activities due to lack of permission

```PowerShell
ThreadAccessFailure
Error: The permission set (ActivityFeed.Read ServiceHealth.Read) sent in the request does not include the expected permission.
```

## Conclusion

I have not been able to retrieve audit data older than 7 days from the tenant using the script and I have not found any documentation on this. 

This blog post will provide a comprehensive guide to using PnP PowerShell for retrieving and working with SharePoint Audit Logs, making it a valuable resource for SharePoint administrators and developers. Other altenatives is to the **Search-UnifiedAuditLog** as described hin [How to Search the Microsoft 365 Audit Log for Events](https://office365itpros.com/2020/10/08/search-microsoft-365-audit-log?wt.mc_id=MVP_308367).


## SharePoint Audit Actions

The Get-PnPUnifiedAuditLog cmdlet provides detailed insights into various SharePoint activities, including but not limited to:

* AccessRequestCreated
* AddedToGroup
* AddedToSecureLink
* AppCatalogCreateSiteCollectionAppCatalog
* AppCatalogSPCorporateCatalogAccessorBaseAdd
* AppStoreStorefrontTaskGetApps
* AutoSensitivityLabelRuleMatch
* ClientViewSignaled
* CommentCreated
* CommentsDisabled
* CompanyLinkCreated
* CompanyLinkUsed
* DisableSharingForNonOwners
* FileAccessed
* FileAccessedExtended
* FileCheckedIn
* FileCheckedOut
* FileCheckOutDiscarded
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
* FolderCopied
* FolderCreated
* FolderModified
* FolderMoved
* FolderRecycled
* FolderRenamed
* GroupAdded
* GroupUpdated
* ListColumnCreated
* ListColumnUpdated
* ListCreated
* ListItemCreated
* ListItemDeleted
* ListItemRecycled
* ListItemUpdated
* ListItemViewed
* ListUpdated
* ListViewed
* ListViewUpdated
* ManagedSyncClientAllowed
* MipAutoLabelSimulationCompletion
* MipAutoLabelSimulationProgress
* MipAutoLabelSimulationStatistics
* PagePrefetched
* PageViewed
* PageViewedExtended
* PIMRoleAssigned
* RemovedFromGroup
* SearchQueryPerformed
* SecureLinkCreated
* SecureLinkUpdated
* SecureLinkUsed
* SharingInheritanceBroken
* SharingInvitationBlocked
* SharingLinkCreated
* SharingPolicyChanged
* SharingRevoked
* SharingSet
* ShortcutAdded
* SignInEvent
* SiteCollectionAdminAdded
* SiteCollectionAdminRemoved
* SiteCollectionCreated
* SiteCollectionQuotaModified
* SiteColumnCreated
* SiteDeleted
* SiteDesignScheduled
* SiteIBModeSet
* SiteLocksChanged
* SPCorporateCatalogAppMetadataDeployWithFeatureDeployment
* WebMembersCanShareModified
* WebRequestAccessModified

## Azure Active Directory Audit Actions

For Azure Active Directory, potential actions include:

* Add a partner to cross-tenant access setting.
* Add app role assignment grant to user.
* Add app role assignment to service principal.
* Add delegated permission grant.
* Add device.
* Add group.
* Add member to group.
* Add member to role.
* Add owner to group.
* Add owner to service principal.
* Add registered owner to device.
* Add registered users to device.
* Add service principal credentials.
* Add service principal.
* Add user.
* Change user license.
* Change user password.
* Consent to application.
* Delete device.
* Delete group.
* Delete user.       
* Device no longer compliant.
* Device no longer managed.
* Disable account.
* Enable account.
* Remove delegated permission grant.
* Remove member from group.
* Remove member from role.
* Remove owner from group.
* Remove registered owner from device.
* Remove registered users from device.
* Remove service principal credentials.
* Remove service principal.
* Reset user password.
* Set Company Information.
* Update a partner cross-tenant access setting.
* Update agreement.
* Update application – Certificates and secrets management 
* Update application.
* Update device.
* Update group.
* Update policy.
* Update service principal.
* Update StsRefreshTokenValidFrom Timestamp.
* Update user.
* UserLoggedIn
* UserLoginFailed

## Exchange Audit Actions

Possible activities retrieved by the cmdlet **Get-PnPUnifiedAuditLog** for Exchange
* Add-DistributionGroupMember
* AddFolderPermissions
* Add-MailboxLocation
* Add-MailboxPermission
* Add-RecipientPermission   
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
* New-DistributionGroup
* New-DynamicDistributionGroup
* New-ExchangeAssistanceConfig
* New-FolderMoveRequest
* New-Mailbox
* Remove-DistributionGroup
* Remove-DistributionGroupMember
* Remove-DynamicDistributionGroup
* RemoveFolderPermissions
* Remove-InboxRule
* Remove-Mailbox
* Remove-MailboxPermission
* Remove-RecipientPermission
* Remove-StoreMailbox
* Remove-UnifiedGroup
* Send
* SendAs
* SendOnBehalf
* Set-AdminAuditLogConfig
* Set-ConditionalAccessPolicy
* Set-DynamicDistributionGroup
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
* AccessRequestCreated
* Add a partner to cross-tenant access setting.
* Add app role assignment grant to user.
* Add app role assignment to service principal.
* Add delegated permission grant.
* Add device.
* Add group.
* Add member to group.
* Add member to role.
* Add owner to group.
* Add owner to service principal.
* Add registered owner to device.
* Add registered users to device.
* Add service principal credentials.
* Add service principal.
* Add user.
* Add-DistributionGroupMember
* AddedToGroup
* AddedToSecureLink
* AddFolderPermissions
* AddFormCoauthor
* Add-MailboxLocation
* Add-MailboxPermission
* Add-RecipientPermission
* AlertStatusChanged
* AnalyzedByExternalApplication
* AppCatalogSPCorporateCatalogAccessorBaseAdd
* AppDlpEvaluationResultChange
* Authorize
* BindMonikersToDatasources
* Change user license.
* Change user password.
* ClientViewSignaled
* CommentCreated
* CommentsDisabled
* CompanyLinkCreated
* CompanyLinkUsed
* Consent to application.
* Create
* CreateDataset
* CreateFlow
* CreateGatewayClusterDatasource
* CreatePowerApp
* CreateReport
* CreateResponse
* CrmDefaultActivity
* Delete device.
* Delete group.
* Delete user.
* Device no longer compliant.
* Device no longer managed.
* Disable account.
* DownloadReport
* EditFlow
* EditForm
* Enable account.
* Enable-AddressListPaging
* EnableSpecificCollaboaration
* EnableWorkOrSchoolCollaboration
* ExportForm
* FileAccessed
* FileAccessedExtended
* FileCheckedIn
* FileCheckedOut
* FileCheckOutDiscarded
* FileCopied
* FileDeleted
* FileDownloaded
* FileModified
* FileModifiedExtended    
* FileMoved
* FilePreviewed
* FileRecycled
* FileRenamed
* FileRestored
* FileSensitivityLabelApplied
* FileSyncDownloadedFull  
* FileSyncUploadedFull
* FileUploaded  
* FileVersionsAllDeleted
* FolderBind
* FolderCopied
* FolderCreated
* FolderModified
* FolderMoved
* FolderRecycled
* FolderRenamed
* GenerateCustomVisualAADAccessToken
* GenerateScreenshot
* GetAllGatewayClusterDatasources
* GetGatewayClusters
* GetGatewayClusterSupportedDatasources
* GetGatewayRegions
* GetSnapshots
* GroupAdded
* GroupUpdated
* HardDelete
* HubSiteJoined
* HubSiteRegistered
* InitiateGatewayClusterOAuthLogin
* Install-AdminAuditLogConfig
* Install-DataClassificationConfig
* Install-DefaultSharingPolicy
* Install-ResourceConfig
* LaunchPowerApp
* ListColumnCreated
* ListColumnUpdated
* ListContentTypeUpdated
* ListCreated
* ListForms
* ListIndexCreated
* ListItemCreated
* ListItemDeleted
* ListItemRecycled
* ListItemUpdated     
* ListItemViewed
* ListUpdated
* ListViewCreated
* ListViewed
* ListViewUpdated
* MailItemsAccessed
* ManagedSyncClientAllowed
* MarkedMessageChanged
* MDCAssessments
* MDCRegulatoryComplianceAssessments
* MeetingDetail
* MeetingParticipantDetail
* MemberAdded
* MemberRemoved
* MemberRoleChanged
* MessageCreatedHasLink
* MessageCreation
* MessageEditedHasLink
* MessageReadReceiptReceived
* MessageUpdated
* MipLabel
* ModifyFolderPermissions
* Move
* MoveToDeletedItems
* New-DistributionGroup
* New-DynamicDistributionGroup
* New-ExchangeAssistanceConfig
* New-FolderMoveRequest
* New-Mailbox
* PagePrefetched
* PageViewed
* PageViewedExtended
* PermissionLevelAdded
* PIMRoleAssigned
* PlanListRead
* PlanModified
* PlanRead
* PortalLoginAuthorization
* PutConnection
* ReactedToMessage
* RefreshDataset
* Remove delegated permission grant.
* Remove member from group.
* Remove member from role.
* Remove owner from group.
* Remove registered owner from device.
* Remove registered users from device.
* Remove service principal credentials.
* Remove service principal.
* RemovedFromGroup
* Remove-DistributionGroup
* Remove-DistributionGroupMember
* Remove-DynamicDistributionGroup
* RemoveFolderPermissions
* Remove-InboxRule
* Remove-Mailbox
* Remove-MailboxLocation
* Remove-MailboxPermission
* Remove-RecipientPermission
* Remove-StoreMailbox
* Remove-UnifiedGroup
* Reset user password.
* RespondedWithCustomResponse
* RunEmailSubscription
* Search
* SearchQueryPerformed    
* SecureLinkCreated
* SecureLinkUpdated
* SecureLinkUsed
* Send
* SendAs
* SendOnBehalf
* SensitivityLabelApplied
* SensitivityLabeledFileOpened
* SensitivityLabeledFileRenamed
* SensitivityLabelUpdated
* Set Company Information.
* Set-AdminAuditLogConfig
* Set-ConditionalAccessPolicy
* Set-DynamicDistributionGroup
* Set-ExchangeAssistanceConfig
* Set-Mailbox
* Set-MailboxPlan
* Set-OwaMailboxPolicy
* Set-RecipientEnforcementProvisioningPolicy
* SetScheduledRefresh
* Set-TenantObjectVersion
* Set-TransportConfig
* SharingInheritanceBroken
* SharingInvitationBlocked
* SharingLinkCreated
* SharingLinkUsed
* SharingPolicyChanged
* SharingRevoked
* SharingSet
* ShortcutAdded
* SiteCollectionAdminAdded
* SiteCollectionAdminRemoved
* SiteCollectionCreated
* SiteCollectionQuotaModified
* SiteColumnCreated
* SiteContentTypeCreated
* SiteDeleted
* SiteDesignScheduled
* SiteIBModeChanged
* SiteIBModeSet
* SiteLocksChanged
* SoftDelete
* SPCorporateCatalogAppMetadataDeployWithFeatureDeployment
* TabAdded
* TakeOverDataset
* TaskListRead
* TeamCreated
* TeamsSessionStarted
* TeamsUserSignedOut
* ThreadAccessFailure
* ThreadViewed
* Update
* Update a partner cross-tenant access setting.
* Update agreement.
* Update application – Certificates and secrets management
* Update application.
* Update device.
* Update group.
* Update policy.
* Update service principal.
* Update StsRefreshTokenValidFrom Timestamp.
* Update user.
* UpdateDatasourceCredentials
* UpdateFeaturedTables
* UpdateInboxRules
* UpdatePowerApp
* UserLoggedIn
* UserLoginFailed
* Validate
* ViewedApprovalRequest
* ViewForm
* ViewReport
* ViewResponses
* ViewRuntimeForm
* WebMembersCanShareModified
* WebRequestAccessModified

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

## References

[Report of SharePoint Files Incidents - PowerShell Script](https://michalkornet.com/2023/09/19/Report-of-SharePoint-Files-Incidents.html)

[Enumerate activities (preview)](https://learn.microsoft.com/en-us/onedrive/developer/rest-api/api/activities_list?view=odsp-graph-online?wt.mc_id=MVP_308367)

[ItemActivity resource type](https://learn.microsoft.com/en-us/onedrive/developer/rest-api/resources/itemactivity?view=odsp-graph-online?wt.mc_id=MVP_308367)

[Get-PnPUnifiedAuditLog](https://pnp.github.io/powershell/cmdlets/Get-PnPUnifiedAuditLog?wt.mc_id=MVP_308367)

[Use a PowerShell script to search the audit log](https://learn.microsoft.com/en-us/purview/audit-log-search-script?wt.mc_id=MVP_308367)
