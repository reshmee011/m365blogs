---
title: "PowerShell Script to Query Unique Permissions in SharePoint"
date: 2024-04-28T22:31:33+01:00
tags: ["SharePoint","Unique Permissions","SharingLinks","PowerShell","Sites", "Security","Copilot for M365", "Governance","CSOM"]
featured_image: '/posts/images/powershell-query-unique-permissions-sharepoint/report.png'
draft: false
---

# Query Unique Permissions in SharePoint using CSOM and PnP PowerShell

Managing permissions in SharePoint is a critical aspect of maintaining data security and compliance within organisations. However, as SharePoint environments grow in complexity, manually auditing and managing permissions becomes increasingly challenging. To address this challenge, PowerShell scripts can be leveraged to automate the auditing process, providing administrators with valuable insights into permission structures across SharePoint sites and libraries.

For Copilot for M365 implementations, ensuring there is no oversharing is a critical aspect of safeguarding sensitive information and maintaining regulatory compliance. By integrating the unique permissions audit process, administrators can preemptively address security vulnerabilities and uphold the integrity of M365 environments. Continuous monitoring and optimisation allows to harness the full potential of M365 collaboration tools while safeguarding against unauthorised access and data leaks.

## Introduction

In this blog post, we'll explore a PowerShell script designed to automate the auditing of SharePoint permissions. This script facilitates the retrieval of unique permission assignments and sharing links for sites, lists, and items within a SharePoint environment. By executing this script, administrators can generate comprehensive reports detailing permission assignments, enabling them to identify potential security risks and ensure compliance with organizational policies.

![ManageLinks](../images/powershell-query-unique-permissions-sharepoint/UniquePermissions.png)

I have not been able to determine why unique permissions are sometimes created without generating a sharing link. One of the scenerios I noticed is the sharing link is only created when "Copy Link" is clicked on.

![CopyLinks](../images/powershell-get-sharing-links-sharepoint/linkcopied.png)

## PowerShell script Overview

{{< gist reshmee011 c23123e5f1abedd1085876279ac17b7f >}}

