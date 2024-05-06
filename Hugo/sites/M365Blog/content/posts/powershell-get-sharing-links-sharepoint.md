---
title: "Oversight of Sharing Links in SharePoint sites using PowerShell"
date: 2024-04-26T06:57:18+01:00
tags: ["SharePoint","SharingLinks","PowerShell","Sites", "Security","Copilot for M365", "Governance","CSOM","REST", "PnP","Microsoft Graph" ]
featured_image: '/posts/images/powershell-get-sharing-links-sharepoint/report.png'
draft: false
---

# Oversight of Sharing Links and Sharing Informationin SharePoint sites using PowerShell

Effective oversight of sharing links and sharing information are paramount to ensuring data security, compliance, and optimal collaboration experiences. 

As organisations migrate to M365 environments, they inherit powerful collaboration tools that facilitate seamless sharing of documents and resources. However, without proper governance, these capabilities can lead to unintended consequences such as data breaches, compliance violations, and loss of intellectual property.

Sharing is a powerful feature for collaboration. However depending on how items, files or folders are shared, a sharing link might be created or unique permissions on these items are created.
![ManageLinks](../images/powershell-get-sharing-links-sharepoint/ManageLinks.png)

For Copilot for M365 implementations, ensuring there is no oversharing is a critical aspect of safeguarding sensitive information and maintaining regulatory compliance.

