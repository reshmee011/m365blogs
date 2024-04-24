---
title: "Powershell Query Unique Permissions Sharepoint"
date: 2024-04-24T22:31:33+01:00
draft: true
---

```Powershell
Clear-Host

$properties=@{SiteUrl='';SiteTitle='';ListTitle='';Type='';RelativeUrl='';ParentGroup='';MemberType='';MemberName='';MemberLoginName='';Roles='';}; 
 
$ExportFileDirectory = (get-location).ToString();

$SiteCollectionUrl = Read-Host -Prompt "Enter site collection URL: ";
$global:siteTitle= "";
#Exclude certain libraries
$ExcludedLibraries = @("Form Templates", "Preservation Hold Library", "Site Assets", "Site Pages", "Images", "Pages", "Settings", "Videos",
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

   ## Write-Host  "Site URL: " $_siteUrl  "Site Title"  $_siteTitle  "List Title" $_istTitle "Member Type" $_memberType "Relative URL" $_RelativeUrl "Member Name" $_memberName "Role Definition" $_roleDefinitionBindings -Foregroundcolor "Green";
    return $permission;
}

Function SharingLinkObject($_object,$_type,$_relativeUrl,$_siteUrl,$_siteTitle,$_listTitle,$_memberName,$_memberLoginName,$_memberEmail,$_accessType)
{
    $sharingLink = New-Object -TypeName PSObject -Property $properties; 
    $sharingLink.SiteUrl =$_siteUrl; 
    $sharingLink.SiteTitle = $_siteTitle; 
    $sharingLink.ListTitle = $_listTitle; 
    $sharingLink.Type = $_type; 
    $sharingLink.RelativeUrl = $_relativeUrl; 
    $sharingLink.MemberName = $_memberName; 
    $sharingLink.MemberLoginName = $_memberLoginName; 
    $sharingLink.Email = $_memberEmail; 
    $sharingLink.Access = $accessType; 

   ## Write-Host  "Site URL: " $_siteUrl  "Site Title"  $_siteTitle  "List Title" $_istTitle "Member Type" $_memberType "Relative URL" $_RelativeUrl "Member Name" $_memberName "Role Definition" $_roleDefinitionBindings -Foregroundcolor "Green";
    return $sharingLink;
}

Function QueryUniquePermissionsByObject($_web,$_object,$_Type,$_RelativeUrl,$_siteUrl,$_siteTitle,$_listTitle)
{
  $roleAssignments = Get-PnPProperty -ClientObject $_object -Property RoleAssignments
  
  foreach($roleAssign in $roleAssignments){
      Get-PnPProperty -ClientObject $roleAssign -Property RoleDefinitionBindings,Member;
      $PermissionLevels = $roleAssign.RoleDefinitionBindings | Select -ExpandProperty Name;
      
      $collGroups = Get-PnPSiteGroup;

   if($MemberType -eq "Group" -or $MemberType -eq "User")
    { 
       # Get-PnPProperty -ClientObject $roleAssign.Member -Property LoginName,Title;    
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
 
        $global:permissions += (PermissionObject $_object $_Type $_RelativeUrl $_siteUrl $_siteTitle $_listTitle $MemberType $ParentGroup $MemberName $MemberLoginName $PermissionLevels); 
     
       if($_Type  -eq "Site" -and $MemberType -eq "Group")
       {
          foreach($group in $collGroups)
          {
            if($group.Title -eq $MemberName)
             {
              $groupUsers = Get-PnPGroupMember -Group $group
               ##Write-Host "Number of users" $group.Users.Count;
              $groupUsers|foreach-object{ 
                $global:permissions += (PermissionObject $_object "Site" $_RelativeUrl $_siteUrl $_siteTitle "" "GroupMember" $group.Title $_.Title $_.LoginName $PermissionLevels); 
                  ##Write-Host  $permissions.Count
                 }
               }
          }
       } 
     }
      
   }
#  return $_permissions;
}


Function QuerySharingLinksByObject($_web,$_object,$_Type,$_RelativeUrl,$_siteUrl,$_siteTitle,$_listTitle)
{
  $roleAssignments = Get-PnPProperty -ClientObject $_object -Property RoleAssignments
  
  foreach($roleAssign in $roleAssignments){
      Get-PnPProperty -ClientObject $roleAssign -Property RoleDefinitionBindings,Member;
       #Get list of users
      $Users = Get-PnPProperty -ClientObject ($roleAssign.Member) -Property Users -ErrorAction SilentlyContinue
      #Get Access type
      $AccessType = $roleAssign.RoleDefinitionBindings.Name
      If ($roleAssign.Member.Title -like "SharingLinks*")
      {
        #$Item.FieldValues["FileLeafRef"]       
        If ($Users -ne $null)
          {
              ForEach ($User in $Users)
              {
                $global:sharingLinks += (SharingLinkObject $_object $_Type $Item.FieldValues["FileRef"] $_siteUrl $_siteTitle $_listTitle $user.Title $User.LoginName $User.Email $AccessType);
              }
            }      
        }
  }
 # return $_sharingLinks;
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
       QueryUniquePermissionsByObject $_web $list $Type $listUrl $siteUrl $siteTitle  $listTitle;
       QuerySharingLinksByObject $_web $list $Type $listUrl $siteUrl $siteTitle  $listTitle;
 
       if($list.BaseType -eq "DocumentLibrary")
       { 
        #todo unique persmissions on Folders
       # $folders = Get-PnPFolderInFolder -Identity $list  -Recurse
         $collListItem = Get-PnPListItem -PageSize 2000 -List $list
         $count = $collListItem.Count
            Write-Host  "Number of items : $count within list $listTitle" 
            foreach($item in $collListItem) 
            {
                Get-PnPProperty -ClientObject $item -Property File,HasUniqueRoleAssignments;
                $fileUrl = $item.FieldValues.FileRef;  
                if($item.HasUniqueRoleAssignments -eq $True)
                { 
                  $Type = $item.FileSystemObjectType; 
                  #$permissions += (
                    QueryUniquePermissionsByObject $_web $item $Type $fileUrl $siteUrl $siteTitle $listTitle;
                    #);
                  #$sharingLinks += (
                    QuerySharingLinksByObject $_web $list $Type $listUrl $siteUrl $siteTitle  $listTitle;
                    #);
                } 
             } 
           } 
         } 
       }
     }
    #return  $permissions;
}

if(Test-Path $ExportFileDirectory){
 
  Connect-PnPOnline -Url $SiteCollectionUrl -Interactive
  #array storing permissions
   #$Permissions = @(); 
   $web = Get-PnPWeb
   #root web , i.e. site collection level
   #$Permissions += 
   QueryUniquePermissions($web);
   Write-Host "Permission count: $($global:permissions.Count)";
   Write-Host "Sharing Link count: $($global:sharingLinks.Count)";

   $todayDateTime = Get-Date -format "yyyyMMMdd_hhmmss"

   $exportFilePath = Join-Path -Path $ExportFileDirectory -ChildPath $([string]::Concat($siteTitle,"-Permissions_",$todayDateTime,".csv"));
  
   Write-Host "Export File Path is:" $exportFilePath
   Write-Host "Number of lines exported is :" $global:permissions.Count
 
   $global:permissions|Select SiteUrl,SiteTitle,Type,RelativeUrl,ListTitle,MemberType,MemberName,MemberLoginName,ParentGroup,Roles|Export-CSV -Path $exportFilePath -NoTypeInformation;
   $exportSFilePath = Join-Path -Path $ExportFileDirectory -ChildPath $([string]::Concat($siteTitle,"-Sharing_",$todayDateTime,".csv"));
   $global:sharingLinks|Select SiteUrl,SiteTitle,Type,RelativeUrl,ListTitle,MemberType,MemberName,MemberLoginName,ParentGroup,Roles|Export-CSV -Path $exportSFilePath -NoTypeInformation;
}
else{
 
Write-Host "Invalid directory path:" $ExportFileDirectory -ForegroundColor "Red";
 
}
```

## Conclusion