```powershell

Clear-Host

$properties=@{SiteUrl='';SiteTitle='';ListTitle='';Type='';RelativeUrl='';ParentGroup='';MemberType='';MemberName='';MemberLoginName='';Roles='';}; 
 
$dateTime = (Get-Date).toString("dd-MM-yyyy-hh-ss")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$excludeLimitedAccess = $true; # update to $false to include limited access permissions
$includeListsItems = $true; # update to $false to exclude list items/files 

$SiteCollectionUrl = Read-Host -Prompt "Enter site collection URL ";
$global:siteTitle= "";
#Exclude certain libraries
$ExcludedLibraries = @("Form Templates", "Preservation Hold Library", "Site Assets", "Images", "Pages", "Settings", "Videos","Timesheet"
  "Site Collection Documents", "Site Collection Images", "Style Library", "AppPages", "Apps for SharePoint", "Apps for Office")

$global:permissions =@();
$global:sharingLinks = @();

Function PermissionObject($_object,$_type,$_relativeUrl,$_siteUrl,$_siteTitle,$_listTitle,$_memberType,$_parentGroup,$_memberName,$_memberLoginName,$_roleDefinitionBindings)
{
  $permission = New-Object -TypeName PSObject -Property $properties; 
  $permission.SiteUrl =$_siteUrl; 
  $permission.SiteTitle = $_siteTitle; 
  $permission.ListTitle = $_listTitle; 
  $permission.Type = $_type; 
  $permission.RelativeUrl = $_relativeUrl; 
  $permission.MemberType = $_memberType; 
  $permission.ParentGroup = $_parentGroup; 
  $permission.MemberName = $_memberName; 
  $permission.MemberLoginName = $_memberLoginName; 
  $permission.Roles = $_roleDefinitionBindings -join ","; 
  $global:permissions += $permission;
}

Function Extract-Guid ($inputString) {
  $splitString = $inputString -split '\|'
  return $splitString[2].TrimEnd('_o')
}

Function QueryUniquePermissionsByObject($_web,$_object,$_Type,$_RelativeUrl,$_siteUrl,$_siteTitle,$_listTitle)
{
  $roleAssignments = Get-PnPProperty -ClientObject $_object -Property RoleAssignments
  
  foreach($roleAssign in $roleAssignments){
    Get-PnPProperty -ClientObject $roleAssign -Property RoleDefinitionBindings,Member;
    $PermissionLevels = $roleAssign.RoleDefinitionBindings | Select -ExpandProperty Name;
    #Get all permission levels assigned (Excluding:Limited Access)  
    if($excludeLimitedAccess -eq $true){
       $PermissionLevels = ($PermissionLevels | Where { $_ -ne "Limited Access"}) -join ","  
    }
    $Users = Get-PnPProperty -ClientObject ($roleAssign.Member) -Property Users -ErrorAction SilentlyContinue
    #Get Access type
    $AccessType = $roleAssign.RoleDefinitionBindings.Name
    $MemberType = $roleAssign.Member.GetType().Name; 
    #Get the Principal Type: User, SP Group, AD Group  
    $PermissionType = $roleAssign.Member.PrincipalType  
    If($PermissionLevels.Length -gt 0) {
      $MemberType = $roleAssign.Member.GetType().Name; 
       #Sharing link is in the format SharingLinks.03012675-2057-4d1d-91e0-8e3b176edd94.OrganizationView.20d346d3-d359-453b-900c-633c1551ccaa
        If ($roleAssign.Member.Title -like "SharingLinks*")
        {
            If ($Users)
            {
                ForEach ($User in $Users)
                {
                PermissionObject $_object $_Type $_RelativeUrl $_siteUrl $_siteTitle $_listTitle "Sharing Links" $roleAssign.Member.LoginName  $user.Title $User.LoginName  $AccessType; 
                }
            }      
        }
      ElseIf($MemberType -eq "Group" -or $MemberType -eq "User")
      { 
        $MemberName = $roleAssign.Member.Title; 
        $MemberLoginName = $roleAssign.Member.LoginName;    
        if($MemberType -eq "User")
        {
          $ParentGroup = "NA";
        }
        else
        {
          $ParentGroup = $MemberName;
        }
        (PermissionObject $_object $_Type $_RelativeUrl $_siteUrl $_siteTitle $_listTitle $MemberType $ParentGroup $MemberName $MemberLoginName $PermissionLevels); 
      }

      if($_Type  -eq "Site" -and $MemberType -eq "Group")
      {
        If($PermissionType -eq "SharePointGroup")  {  
          #Get Group Members  
          $groupUsers = Get-PnPGroupMember -Identity $roleAssign.Member.LoginName                  
          $groupUsers|foreach-object{ 
            if ($_.LoginName.StartsWith("c:0o.c|federateddirectoryclaimprovider|") -and $_.LoginName.EndsWith("_0")) {
              $guid = Extract-Guid $_.LoginName
              
              Get-PnPMicrosoft365GroupOwners -Identity $guid | ForEach-Object {
                $user = $_
                (PermissionObject $_object "Site" $_RelativeUrl $_siteUrl $_siteTitle "" "GroupMember" $roleAssign.Member.LoginName $user.DisplayName $user.UserPrincipalName $PermissionLevels); 
              }
            }
            elseif ($_.LoginName.StartsWith("c:0o.c|federateddirectoryclaimprovider|")) {
              $guid = Extract-Guid $_.LoginName
              
              Get-PnPMicrosoft365GroupMembers -Identity $guid | ForEach-Object {
                $user = $_
                (PermissionObject $_object "Site" $_RelativeUrl $_siteUrl $_siteTitle "" "GroupMember" $roleAssign.Member.LoginName $user.DisplayName $user.UserPrincipalName $PermissionLevels); 
              }
            }

            (PermissionObject $_object "Site" $_RelativeUrl $_siteUrl $_siteTitle "" "GroupMember" $roleAssign.Member.LoginName $_.Title $_.LoginName $PermissionLevels);   
          }
        }
      } 
    }      
  }
}
Function QueryUniquePermissions($_web)
{
  ##query list, files and items unique permissions
  Write-Host "Querying web $($_web.Title)";
  $siteUrl = $_web.Url; 
 
  Write-Host $siteUrl -Foregroundcolor "Red"; 
  $global:siteTitle = $_web.Title; 
  $ll = Get-PnPList -Includes BaseType, Hidden, Title,HasUniqueRoleAssignments,RootFolder  -Connection $siteconn | Where-Object {$_.Hidden -eq $False -and $_.Title -notin $ExcludedLibraries } #$_.BaseType -eq "DocumentLibrary" 
  Write-Host "Number of lists $($ll.Count)";

  QueryUniquePermissionsByObject $_web $_web "Site" "" $siteUrl $siteTitle  "";
 
  foreach($list in $ll)
  {      
    $listUrl = $list.RootFolder.ServerRelativeUrl; 
    #Exclude internal system lists and check if it has unique permissions 
    if($list.Hidden -ne $True)
    { 
      Write-Host $list.Title  -Foregroundcolor "Yellow"; 
      $listTitle = $list.Title; 
      #Check List Permissions 
      if($list.HasUniqueRoleAssignments -eq $True)
      { 
        $Type = $list.BaseType.ToString(); 
        QueryUniquePermissionsByObject $_web $list $Type $listUrl $siteUrl $siteTitle $listTitle;
      }
      
      if($includeListsItems){         
        $collListItem = Get-PnPListItem -PageSize 2000 -List $list
        $count = $collListItem.Count
        Write-Host  "Number of items : $count within list $listTitle" 
        foreach($item in $collListItem) 
        {
          Get-PnPProperty -ClientObject $item -Property File,HasUniqueRoleAssignments; 
          if($item.HasUniqueRoleAssignments -eq $True)
          { 
            if($list.BaseType -eq "DocumentLibrary")
            {
              $Type = $item.FileSystemObjectType; 
              $fileUrl = $item.FieldValues.FileRef;  
            }
            else
            {
              $Type = "item"
              $fileUrl = "$siteurl/lists/$listTitle/AllItems.aspx?FilterField1=ID&FilterValue1=$($item.id)"
            }
            QueryUniquePermissionsByObject $_web $item $Type $fileUrl $siteUrl $siteTitle $listTitle;
          } 
        } 
      }
    }
  }
}

if(Test-Path $directorypath){
 
  Connect-PnPOnline -Url $SiteCollectionUrl -Interactive
  #array storing permissions
  $web = Get-PnPWeb
  #root web , i.e. site collection level
  QueryUniquePermissions($web);

  Write-Host "Permission count: $($global:permissions.Count)";
  $exportFilePath = Join-Path -Path $directorypath -ChildPath $([string]::Concat($siteTitle,"-Permissions_",$dateTime,".csv"));
  
  Write-Host "Export File Path is:" $exportFilePath
  Write-Host "Number of lines exported is :" $global:permissions.Count
 
  $global:permissions | Select-Object SiteUrl,SiteTitle,Type,RelativeUrl,ListTitle,MemberType,MemberName,MemberLoginName,ParentGroup,Roles|Export-CSV -Path $exportFilePath -NoTypeInformation;
  
}
else{
  Write-Host "Invalid directory path:" $directorypath -ForegroundColor "Red";
}
```

