---
title: "PowerShell Get Permissions Sites Libraries"
date: 2024-02-09T14:00:52Z
draft: true
---

```PowerShell
function FindUserinRoleAssignment($RoleAssignments, $list, $web){
    $RoleAssignments| foreach-object {  
        $roleAssignment = $_
        #Get the Permission Levels assigned and Member  
        Get-PnPProperty -ClientObject $roleAssignment -Property RoleDefinitionBindings, Member  
        #Get the Principal Type: User, SP Group, AD Group  
        $PermissionType = $RoleAssignment.Member.PrincipalType  
        $PermissionLevels = $RoleAssignment.RoleDefinitionBindings | Select -ExpandProperty Name  
        #Get all permission levels assigned (Excluding:Limited Access)  
        $PermissionLevels = ($PermissionLevels | Where { $_ -ne "Limited Access"}) -join ","  
        If($PermissionLevels.Length -gt 0) {
        #Get SharePoint group members  
            If($PermissionType -eq "SharePointGroup")  {  
                #Get Group Members  
                $GroupMembers = Get-PnPGroupMember -Identity $RoleAssignment.Member.LoginName                  
                #Leave Empty Groups  
                If($GroupMembers.count -gt 0){
                ForEach($user in $GroupMembers)  {  
                    if($global:userEmail -contains $user.email){
                        # If the user has permissions, output the site collection URL
                        #Write-Host "User $($userEmail) has restricted access of type $PermissionType to $($web.url) on library $($list.Title)"
                   
                    #Add the Data to Object  
                    $Permissions = [PSCustomObject]@{
                        User = $User.Title
                        Type= $PermissionType
                        Permissions= $PermissionLevels  
                        library= $list.Title  
                        site= $web.Url
                        GrantedThrough= ("SharePoint Group: $($RoleAssignment.Member.LoginName)")
                    }
                    $global:PermissionCollection += $Permissions
                }
            }  
          }  
       }
        else  
        {  
            if($global:userEmail -contains $RoleAssignment.Member.email){
                # If the user has permissions, output the site collection URL
                Write-Host "User $($userEmail) has direct permissions $PermissionType to $($web.Url) on library $($list.Title)"
           
            #Add the Data to Object  
            $Permissions = [PSCustomObject]@{
                User = $RoleAssignment.Member.Title
                Type= $PermissionType
                Permissions= $PermissionLevels
                library= $list.Title  
                site= $web.Url
                GrantedThrough= "Direct Permissions"
            }
            $global:PermissionCollection += $Permissions
         }
        }  
    }
  }
}
 
# Set the user's email address
$global:userEmail = @("test1@contoso.co.uk","test2@contoso.co.uk")
 
#Set Parameters
$AdminCenterURL="https://contoso-admin.sharepoint.com"
#$SiteURL = "https://contoso.sharepoint.com/teams/d-team-site"
Connect-PnPOnline -Url $AdminCenterURL -Interactive
$dateTime = (Get-Date).toString("dd-MM-yyyy-HH-mm")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "permissions-" + $dateTime + ".csv"
$OutPutView = $directorypath + "\Logs\"+ $fileName
#Array to Skip System Lists and Libraries
$SystemLists = @("Converted Forms", "Master Page Gallery", "Customized Reports", "Form Templates", "List Template Gallery", "Theme Gallery","Apps for SharePoint",
                            "Reporting Templates", "Solution Gallery", "Style Library", "Web Part Gallery","Site Assets", "wfpub", "Site Pages", "Images", "MicroFeed","Pages")
 
$global:PermissionCollection = @()  
$m365Sites = Get-PnPTenantSite -Detailed | Where-Object {($_.Url -like '*/TEAM-*' -or $_.Template -eq 'TEAMCHANNEL#1') -and $_.Template -ne 'RedirectSite#0' }
$m365Sites | ForEach-Object {
$siteUrl = $_.Url;    
Connect-PnPOnline -Url $siteUrl -Interactive
$web = Get-PnPWeb  -Includes RoleAssignments
 
 
FindUserinRoleAssignment $web.RoleAssignments "" $web
 
Get-PnPList  -Includes RoleAssignments,HasUniqueRoleAssignments  | Where-object {$_.Hidden -eq $false -and $SystemLists -notcontains $_.Title -and $_.BaseTemplate -eq 101 } | ForEach-Object {
$list = $_
    #check if broken inheritance
if($_.HasUniqueRoleAssignments){
 
# Get all users and groups who has access  
    $RoleAssignments = $list.RoleAssignments  
    FindUserinRoleAssignment $RoleAssignments $list $web
    }
  }
}
 
#Export Permissions to CSV File  
#$PermissionCollection  
$global:PermissionCollection | Export-CSV $OutPutView -NoTypeInformation  
Write-host -f Green "Permission Report Generated Successfully!"
```

<#get-pnpentraidgroupmember -identity "AAD_DLG_AzDevOps_D365_Marvin_Team"
$RestMethodUrl = $RestMethodUrl = 'v1.0/users/reshmee.auckloo@ppf.co.uk/transitiveMemberof/microsoft.graph.group?$count=true&$orderby=displayName&$search="displayName:AAD_DLG_U_SP_IntranetOwners"&$select=displayName,id'
$groups = (Invoke-PnPGraphMethod -Url $RestMethodUrl -Method Get -ConsistencyLevelEventual).value

<#get-pnpentraidgroupmember -identity "AAD_DLG_AzDevOps_D365_Marvin_Team"
$RestMethodUrl = $RestMethodUrl = 'v1.0/groups/e6b11663-3a4a-408b-beaa-c6061be04cdb/transitiveMembers/microsoft.graph.user?$count=true&$orderby=displayName&$search="displayName:kevin"&$select=displayName,id'
$groups = (Invoke-PnPGraphMethod -Url $RestMethodUrl -Method Get -ConsistencyLevelEventual).value

## references
https://learn.microsoft.com/en-us/graph/api/group-list-transitivemembers?view=graph-rest-1.0&tabs=http
https://powerusers.microsoft.com/t5/Building-Flows/List-users-together-with-nested-groups-from-Azure-Active/td-p/2263771
