---
title: "Oversight of Sharing Links in SharePoint sites using PowerShell"
date: 2024-04-26T06:57:18+01:00
tags: ["SharePoint","SharingLinks","PowerShell","Sites", "Security","Copilot for M365", "Governance","CSOM"]
featured_image: '/posts/images/powershell-get-sharing-links-sharepoint/report.png'
draft: false
---

# Oversight of Sharing Links in SharePoint sites using PowerShell

Effective oversight of sharing links is paramount to ensuring data security, compliance, and optimal collaboration experiences. As organizations migrate to M365 environments, they inherit powerful collaboration tools that facilitate seamless sharing of documents and resources. However, without proper governance, these capabilities can lead to unintended consequences such as data breaches, compliance violations, and loss of intellectual property.

![ManageLiks](../images/powershell-sharePoint-sharing-permissions-copilot/ManageLinks.png)

For Copilot for M365 implementations, ensuring there is no oversharing is a critical aspect of safeguarding sensitive information and maintaining regulatory compliance. By integrating the sharing link audit process into their deployment strategies, administrators can preemptively address security vulnerabilities and uphold the integrity of M365 environments. Continuous monitoring and optimisation allows to harness the full potential of M365 collaboration tools while safeguarding against unauthorised access and data leaks.

However, manually tracking down these links across multiple sites and libraries can be a daunting task. There are few options available, each with its own limitations.

1. [Report on file and folder sharing in a SharePoint site](https://learn.microsoft.com/nl-nl/sharepoint/sharing-reports?wt.mc_id=MVP_308367) allows to report on external sharing per site only.
 
2. [Data access governance reports for SharePoint sites](https://learn.microsoft.com/en-us/sharepoint/data-access-governance-reports?wt.mc_id=MVP_308367) provide a very high level view of the sharing links without details on which folder or item the sharing link was created.

3. [Use sharing auditing in the audit log](https://learn.microsoft.com/en-us/purview/audit-log-sharing?view=o365-worldwide&tabs=microsoft-purview-portal#how-to-identify-resources-shared-with-external-users?wt.mc_id=MVP_308367) is restricted to the filter criteria used, which may not retrieve all sharing links.

In this blog post, we'll explore a PowerShell script that automates the process of retrieving sharing links in SharePoint Online and on which file or folder it was created on, empowering administrators to efficiently audit and manage permissions. The script was adapted from 
[How to get a list of shared links in a SharePoint Online document library? Any PowerShell or other way?](https://learn.microsoft.com/en-us/answers/questions/992330/how-to-get-a-list-of-shared-links-in-a-sharepoint) using the CSOM method GetObjectSharingInformation

## PowerShell Script overview

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
                  $ObjectType = $item.FileSystemObjectType; 
                  $Name  = $Item.FieldValues["FileLeafRef"]           
                  $RelativeURL = $Item.FieldValues["FileRef"]
                }
                else
                {
                    $ObjectType = $_type;
                    $Name = "";
                    $RelativeURL = $listUrl ?? $SiteUrl;
                }
                $result = New-Object PSObject -property $([ordered]@{
                    SiteUrl = $SiteURL
                    listUrl = $listUrl
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

The provided PowerShell script facilitates the retrieval of sharing links across SharePoint sites and libraries. Let's delve into the key components and functionalities of the script:

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

## Conclusion

This PowerShell script can help with compliance or simply optimise security practices for effective SharePoint management.

## References

[How to get a list of shared links in a SharePoint Online document library? Any PowerShell or other way?](https://learn.microsoft.com/en-us/answers/questions/992330/how-to-get-a-list-of-shared-links-in-a-sharepoint?wt.mc_id=MVP_308367)

[How to check links shared in SharePoint Online or OneDrive for Business?](https://erik365.blog/2023/03/16/how-to-check-links-shared-in-sharepoint-online-or-onedrive-for-business/#:~:text=In%20the%20SharePoint%20Online%20report,on%20your%20Microsoft%20365%20users)

[Data access governance reports for SharePoint sites](https://learn.microsoft.com/en-us/sharepoint/data-access-governance-reports?wt.mc_id=MVP_308367)

[Use sharing auditing in the audit log](https://learn.microsoft.com/en-us/purview/audit-log-sharing?view=o365-worldwide&tabs=microsoft-purview-portal#how-to-identify-resources-shared-with-external-users?wt.mc_id=MVP_308367) 