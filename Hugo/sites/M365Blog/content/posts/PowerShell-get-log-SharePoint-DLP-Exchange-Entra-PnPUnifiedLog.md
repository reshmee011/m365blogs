---
title: "Unveiling Audit Logs with PnP and Cli for M365 PowerShell"
date: 2024-02-09T16:16:09Z
tags: ["SharePoint","PnP","PowerShell","CLI for M365" ,"Audit Logs","DLP", "Exchange", "AzureDirectory"]
featured_image: '/posts/images/PowerShell_PnPUnifiedLog/Sample.png'
omit_header_text: true
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
$startDayInthePast = 7 ## 7 or less with 1 hour margin
$endDay = 0 ##less than startDayInthePast

# Generate a unique log file name using today's date
$dateTime = (Get-Date).toString("dd-MM-yyyy_HHmm")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "logReport-" + $dateTime + ".csv"
$OutPutView = $directorypath + "\Logs\"+ $fileName
 
$logCollection = @()
   $activities += m365 purview auditlog list --contentType SharePoint --startTime ((Get-date).adddays(-$startDayInthePast) | Get-Date -uFormat '%Y-%m-%dT%H:%M:%SZ') --endTime ((Get-date).adddays(-($endDay)) | Get-Date -uFormat '%Y-%m-%dT%H:%M:%SZ') --output 'json' | ConvertFrom-Json
   $activities += m365 purview auditlog list --contentType AzureActiveDirectory --startTime ((Get-date).adddays(-$startDayInthePast) | Get-Date -uFormat '%Y-%m-%dT%H:%M:%SZ') --endTime ((Get-date).adddays(-($endDay)) | Get-Date -uFormat '%Y-%m-%dT%H:%M:%SZ') --output 'json'| ConvertFrom-Json
   $activities += m365 purview auditlog list --contentType DLP  --startTime ((Get-date).adddays(-$startDayInthePast) | Get-Date -uFormat '%Y-%m-%dT%H:%M:%SZ') --endTime ((Get-date).adddays(-($endDay)) | Get-Date -uFormat '%Y-%m-%dT%H:%M:%SZ') --output 'json' | ConvertFrom-Json
   $activities += m365 purview auditlog list --contentType Exchange --startTime ((Get-date).adddays(-$startDayInthePast) | Get-Date -uFormat '%Y-%m-%dT%H:%M:%SZ') --endTime ((Get-date).adddays(-($endDay)) | Get-Date -uFormat '%Y-%m-%dT%H:%M:%SZ') --output 'json' | ConvertFrom-Json
   $activities += m365 purview auditlog list --contentType General --startTime ((Get-date).adddays(-$startDayInthePast) | Get-Date -uFormat '%Y-%m-%dT%H:%M:%SZ') --endTime ((Get-date).adddays(-($endDay)) | Get-Date -uFormat '%Y-%m-%dT%H:%M:%SZ') --output 'json' | ConvertFrom-Json
 
if($activity.SiteUrl ){#-and $activity.SiteUrl
   if($activity.SiteUrl.ToLower() -eq $SiteUrl)    #-$activity.UserId.ToLower() -eq $userId
 {  
    $logCollection += $activity
 }
}

$activities | sort-object "Operation" |Export-CSV $OutPutView -Force -NoTypeInformation
```

> [!Note]
Due to the amount of data retrieved, I faced memory limit reached issue, ideally global admin rights would help to avoid permission issues. 

> [!Note]
> You may encounter error Error: The permission set (ActivityFeed.Read ServiceHealth.Read) sent in the request does not include the expected permission with contentType DLP and be mindful of the amount of data returned from a large tenant which may cause memory issues or lack of disk space to save the log file.

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

## General Notes

### Permissions

Also has issues retrieving some activities related to DLP due lack of permissions which I have been unable to identity.

```PowerShell
ThreadAccessFailure
Error: The permission set (ActivityFeed.Read ServiceHealth.Read) sent in the request does not include the expected permission.
```

 I ran Register-PnPManagementShellAccess [PnP Authentication](https://pnp.github.io/powershell/articles/authentication.html) to ensure all "PnP PowerShell" has the required permissions but it did not resolve the issue. 

I created another app registration.

I amended the app registration for the app used to connect to include the following in the app manifest as it is missing from the PnP PowerShell app registration to resolve the authentication error. These two permissions "ActivityReports.Read" and "ThreatIntelligence.Read" can't be added from the UI yet hence adding it via manifest file. I could not add those using the cmdlet Register-PnPAzureADApp which accepts only parameters [-GraphApplicationPermissions <Permission[]>],[-GraphDelegatePermissions <Permission[]>],[-SharePointApplicationPermissions <Permission[]>] and  [-SharePointDelegatePermissions <Permission[]>] but no **Office 365 Management APIs**. From the UI only the "ActivityFeed.Read", "ActivityFeed.ReadDlp","ServiceHealth.Read" 

![Office 365 Management APIs Permissions](../images/PowerShell_PnPUnifiedLog/Office365ManagementAPIs.png)

```json
{
            "adminConsentDescription": "Allows the application to read service health information for your organization.",
            "adminConsentDisplayName": "Read activity reports for your organization",
            "id": "b3b78c39-cb1d-4d17-820a-25d9196a800e",
            "isEnabled": true,
            "isAdmin": true,
            "consentDescription": "Allows the application to read service health information for your organization.",
            "consentDisplayName": "Read service health information for your organization",
            "value": "ActivityReports.Read"
        },
        {
            "adminConsentDescription": "Allows the application to read threat intelligence data for your organization",
            "adminConsentDisplayName": "Read threat intelligence data for your organization",
            "id": "17f1c501-83cd-414c-9064-cd10f7aef836",
            "isEnabled": true,
            "isAdmin": true,
            "consentDescription": "Allows the application to read threat intelligence data for your organization",
            "consentDisplayName": "Read threat intelligence data for your organization",
            "value": "ThreatIntelligence.Read"
        }