### Script Overview

Let's break down the key components of the PowerShell script:

### Parameters

The script begins by defining parameters such as $tenantUrl, $dateTime, and $directorypath. These parameters allow users to specify the SharePoint site collection URL, generate unique file names for reports, and specify the directory path for exporting reports.

Also two additional parameters are available  $excludeLimitedAccess and $includeListsItems to change behaviour of the script as described in the comments

```powershell
    $excludeLimitedAccess = $true; # update to $false to include limited access permissions
    $includeListsItems = $true; # update to $false to exclude list items/files 
```

### Connection to SharePoint Online Site Collection

Using the Connect-PnPOnline cmdlet, the script establishes a connection to the SharePoint Online site collection. User is prompted to enter the URL of the site collection interactively.

### Retrieval of SharePoint Permissions

The script contains functions (PermissionObject, Extract-Guid, QueryUniquePermissionsByObject, and QueryUniquePermissions) responsible for querying and extracting unique permission assignments for sites, lists, and items within the SharePoint environment. These functions utilise various PnP PowerShell cmdlets to retrieve role assignments, member details, and permission levels.

### Exclusion of Certain Libraries

To streamline the auditing process, the script excludes specified libraries, such as system libraries, using the $ExcludedLibraries array. This ensures that only relevant data is included in the permission audit.

### Exporting the Report

Once the permission data is collected, the script exports it to a CSV file with a timestamped filename. The exported report includes details such as site URL, site title, object type (site, list, or item), relative URL, list title, member type, member name, member login name, parent group, and assigned roles.

### Conclusion

By automating the auditing of SharePoint permissions with PowerShell, organisations can streamline their security and compliance efforts. This script empowers administrators to gain comprehensive insights into permission structures across SharePoint sites and libraries, enabling them to proactively address security vulnerabilities and ensure adherence to regulatory requirements. As SharePoint environments continue to evolve, automation tools like PowerShell play a crucial role in simplifying administrative tasks and enhancing overall governance.
 
## Conclusion

This script provides a comprehensive way to query unique permissions in SharePoint. It can be used to audit permissions, identify security risks, and ensure compliance with organizational policies.

## References

[How to check links shared in SharePoint Online or OneDrive for Business?](https://erik365.blog/2023/03/16/how-to-check-links-shared-in-sharepoint-online-or-onedrive-for-business/#:~:text=In%20the%20SharePoint%20Online%20report,on%20your%20Microsoft%20365%20users)

[CSOM in PowerShell Query All Unique Permissions](https://reshmeeauckloo.wordpress.com/tag/query-unique-permissions-site-webs-and-lists-csom-powershell/)