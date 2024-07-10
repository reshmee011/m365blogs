---
title: "Effective permissions of end user using across sites PowerShell"
date: 2024-07-10T16:16:09Z
tags: ["SharePoint","PnP","PowerShell","Effective permissions" ,"Security","Permissions"]
featured_image: '/posts/images/PowerShell_PnPUnifiedLog/Sample.png'
omit_header_text: true
draft: true
---

{{< gist reshmee011 7e9e09d016e0384a6aff31d7d10804e8 >}}

#Parameters
$tenantUrl = Read-Host -Prompt "Enter tenant collection URL";
$dateTime = (Get-Date).toString("dd-MM-yyyy-hh-ss")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "EffectivePermissionsReport-" + $dateTime + ".csv"
$ReportOutput = $directorypath + "\Logs\"+ $fileName
$userName =  "i:0#.f|membership|test.test@contoso.co.uk"
#Connect to PnP Online
Connect-PnPOnline -Url $tenantUrl -Interactive

$global:Results = @();

function GetUserEffectivePermissions($_ctx,$_object,$_userName,$_type,$_siteUrl,$_listUrl)
{
    $user = get-pnpuser -Identity $_userName

     # Retrieve the user permissions on the site
     if($user){
     $permissions = $_object.GetUserEffectivePermissions($user.LoginName)
     $_ctx.ExecuteQuery()
     #get all base permissions granted to the user
     $PermissionKindObj= New-Object Microsoft.SharePoint.Client.PermissionKind 
     $PermissionKindType=$PermissionKindObj.getType()
     $permissionsToExport = @()
     ForEach ($PermissionKind in [System.Enum]::GetValues($PermissionKindType))
     {
         $hasPermisssion = $permissions.Value.Has($PermissionKind)
         if ($hasPermisssion)
         {
             $permissionsToExport +=$permissionKind.ToString()                   
         }
     }
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
                    User = $User.LoginName
                    EffectivePermissions =  ($permissionsToExport -join ",")
                })
                $global:Results +=$result;
            }
        }

#Exclude certain libraries
$ExcludedLists = @("Form Templates", "Preservation Hold Library", "Site Assets", "Images", "Pages", "Settings", "Videos","Timesheet"
    "Site Collection Documents", "Site Collection Images", "Style Library", "AppPages", "Apps for SharePoint", "Apps for Office")

$m365Sites = Get-PnPTenantSite| Where-Object { ($_.Url -like '*Comm*' -and $_.Url -like '*/TEAM-*') -and $_.Template -ne 'RedirectSite#0' } #($_.Url -like '*/TEAM-*' -or  $_.Url -like '*/ACT-*' -or  $_.Url -like '*/PROJ-*' -or $_.Template -eq 'TEAMCHANNEL#1')
#$m365Sites = Get-PnPTenantSite -Detailed | Where-Object {($_.Url -like '*Comm*') -and $_.Template -ne 'RedirectSite#0' }
$m365Sites | ForEach-Object {
$siteUrl = $_.Url;     
Connect-PnPOnline -Url $siteUrl -Interactive
$ctx = Get-PnPContext
$web= Get-PnPWeb

Write-Host "Processing site $siteUrl"  -Foregroundcolor "Red"; 

GetUserEffectivePermissions $ctx $web $userName "site" $siteUrl "";
$ll = Get-PnPList -Includes BaseType, Hidden, Title,HasUniqueRoleAssignments,RootFolder | Where-Object {$_.Hidden -eq $False -and $_.Title -notin $ExcludedLists } #$_.BaseType -eq "DocumentLibrary" 
  Write-Host "Number of lists $($ll.Count)";

  foreach($list in $ll)
  {
    $listUrl = $list.RootFolder.ServerRelativeUrl;       

    #Get all list items in batches
    $ListItems = Get-PnPListItem -List $list -PageSize 2000 

 GetUserEffectivePermissions $ctx $list $userName "list/library" $siteUrl $listUrl;
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
                GetUserEffectivePermissions $ctx $item $userName $type $siteUrl $listUrl;
            }
        }
    }
 }
  $global:Results | Export-CSV $ReportOutput -NoTypeInformation
Write-host -f Green "Effective Permissions for user generated Successfully!"