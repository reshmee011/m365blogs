---
title: "Powershell Query Unique Permissions Sharepoint"
date: 2024-04-24T22:31:33+01:00
draft: true
---

```Powershell
Clear-Host

$properties=@{SiteUrl='';SiteTitle='';ListTitle='';Type='';RelativeUrl='';ParentGroup='';MemberType='';MemberName='';MemberLoginName='';Roles='';}; 
 
$dateTime = (Get-Date).toString("dd-MM-yyyy-hh-ss")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path

$includeListsItems = $true;

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

   ## Write-Host  "Site URL: " $_siteUrl  "Site Title"  $_siteTitle  "List Title" $_istTitle "Member Type" $_memberType "Relative URL" $_RelativeUrl "Member Name" $_memberName "Role Definition" $_roleDefinitionBindings -Foregroundcolor "Green";
   $global:permissions += $permission;
   #return $global:permissions;
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

      $PermissionLevels = ($PermissionLevels | Where { $_ -ne "Limited Access"}) -join ","  
      #Get the Principal Type: User, SP Group, AD Group  
      $PermissionType = $roleAssign.Member.PrincipalType  
      If($PermissionLevels.Length -gt 0) {
        $MemberType = $roleAssign.Member.GetType().Name; 
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
  
          (PermissionObject $_object $_Type $_RelativeUrl $_siteUrl $_siteTitle $_listTitle $MemberType $ParentGroup $MemberName $MemberLoginName $PermissionLevels); 
        }

        if($_Type  -eq "Site" -and $MemberType -eq "Group")
        {
            If($PermissionType -eq "SharePointGroup")  {  
                #Get Group Members  
                $groupUsers = Get-PnPGroupMember -Identity $roleAssign.Member.LoginName                  
                
                ##Write-Host "Number of users" $group.Users.Count;
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
                    ##Write-Host  $permissions.Count
                  
                }
            }
        } 
      }      
 }
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
      $MemberType = $roleAssign.Member.GetType().Name; 
      $description = $roleAssign.Member.Description;  
      #Sharing link is in the format SharingLinks.03012675-2057-4d1d-91e0-8e3b176edd94.OrganizationView.20d346d3-d359-453b-900c-633c1551ccaa
      If ($roleAssign.Member.Title -like "SharingLinks*")
      {
        #$Item.FieldValues["FileLeafRef"]       
        If ($Users -ne $null)
          {
              ForEach ($User in $Users)
              {
               # PermissionObject $_object $_Type $Item.FieldValues["FileRef"] $_siteUrl $_siteTitle $_listTitle $user.Title $User.LoginName $User.Email $AccessType;
                PermissionObject $_object $_Type $_RelativeUrl $_siteUrl $_siteTitle $_listTitle "Sharing Links" $description  $user.Title $User.LoginName  $AccessType; 
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

  QueryUniquePermissionsByObject $_web $_web "Site" "" $siteUrl $siteTitle  "";
  QuerySharingLinksByObject $_web $_web "Site" "" $siteUrl $siteTitle  "";

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
    }
      
        #todo unique persmissions on Folders
       if($includeListsItems){         $collListItem = Get-PnPListItem -PageSize 2000 -List $list
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
                  #$permissions += (
                    QueryUniquePermissionsByObject $_web $item $Type $fileUrl $siteUrl $siteTitle $listTitle;
                    #);
                  #$sharingLinks += (
                    QuerySharingLinksByObject $_web $item $Type $fileUrl $siteUrl $siteTitle  $listTitle;
        
                    #);
                } 
             } 
           }
          }
     }
    #return  $permissions;
}

if(Test-Path $directorypath){
 
  Connect-PnPOnline -Url $SiteCollectionUrl -Interactive
  #array storing permissions
   #$Permissions = @(); 
   $web = Get-PnPWeb
   #root web , i.e. site collection level
   #$Permissions += 
   QueryUniquePermissions($web);


  #QuerySharingLinksByObject $web "Site Pages" "file" "https://ppfonline.sharepoint.com/sites/Connect-News-BusinessUpdates/SitePages/A-message-from-Kate-Jones.aspx" "https://ppfonline.sharepoint.com/sites/Connect-News-BusinessUpdates" "Connect-News-BusinessUpdates"  "Site Pages";

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

## Conclusion
Sharing files is an immanent collaboration feature. However, to which extend it is used or can be used, especially external sharing, should be up to every organization. There are different requirements, regulations and compliance aspects to keep in mind. These basic views for users and admins are something to start with but depending on more specific requirements this can be insufficient. For IT admins the reports are just a supportive (tiny) part of the compliance, regulation and monitoring puzzle they have to face, in my opinion.

For data governance a holistic approach is required. This short abstract only highlights the three areas, two for users and one for admins to gain insights on shared file via OneDrive for Business and SharePoint Online. These are not the only options, of course.


## References

[How to check links shared in SharePoint Online or OneDrive for Business?](https://erik365.blog/2023/03/16/how-to-check-links-shared-in-sharepoint-online-or-onedrive-for-business/#:~:text=In%20the%20SharePoint%20Online%20report,on%20your%20Microsoft%20365%20users)
