---
title: "How to recreated deleted owners group for M365 connected SharePoint site"
date: 2023-11-20T06:19:42Z
tags: ["Power Automate","Copilot"]
featured_image: '/posts/images/PowerAutomate_Copilot/NewFlowExperience_withPrompting.png'
draft: true
---

# How to recreated deleted owners group for M365 connected SharePoint site


End users are not able to delete OOB groups from Site Permissions {siteUrl}/_layouts/15/user.aspx
[User aspx](../images/SharePoint-Restoring-Owners-Groups/DeleteGroup_User_aspx.png)  


End users are able to delete OOB SharePoint groups : owners, members and visitors from {siteUrl}/_layouts/15/groups.aspx
[Group aspx](../images/SharePoint-Restoring-Owners-Groups/DeleteGroup_Group_aspx.png) 

Clicking on the Edit icon and delete will delete the group
[Delete Group aspx](../images/SharePoint-Restoring-Owners-Groups/DeleteOwnerGroup.png) 

Once deleted, the group is gone and there is no way to restore it. 

[Deleted Group](../images/SharePoint-Restoring-Owners-Groups/DeletedOwnerGroup.png) 

Using the link {siteUrl}_layouts/15/permsetup.aspx we can recreate the SharePoint owners, members and visitors groups. View [Restoring Deleted Permission](https://learn.microsoft.com/en-us/answers/questions/526452/sharepoint-online-restoring-deleted-permission-gro)

[Recreate Group](../images/SharePoint-Restoring-Owners-Groups/RecreatetheOwnersGroup.png) 


To find all users , go to 
{siteUrl}/_layouts/15/people.aspx?MembershipGroupId=0

Permissions have to be granted to individual libraires, e.g. Documents

The recreated owners group is not linked to the M365 group owners.If an user is added as owners using the modern permission interface, the user is not granted any permissions to the site by default
[Check Permission](../images/SharePoint-Restoring-Owners-Groups/CheckPermissions.png) 


Using the article [https://learn.microsoft.com/en-us/sharepoint/dev/transform/modernize-connect-to-office365-group-permissions](https://learn.microsoft.com/en-us/sharepoint/dev/transform/modernize-connect-to-office365-group-permissions) has great insights how the M365 Group are linked to the SharePoint groups.

I have been unable to add the M365 Group Owners principal back into the SharePoint Owners group using the User Interface from SharePoint Admin Centre or SharePoint site . 

M365 Group Members claims
[Group Members](../images/SharePoint-Restoring-Owners-Groups/GroupMembersAccount.png) 


M365 Owner Claims
[Group Owners](../images/SharePoint-Restoring-Owners-Groups/GroupOwnersAccount.png) 

The owners claims is the same as Members claims finishing with _o

By default the M365 group owners is added as site admins and just confirm it is in the group, else

```PowerShell
$SiteUrl = "https://contoso.sharepoint.com/teams/d-dev-testdeletedSPOwners"
$m365GroupName = "Dev Test Deleted SP Owners";
$m365GroupOwnersName = $m365GroupName + " Owners"
#$SiteUrl = "https://ppfonline.sharepoint.com/teams/proj-ppfremit"
Connect-PnPOnline -url $SiteUrl -Interactive
 
$siteAdmin = Get-PnPSiteCollectionAdmin | select-object {$_.Title -eq $m365GroupOwnersName}
#Add-PnPGroupMember -Group "Dev Test Deleted SP Owners Owners" -LoginName $siteAdmin.LoginName | Out-Null
 
$list = get-pnplist "User Information List"
 
$owner = get-pnplistitem  -List $list | where-object {$_.FieldValues.Title -eq $m365GroupOwnersName -and $_.FieldValues.EMail}
$owner.FieldValues.UserInfoHidden
#set-pnplistitem -List $list -Identity $owner.Id -Values @{UserInfoHidden = $true;}
#$owner = get-pnplistitem  -List $list | where-object {$_.FieldValues.Title -eq $m365GroupOwnersName -and $_.FieldValues.EMail}
 
#$owner.FieldValues.UserInfoHidden
#UserInfoHiddenset-pnplistitem -list

```


By default the owners group is added as a hidden principal but I have been unable to change the property UserInfoHidden for the owners group.
# References

[Add O365 Group Owner to Site Collection Administration List](https://techcommunity.microsoft.com/t5/sharepoint/add-o365-group-owner-to-site-collection-administration-list/m-p/401027)

[https://learn.microsoft.com/en-us/sharepoint/dev/transform/modernize-connect-to-office365-group-permissions](https://learn.microsoft.com/en-us/sharepoint/dev/transform/modernize-connect-to-office365-group-permissions)