```

![Permissions](../images/PowerShell_PnPUnifiedLog/PermissionsRequired_Activityreports_ThreatIntelligence.png)


```powershell
$app = Register-PnPAzureADApp -ApplicationName "YourApplicationName" -Tenant reshmeeauckloo.onmicrosoft.com -Interactive
#amend permissions to include Office 365 Management APIs (8):ActivityFeed.Read,ActivityFeed.ReadDlp,ActivityReports.Read,ServiceHealth.Read and ThreatIntelligence.Read
#note down the Base64Encoded and client id to connect 
$ClientId = "xxxxxxx"
$base64Encoded = "dddd"
Connect-PnPOnline reshmeeauckloo.sharepoint.com -ClientId $ClientId -Tenant reshmeeauckloo.onmicrosoft.com -CertificateBase64Encoded $base64Encoded
##run above code using pnp powershell version
```

### Date parameters

The date parameters need to be in ISO 8601 string (2021-12-16T18:28:48.6964197Z). Start time cannot be more than 7 days in the past. Default value is 24h ago, otherwise you might receive the following message.

```dotnetcli
Get-PnPUnifiedAuditLog: Call to Microsoft Graph URL https://manage.office.com/api/v1.0/d872ec63-6bea-4678-9429-078f4fa93560/activity/feed/subscriptions/content?contentType=Audit.SharePoint&PublisherIdentifier=$d872ec63-6bea-4678-9429-078f4fa93560&startTime=2024-02-09T11:03:27&endTime=2024-02-16T11:03:27 failed with status code BadRequest: Start time and end time must both be specified (or both omitted) and must be less than or equal to 24 hours apart, with the start time prior to  end time and start time no more than 7 days in the past. StartTime:2024-02-09T11:03:27, EndTime:2024-02-16T11:03:27
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

* AppCleanedUpAfterExpiration
* AppDlpEvaluationResultChange
* BindMonikersToDatasources
* CreateGatewayClusterDatasource
* CreatePowerApp
* CreateResponse
* CrmDefaultActivity
* DownloadReport
* ExportForm
* GenerateCustomVisualAADAccessToken
* GenerateScreenshot  
* GetAllGatewayClusterDatasources
* GetGatewayRegions
* GetSnapshots
* LaunchPowerApp
* ListForms
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
* PortalLoginAuthorization
* ReactedToMessage
* RunEmailSubscription
* Search
* SensitivityLabelApplied
* SensitivityLabeledFileOpened
* SensitivityLabeledFileRenamed
* SensitivityLabelRemoved
* SensitivityLabelUpdated
* TakeOverDataset
* TeamsSessionStarted
* TeamsUserSignedOut
* ThreadViewed
* TIMailData
* UpdateDatasourceCredentials
* UpdatePowerApp
* Validate
* ViewForm
* ViewReport
* ViewResponses
* ViewRuntimeForm

## References

[Report of SharePoint Files Incidents - PowerShell Script](https://michalkornet.com/2023/09/19/Report-of-SharePoint-Files-Incidents.html)

[Enumerate activities (preview)](https://learn.microsoft.com/en-us/onedrive/developer/rest-api/api/activities_list?view=odsp-graph-online?wt.mc_id=MVP_308367)

[ItemActivity resource type](https://learn.microsoft.com/en-us/onedrive/developer/rest-api/resources/itemactivity?view=odsp-graph-online?wt.mc_id=MVP_308367)

[Get-PnPUnifiedAuditLog](https://pnp.github.io/powershell/cmdlets/Get-PnPUnifiedAuditLog?wt.mc_id=MVP_308367)

[Use a PowerShell script to search the audit log](https://learn.microsoft.com/en-us/purview/audit-log-search-script?wt.mc_id=MVP_308367)

[Microsoft Office 365 Audit](https://hstechdocs.helpsystems.com/manuals/corects/eventmanager/current/gtthelp/content/resources/microsoft_office_365_audit.htm)

[CLI for M365 m365 purview audit log](https://pnp.github.io/cli-microsoft365/cmd/purview/auditlog/auditlog-list/)