An extract from [Announcing SharePoint advanced management innovations for the AI and Copilot era](https://techcommunity.microsoft.com/t5/sharepoint-premium-blog/announcing-sharepoint-advanced-management-innovations-for-the-ai/ba-p/4126366?WT.mc_id=5005104&ck_subscriber_id=2673998245)

"With Copilot and AI, security has become a concern. Not because Copilot allows people to access anything more than they could previously; it just allows them to find information they have access to faster. A term used sometimes in SharePoint is "Security by obscurity"; hide stuff and hope people don't find it. That doesn't work as well anymore with Copilot. It surfaces data more broadly and quickly."

Refer to [Microsoft Copilot for Microsoft 365 - best practices with SharePoint](https://learn.microsoft.com/en-us/SharePoint/sharepoint-copilot-best-practices?wt.mc_id=MVP_308367).

However, manually tracking down these links across multiple sites and libraries can be a daunting task. There are few options available, each with its own limitations.

1. [Report on file and folder sharing in a SharePoint site](https://learn.microsoft.com/nl-nl/sharepoint/sharing-reports?wt.mc_id=MVP_308367) allows to report on external sharing per site only.
 
2. [Data access governance reports for SharePoint sites](https://learn.microsoft.com/en-us/sharepoint/data-access-governance-reports?wt.mc_id=MVP_308367) provide a very high level view of the sharing links without details on which folder or item the sharing link was created. At the time of writing this blog post in April/May 2024, Data Access Governance reports show new sharing links in the past 28 days, which makes it very difficult to find content that was shared using an Everyone Except External Users or Anyone links more than a month ago.

3. [Use sharing auditing in the audit log](https://learn.microsoft.com/en-us/purview/audit-log-sharing?view=o365-worldwide&tabs=microsoft-purview-portal#how-to-identify-resources-shared-with-external-users?wt.mc_id=MVP_308367) is restricted to the filter criteria used, which may not retrieve all sharing links.

In this blog post, we'll explore a PowerShell script that automates the process of retrieving sharing links in SharePoint Online and on which file or folder it was created on, empowering administrators to efficiently audit and manage permissions. The script was adapted from 
[How to get a list of shared links in a SharePoint Online document library? Any PowerShell or other way?](https://learn.microsoft.com/en-us/answers/questions/992330/how-to-get-a-list-of-shared-links-in-a-sharepoint) using the CSOM method GetObjectSharingInformation


I have not been able to determine why unique permissions are sometimes created without generating a sharing link. One of the scenerios I noticed is the sharing link is only created when "Copy Link" is clicked on.

![CopyLinks](../images/powershell-get-sharing-links-sharepoint/linkcopied.png)

There have been changes to sharing as per MC706173, please refer for more info[M365 Changelog: (Updated) Invite people you choose in the Share control in Microsoft 365 apps](https://petri.com/microsoft-changelog/m365-changelog-invite-people-you-choose-in-the-share-control-in-microsoft-365-apps/) though the full changes are not clear.

So reporting on sharing links might not be enough and look into drilling into unique permissions applied to each file, folder or item. I have provided two flavours of scripts using CSOM, PnP cmdlets and REST API

## PowerShell Script overview using the CSOM method GetObjectSharingInformation

```PowerShell
#Parameters
$tenantUrl = Read-Host -Prompt "Enter tenant collection URL";
$dateTime = (Get-Date).toString("dd-MM-yyyy-hh-ss")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "SharedLinks-" + $dateTime + ".csv"
$ReportOutput = $directorypath + "\Logs\"+ $fileName

#Connect to PnP Online
Connect-PnPOnline -Url $tenantUrl -Interactive

$global:Results = @();

function getSharingLink($_ctx,$_object,$_type,$_siteUrl,$_listUrl)
{
    $SharingInfo = [Microsoft.SharePoint.Client.ObjectSharingInformation]::GetObjectSharingInformation($_ctx, $_object, $false, $false, $false, $true, $true, $true, $true)
        $ctx.Load($SharingInfo)
        $ctx.ExecuteQuery()
        ForEach($ShareLink in $SharingInfo.SharingLinks)
        {
            If($ShareLink.Url)
            {           
                If($ShareLink.IsEditLink)
                {
                    $AccessType="Edit"
                }
                ElseIf($shareLink.IsReviewLink)
                {
                    $AccessType="Review"
                }
                Else
                {
                    $AccessType="ViewOnly"
                }
                #Collect the data

                if($_type -eq "File")
                {
                  $ObjectType = $_object.FileSystemObjectType; 
                  $Name  = $_object.FieldValues["FileLeafRef"]           
                  $RelativeURL = $_object.FieldValues["FileRef"]
                }
                else
                {
                    $ObjectType = $_type;
                    $Name = "";
                    $RelativeURL = $listUrl ?? $SiteUrl;
                }
                $result = New-Object PSObject -property $([ordered]@{
                    SiteUrl = $_siteURL
                    listUrl = $_listUrl
                    Name  = $Name           
                    RelativeURL = $RelativeURL
                    ObjectType = $ObjectType
                    ShareLink  = $ShareLink.Url
                    ShareLinkAccess  =  $AccessType
                    HasExternalGuestInvitees = $ShareLink.HasExternalGuestInvitees
                    ShareLinkType  = $ShareLink.LinkKind
                    AllowsAnonymousAccess  = $ShareLink.AllowsAnonymousAccess
                    IsActive  = $ShareLink.IsActive
                    Expiration = $ShareLink.Expiration
                })
                $global:Results +=$result;
            }
        }
}

#Exclude certain libraries
$ExcludedLists = @("Form Templates", "Preservation Hold Library", "Site Assets", "Images", "Pages", "Settings", "Videos","Timesheet"
    "Site Collection Documents", "Site Collection Images", "Style Library", "AppPages", "Apps for SharePoint", "Apps for Office")

$m365Sites = Get-PnPTenantSite| Where-Object { ( $_.Url -like '*/sites/*') -and $_.Template -ne 'RedirectSite#0' } 
$m365Sites | ForEach-Object {
$siteUrl = $_.Url;     
Connect-PnPOnline -Url $siteUrl -Interactive
$ctx = Get-PnPContext
$web= Get-PnPWeb

Write-Host "Processing site $siteUrl"  -Foregroundcolor "Red"; 

#getSharingLink $ctx $web "site" $siteUrl "";
$ll = Get-PnPList -Includes BaseType, Hidden, Title,HasUniqueRoleAssignments,RootFolder | Where-Object {$_.Hidden -eq $False -and $_.Title -notin $ExcludedLists } #$_.BaseType -eq "DocumentLibrary" 
  Write-Host "Number of lists $($ll.Count)";

  foreach($list in $ll)
  {
    $listUrl = $list.RootFolder.ServerRelativeUrl;       

    #Get all list items in batches
    $ListItems = Get-PnPListItem -List $list -PageSize 2000 
 #   getSharingLink $ctx $list "list/library" $siteUrl $listUrl;
        #Iterate through each list item
        ForEach($item in $ListItems)
        {
            $ItemCount = $ListItems.Count
            #Check if the Item has unique permissions
            $HasUniquePermissions = Get-PnPProperty -ClientObject $Item -Property "HasUniqueRoleAssignments"
            If($HasUniquePermissions)
            {       
                #Get Shared Links
                if($list.BaseType -eq "DocumentLibrary")
                {
                    $type= "File";
                }
                else
                {
                    $type= "Item";
                }
                getSharingLink $ctx $item $type $siteUrl $listUrl;
            }
        }
    }
 }
$global:Results | Export-CSV $ReportOutput -NoTypeInformation
Write-host -f Green "Sharing Links Report Generated Successfully!"
```

### Parameters

$tenantUrl: Input parameter to specify the URL of the SharePoint tenant collection.
$dateTime: Current date and time in a specific format for generating unique file names.
$directorypath: Path to the directory containing the script.
$fileName: Name of the CSV file for storing the sharing links report.

### Connection to SharePoint Online

The script connects to the SharePoint Online environment using the Connect-PnPOnline cmdlet, prompting the user to enter the tenant collection URL interactively.

### Retrieval of Sharing Links

The getSharingLink function retrieves sharing links for a given object (site or list item) using the ObjectSharingInformation class. It collects relevant information such as the URL of the shared link, access type, expiration date, and more.

### Exclusion of Certain Libraries

The script excludes specified libraries (e.g., system libraries) from the sharing links audit using the $ExcludedLists array.

### Iteration through Sites and Libraries

Using Get-PnPTenantSite, the script retrieves a list of SharePoint sites matching predefined criteria (e.g., Communication sites, Teams sites). It then iterates through each site and its document libraries to gather sharing link information.

### Exporting the Report

Finally, the collected sharing link data is exported to a CSV file with a timestamped filename in the designated directory.

## Using REST EndPoint

The REST endpoint that can be used to return only sharing links, refer to [Externally Sharing – GetSharingInformation REST API](https://sharepoint.stackexchange.com/questions/309303/get-all-shared-links-to-a-specific-user-in-tenant)

 (siteUrl)/_api/web/Lists(listid)/GetItemById(itemid)/GetSharingInformation?$Expand=permissionsInformation,pickerSettings"

```PowerShell
#Parameters
$tenantUrl = Read-Host -Prompt "Enter tenant collection URL";
$dateTime = (Get-Date).toString("dd-MM-yyyy-hh-ss")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "SharedLinks-" + $dateTime + ".csv"
$ReportOutput = $directorypath + "\Logs\"+ $fileName

#Connect to PnP Online
Connect-PnPOnline -Url $tenantUrl -Interactive

$global:Results = @();

function getSharingLink($_object,$_type,$_siteUrl,$_list)
{
    $relativeUrl = $_object.FieldValues["FileRef"]
    $sharingSettings = Invoke-PnPSPRestMethod -Method Post -Url "$($_siteUrl)/_api/web/Lists(@a1)/GetItemById(@a2)/GetSharingInformation?@a1='{$($_list.Id)}'&@a2='$($_object.Id)'&`$Expand=permissionsInformation,pickerSettings"  -ContentType "application/json;odata=verbose" -Content "{}"
    
    ForEach ($shareLink in $sharingSettings.permissionsInformation.links) 
    {
        Write-Host "Shared links found in '$relativeUrl'" 
        $linkDetails = $shareLink.linkDetails
        if ($linkDetails.Url) {
        $invitees = (
            $linkDetails.Invitations | 
            ForEach-Object { $_.Invitee.email   }
        ) -join '|'
        $LastModified = Get-Date -Date $linkDetails.LastModified 
        $linkAccess = "ViewOnly"
        if ($linkDetails.IsEditLink) {
            $linkAccess = "Edit"
        }
        elseif ($linkDetails.IsReviewLink) {
            $linkAccess = "Review"
        }
        $sharedLinkObject = New-Object PSObject -property $([ordered]@{ 
            ItemID   = $item.FieldValues["UniqueId"]
            ShareLink = $linkDetails.Url
            Invitees  = $invitees
            Name = $_object.FieldValues["FileLeafRef"] ?? $_object.Title       
            FileType = $_object.FieldValues["File_x0020_Type"] ?? "Item"
            RelativeURL = $_object.FieldValues["FileRef"] ?? ""
            LinkAccess = $linkAccess
            LastModified = $LastModified
            # Add other properties as needed
        })
        $global:Results +=$sharedLinkObject;
    }     
  }
}
    #Exclude system lists
$ExcludedLists = @("Access Requests", "App Packages", "appdata", "appfiles", "Apps in Testing", "Cache Profiles", "Composed Looks", "Content and Structure Reports", "Content type publishing error log", "Converted Forms",
    "Device Channels", "Form Templates", "fpdatasources", "Get started with Apps for Office and SharePoint", "List Template Gallery", "Long Running Operation Status", "Maintenance Log Library", "Images", "site collection images"
    , "Master Docs", "Master Page Gallery", "MicroFeed", "NintexFormXml", "Quick Deploy Items", "Relationships List", "Reusable Content", "Reporting Metadata", "Reporting Templates", "Search Config List", "Site Assets", "Preservation Hold Library",
    "Site Pages", "Solution Gallery", "Style Library", "Suggested Content Browser Locations", "Theme Gallery", "TaxonomyHiddenList", "User Information List", "Web Part Gallery", "wfpub", "wfsvc", "Workflow History", "Workflow Tasks", "Pages")

$m365Sites = Get-PnPTenantSite| Where-Object { ( $_.Url -like '*/sites/*') -and $_.Template -ne 'RedirectSite#0' } 
$m365Sites | ForEach-Object {
$siteUrl = $_.Url;     
Connect-PnPOnline -Url $siteUrl -Interactive

Write-Host "Processing site $siteUrl"  -Foregroundcolor "Red"; 

#getSharingLink $ctx $web "site" $siteUrl "";
$ll = Get-PnPList -Includes BaseType, Hidden, Title,HasUniqueRoleAssignments,RootFolder | Where-Object {$_.Hidden -eq $False -and $_.Title -notin $ExcludedLists } #$_.BaseType -eq "DocumentLibrary" 
  Write-Host "Number of lists $($ll.Count)";

  foreach($list in $ll)
  {       
    #Get all list items in batches
    $ListItems = Get-PnPListItem -List $list -PageSize 2000 
 #   getSharingLink $ctx $list "list/library" $siteUrl $listUrl;
        #Iterate through each list item
        ForEach($item in $ListItems)
        {
            $ItemCount = $ListItems.Count
            #Check if the Item has unique permissions
            $HasUniquePermissions = Get-PnPProperty -ClientObject $Item -Property "HasUniqueRoleAssignments"
            If($HasUniquePermissions)
            {       
                #Get Shared Links
                if($list.BaseType -eq "DocumentLibrary")
                {
                    $type= $item.FileSystemObjectType;
                }
                else
                {
                    $type= "Item";
                }
                getSharingLink $item $type $siteUrl $list;
            }
        }
    }
 }
 
 $global:Results | Export-CSV $ReportOutput -NoTypeInformation
Write-host -f Green "Sharing Links Report Generated Successfully!"
```

### Output of the REST EndPoint call
![RESTOutput](../images/powershell-get-sharing-links-sharepoint/RESTOutput.png)

## Using Get-PnPFileSharingLink and Get-PnPFolderSharingLink

The Get-PnPFileSharingLink and Get-PnPFolderSharingLink returns all sharing links created only at file and folder level. 
![ActualLinks](../images/powershell-get-sharing-links-sharepoint/ActualSharingLinks.png)

There is no cmdlet to return sharing link at item level yet.

The cmdlets Get-PnPFileSharingLink and Get-PnPFolderSharingLink uses the Graph EndPoint

https://graph.microsoft.com/v1.0/sites/{SiteId}/drives/{VroomDriveID}/items/{VroomItemID}/permissions?$filter=Link ne null

```powershell
#Parameters
$tenantUrl = Read-Host -Prompt "Enter tenant collection URL";
$dateTime = (Get-Date).toString("dd-MM-yyyy-hh-ss")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "SharedLinks-" + $dateTime + ".csv"
$ReportOutput = $directorypath + "\Logs\"+ $fileName

#Connect to PnP Online
Connect-PnPOnline -Url $tenantUrl -Interactive

$global:Results = @();

function getSharingLink($_object,$_type,$_siteUrl,$_listUrl)
{
    $relativeUrl = $_object.FieldValues["FileRef"]
    $SharingLinks = if ($_type -eq "File") {
        Get-PnPFileSharingLink -File $relativeUrl
    } elseif ($_type -eq "Folder") {
        Get-PnPFolderSharingLink -Folder $relativeUrl
    }
    
    ForEach($ShareLink in $SharingLinks)
    {
        Write-Host "Shared links found in '$relativeUrl'" 
        $ShareLink | Add-Member -NotePropertyName "SiteUrl" -NotePropertyValue $_siteUrl
        $ShareLink | Add-Member -NotePropertyName "ListUrl" -NotePropertyValue $_listUrl
        $ShareLink | Add-Member -NotePropertyName "ServerRelativeUrl" -NotePropertyValue $relativeUrl
        $ShareLink | Add-Member -NotePropertyName "RoleList" -NotePropertyValue ($ShareLink.Roles -join "|")
        $ShareLink | Add-Member -NotePropertyName "LinkUrl" -NotePropertyValue $ShareLink.Link.WebUrl
        $ShareLink | Add-Member -NotePropertyName "LinkScope" -NotePropertyValue $ShareLink.Link.Scope
        $ShareLink | Add-Member -NotePropertyName "LinkType" -NotePropertyValue $ShareLink.Link.Type
        $ShareLink | Add-Member -NotePropertyName "LinkPreventsDownload" -NotePropertyValue $ShareLink.Link.PreventsDowload
        $ShareLink | Add-Member -NotePropertyName "LinkUsers" -NotePropertyValue ($ShareLink.GrantedToIdentitiesV2.User.Email -join "|")
            
        $global:Results +=$ShareLink;
    }     
}

#Exclude certain libraries
$ExcludedLists = @("Access Requests", "App Packages", "appdata", "appfiles", "Apps in Testing", "Cache Profiles", "Composed Looks", "Content and Structure Reports", "Content type publishing error log", "Converted Forms",
    "Device Channels", "Form Templates", "fpdatasources", "Get started with Apps for Office and SharePoint", "List Template Gallery", "Long Running Operation Status", "Maintenance Log Library", "Images", "site collection images"
    , "Master Docs", "Master Page Gallery", "MicroFeed", "NintexFormXml", "Quick Deploy Items", "Relationships List", "Reusable Content", "Reporting Metadata", "Reporting Templates", "Search Config List", "Site Assets", "Preservation Hold Library",
    "Site Pages", "Solution Gallery", "Style Library", "Suggested Content Browser Locations", "Theme Gallery", "TaxonomyHiddenList", "User Information List", "Web Part Gallery", "wfpub", "wfsvc", "Workflow History", "Workflow Tasks", "Pages")

$m365Sites = Get-PnPTenantSite| Where-Object { ( $_.Url -like '*/sites/*') -and $_.Template -ne 'RedirectSite#0' } 
$m365Sites | ForEach-Object {
$siteUrl = $_.Url;     
Connect-PnPOnline -Url $siteUrl -Interactive

Write-Host "Processing site $siteUrl"  -Foregroundcolor "Red"; 

#getSharingLink $ctx $web "site" $siteUrl "";
$ll = Get-PnPList -Includes BaseType, Hidden, Title,HasUniqueRoleAssignments,RootFolder | Where-Object {$_.Hidden -eq $False -and $_.Title -notin $ExcludedLists } #$_.BaseType -eq "DocumentLibrary" 
  Write-Host "Number of lists $($ll.Count)";

  foreach($list in $ll)
  {
    $listUrl = $list.RootFolder.ServerRelativeUrl;       

    #Get all list items in batches
    $ListItems = Get-PnPListItem -List $list -PageSize 2000 

        ForEach($item in $ListItems)
        {
            $ItemCount = $ListItems.Count
            #Check if the Item has unique permissions
            $HasUniquePermissions = Get-PnPProperty -ClientObject $Item -Property "HasUniqueRoleAssignments"
            If($HasUniquePermissions)
            {       
                #Get Shared Links
                if($list.BaseType -eq "DocumentLibrary")
                {
                    $type= $item.FileSystemObjectType;
                }
                else
                {
                    $type= "Item";
                }
                getSharingLink $item $type $siteUrl $listUrl;
            }
        }
    }
 }
 
 $global:Results | select Id,SiteUrl,ListUrl,ServerRelativeUrl,RoleList,LinkUrl,LinkScope,LinkType,LinkPreventsDownload,LinkUsers,Invitation,ShareId,ExpirationDateTime,HasPassword,HasChanges | ConvertTo-Csv -NoTypeInformation | Out-File $ReportOutput 
  #Export-CSV $ReportOutput -NoTypeInformation
Write-host -f Green "Sharing Links Report Generated Successfully!"
```

However using Get-PnPFileSharingLink and Get-PnPFolderSharingLink return only sharing links and not all sharing instances without a link.

Example output

![PnPOutput](../images/powershell-get-sharing-links-sharepoint/PnP_Output.png)

## Conclusion

This PowerShell script can help with compliance or simply optimise security practices for effective SharePoint management. There are different ways to retrieve sharing information. The CSOM method returns all sharing information while the REST and PnP cmdlets (Graph) return only sharing links.

## References

[Microsoft Copilot for Microsoft 365 - best practices with SharePoint](https://learn.microsoft.com/en-us/SharePoint/sharepoint-copilot-best-practices?wt.mc_id=MVP_308367)

[Microsoft Purview data security and compliance protections for Microsoft Copilot](https://learn.microsoft.com/en-us/purview/ai-microsoft-purview?wt.mc_id=MVP_308367)

[How to get a list of shared links in a SharePoint Online document library? Any PowerShell or other way?](https://learn.microsoft.com/en-us/answers/questions/992330/how-to-get-a-list-of-shared-links-in-a-sharepoint?wt.mc_id=MVP_308367)

[How to check links shared in SharePoint Online or OneDrive for Business?](https://erik365.blog/2023/03/16/how-to-check-links-shared-in-sharepoint-online-or-onedrive-for-business/#:~:text=In%20the%20SharePoint%20Online%20report,on%20your%20Microsoft%20365%20users)

[Data access governance reports for SharePoint sites](https://learn.microsoft.com/en-us/sharepoint/data-access-governance-reports?wt.mc_id=MVP_308367)

[Use sharing auditing in the audit log](https://learn.microsoft.com/en-us/purview/audit-log-sharing?view=o365-worldwide&tabs=microsoft-purview-portal#how-to-identify-resources-shared-with-external-users?wt.mc_id=MVP_308367) 
