---
title: "Deletion of sharing links with PowerShell"
date: 2024-06-27T07:17:21+01:00
tags: ["PowerShell", "Sharing Links","Governance","Copilot for M365 rollout"]
featured_image: '/posts/images/powershell-delete-sharinglinks/example.png'
draft: false
---

# Deletion of sharing links with PowerShell

As organisations look to deploy Copilot for Microsoft 365, ensuring the security and proper governance of shared content is important. The rollout of Copilot introduces advanced AI capabilities across Microsoft 365 apps using content from SharePoint/OneDrive, making it even more essential to manage sharing links judiciously to tackle the issue of oversharing.

Sharing is a powerful feature for collaboration. However depending on how items, files or folders are shared, a sharing link might be created or unique permissions on these items are created.
![ManageLinks](../images/powershell-get-sharing-links-sharepoint/ManageLinks.png)

The sharing link is created when the **copy links** is clicked from the sharing pop up options when people other those already have existing access are picked.

![CopyLinks](../images/powershell-get-sharing-links-sharepoint/linkcopied.png)

However by default, if sharing options have not been configured, links to "People in <tenant>" is selected

![default sharing link](../images/powershell-delete-sharinglinks/PeopleOrg.png)

The sharing behaviour can be changed to "People with existing access" using PowerShell to prevent accidental sharing not to worsen the situation.

```powershell
Set-PnPTenantSite -Identity https://contoso.sharepoint.com/sites/SharingTest `
            -DefaultLinkPermission None `
            -DefaultLinkToExistingAccess $true `
            -DisableCompanyWideSharingLinks Disabled `
```

Please refer to [Get sharing links](https://reshmeeauckloo.com/posts/powershell-get-sharing-links-sharepoint/) how to retrieve all sharing instances generated.

If sharing links need to be removed from accidental unintended sharing, there are two options.

1. Run a script to remove unique permissions for all files and items. However it will remove unique permissions deliberately set.
[Reset files permissions unique to Inherited](https://pnp.github.io/script-samples/reset-files-permission-unique-to-inherited/README.html?tabs=pnpps)

2. Delete only the sharing links 
The script using PnP PowerShell below helps to delete only the sharing links set at folder, file and item level

```PowerShell
#Parameters
$siteUrl = Read-Host -Prompt "Enter site collection URL";
$dateTime = (Get-Date).toString("dd-MM-yyyy-hh-ss")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "SharingLinkDeletionReport-" + $dateTime + ".csv"
$ReportOutput = $directorypath + "\Logs\"+ $fileName

$global:Results = @();

#Exclude certain libraries
$ExcludedLists = @("Form Templates", "Preservation Hold Library", "Site Assets", "Images", "Pages", "Settings", "Videos","Timesheet"
    "Site Collection Documents", "Site Collection Images", "Style Library", "AppPages", "Apps for SharePoint", "Apps for Office")

Function QueryDeleteSharingLinksByObject($_web,$_object,$_type,$_relativeUrl,$_siteUrl,$_siteTitle,$_listTitle)
{
  $roleAssignments = Get-PnPProperty -ClientObject $_object -Property RoleAssignments
  
  foreach($roleAssign in $roleAssignments){
      Get-PnPProperty -ClientObject $roleAssign -Property RoleDefinitionBindings,Member;
      #Sharing link is in the format SharingLinks.03012675-2057-4d1d-91e0-8e3b176edd94.OrganizationView.20d346d3-d359-453b-900c-633c1551ccaa
    If ($roleAssign.Member.Title -like "SharingLinks*")
      {
        $global:Results += New-Object PSObject -property $([ordered]@{
            object= $_object.Title
            type = $_type          
            relativeURL = $_relativeURL
            siteUrl = $_siteUrl 
            siteTitle = $_siteTitle
            listTitle = $_listTitle 
            sharinglink = $roleAssign.Member.Title
        })
           #$object.RoleAssignments.GetPrincipalId($roleAssign.PrincipalId).Delete
       #Invoke-PnPQuery
       remove-pnpgroup -identity $roleAssign.Member.Title -force
       
    }
 # return $_sharingLinks;
   }
}

  
Connect-PnPOnline -Url $siteUrl -Interactive

$web= Get-PnPWeb

Write-Host "Processing site $siteUrl"  -Foregroundcolor "Red"; 

$ll = Get-PnPList -Includes BaseType, Hidden, Title,HasUniqueRoleAssignments,RootFolder | Where-Object {$_.Hidden -eq $False -and $_.Title -notin $ExcludedLists } #$_.BaseType -eq "DocumentLibrary" 
  Write-Host "Number of lists $($ll.Count)";

  foreach($list in $ll)
  {
    $listUrl = $list.RootFolder.ServerRelativeUrl;       
    $listTitle = $list.Title; 
    #Get all list items in batches
    $ListItems = Get-PnPListItem -List $list -PageSize 2000 
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
                    $fileUrl = $item.FieldValues.FileRef;
                }
                else
                {
                    $type= "Item";
                    $fileUrl = "$siteurl/lists/$listTitle/AllItems.aspx?FilterField1=ID&FilterValue1=$($item.id)"
                }
                QueryDeleteSharingLinksByObject $web $item $Type $fileUrl $siteUrl $web.Title $listTitle;
            }
        }
    }
 
  $global:Results | Export-CSV $ReportOutput -NoTypeInformation
Write-host -f Green "Sharing Links for user generated Successfully!"
```

The PowerShell script provided is designed to connect to a specified SharePoint Online site collection, enumerate all lists and libraries (excluding certain system libraries), and scan for sharing links. These links are then systematically deleted, and a report is generated to log the actions taken. This process helps administrators maintain control over who has access to what information, ensuring that sharing is done judiciously and in compliance with organizational policies.

## Using Remove-PnPFileSharingLink and Remove-PnPFolderSharingLink

The Remove-PnPFileSharingLink and Remove-PnPFolderSharingLink will remove sharing links created only at file and folder level excluding item level. 

## Conclusion

The PowerShell script outlined in this post offers a practical solution for administrators to mitigate the risks associated with oversharing by deleting unnecessary sharing links in SharePoint Online. By automating this process, organisations can ensure that their collaboration practices do not compromise security or compliance. This approach not only helps in maintaining a secure digital environment but also supports the governance policies that are crucial for a successful Copilot for Microsoft 365 rollout.