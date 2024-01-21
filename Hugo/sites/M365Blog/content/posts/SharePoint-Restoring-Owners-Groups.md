---
title: "Recreating Deleted Owners Group for M365-Connected SharePoint Sites"
date: 2024-01-20T06:19:42Z
tags: ["SharePoint","Owners","M365 Owners" ,"M365", "Team site", "PowerShell","Security", "Governance"]
featured_image: '/posts/images/SharePoint-Restoring-Owners-Groups/DeletedOwnerGroup.png'
draft: False
---

# Recreating Deleted Owners Group for M365-Connected SharePoint Sites

If out-of-the-box (OOB) groups such as owners, members, or visitors have been deleted accidentally from your SharePoint site, this article may assist you in recovering those vanished groups specifically for M365 linked Team site. I recently encountered a distress call from an end user facing data access issues on a SharePoint Team site. To my dismay, I discovered that the SharePoint Owners group had been accidentally deleted, prompting me to seek and implement a solution to restore access. This article outlines the steps to revive essential groups and regain control over your SharePoint Team site.

## The Deletion Dilemma

End users can unwittingly delete standard SharePoint groups, leading to a groupless void.

### The Deletion Process

Users can delete OOB groups from Site Permissions with a few clicks, leaving no trace behind. The problem arises when these groups are crucial for site management, and their absence wreaks havoc.

End users are not able to delete OOB groups from Site Permissions link **{siteUrl}/_layouts/15/user.aspx**.
![User aspx](../images/SharePoint-Restoring-Owners-Groups/DeleteGroup_User_aspx.png)  

However end users are able to delete Out of the Box SharePoint groups : owners, members and visitors from **{siteUrl}/_layouts/15/groups.aspx**
![Group aspx](../images/SharePoint-Restoring-Owners-Groups/DeleteGroup_Group_aspx.png) 

Clicking on the Edit icon and delete will delete the group
![Delete Group aspx](../images/SharePoint-Restoring-Owners-Groups/DeleteOwnerGroup.png) 

Once deleted, the group is gone and there is no way to restore it. 

![Deleted Group](../images/SharePoint-Restoring-Owners-Groups/DeletedOwnerGroup.png) 

## The Restoration Process

Recreating SharePoint owners, members, and visitors groups is essential for restoring normalcy. Utilise the {siteUrl}_layouts/15/permsetup.aspx link to recreate the missing groups. See [Restoring Deleted Permission](https://learn.microsoft.com/en-us/answers/questions/526452/sharepoint-online-restoring-deleted-permission-gro) for more details.

![Recreate Group](../images/SharePoint-Restoring-Owners-Groups/RecreatetheOwnersGroup.png) 

The recreated owners group is not linked to the M365 group owners.If an user is added as owners using the modern permission interface, the user is not granted any permissions to the site by default
![Check Permission](../images/SharePoint-Restoring-Owners-Groups/CheckPermissions.png) 

## Reestablishing the Connection

However, the process doesn't end there. Reconnecting M365 Group Owners to SharePoint Owners is necessary. When a M365 connected Team site is created, the M365 group owners is added as a hidden principal to the SharePoint owners group. 

![Groupify Permissions](https://learn.microsoft.com/en-us/sharepoint/dev/transform/media/modernize/groupifypermissions_1.png)

Refer to the article for more details [https://learn.microsoft.com/en-us/sharepoint/dev/transform/modernize-connect-to-office365-group-permissions](https://learn.microsoft.com/en-us/sharepoint/dev/transform/modernize-connect-to-office365-group-permissions) how the M365 Group are linked to the SharePoint groups.

I have been unable to add the M365 Group Owners principal back into the SharePoint Owners group using the User Interface from SharePoint Admin Centre or SharePoint site . 

## Leveraging PowerShell

The claims of the M365 group uses the prefix c:0o.c|federateddirectoryclaimprovider

- c:0o.c|federateddirectoryclaimprovider|{M365Guid}: this claim represents the Microsoft 365 group members 
![Group Members](../images/SharePoint-Restoring-Owners-Groups/GroupMembersAccount.png) 

- c:0o.c|federateddirectoryclaimprovider|{M365Guid}_o: this claim represents the Microsoft 365 group owners
![Group Owners](../images/SharePoint-Restoring-Owners-Groups/GroupOwnersAccount.png) 

While the M365 members can easily be added back using the user interface, it is a challenge to add back the M365 Owners group.

By default the M365 group owners is added as site admins , a.k.a site collection administrators and just confirm it is in the group from the SharePoint Admin Centre, if not add owners to the site admins.

![Add Owners to SiteAdmin](../images/SharePoint-Restoring-Owners-Groups/AddOwnersToAdmin.png) 

```PowerShell

connect-pnpOnline -Url https://contoso.sharepoint.com/sites/testclone2 -interactive

$m365GroupId = (get-pnpsite -Includes RelatedGroupId).RelatedGroupId

$m365GroupOwnerClaims = "c:0o.c|federateddirectoryclaimprovider|{0}_o" -f $m365GroupId.Guid.ToString()

Add-PnPSiteCollectionAdmin -Owners $m365GroupOwnerClaims

$owner = Get-PnPGroup -AssociatedOwnerGroup | select Title

Add-PnPGroupMember -Group $owner.Title -LoginName $m365GroupOwnerClaims | Out-Null

<## 
 The commented code to attempt to set the owners group as hidden did not work hence left as hidden

$list = get-pnplist "User Information List"
 
$owner = get-pnplistitem  -List $list | where-object {$_.FieldValues.Name -eq $m365GroupOwnerClaims -and $_.FieldValues.EMail}
$owner.FieldValues.UserInfoHidden

set-pnplistitem -List $list -Identity $owner.Id -Values @{UserInfoHidden = $true;}
$owner = get-pnplistitem  -List $list | where-object {$_.FieldValues.Title -eq $m365GroupOwnerClaims -and $_.FieldValues.EMail}
 
$owner.FieldValues.UserInfoHidden
UserInfoHiddenset-pnplistitem -list
#>
```

After running the script, the M365 group owners are linked to the SharePoint Owners group albeit visible.

![LinkedM365GroupOwners.png](../images/SharePoint-Restoring-Owners-Groups/LinkedM365GroupOwners.png)

You can check an owner's permission whether access has been restored.

![Check Owner Permission](../images/SharePoint-Restoring-Owners-Groups/CheckOwnerPermission.png) 

The journey may present challenges, such as the UserInfoHidden property for the owners group to make it hidden principal which I have been unable to set as hidden.

## Alternative solutions

### Recreate the whole M365 Team site

Another M365 connected Team site can be created and all contents and services (Planner, Teams, etc..) can be migrated.

### PITR (Point in Time Restore)

You may contact Microsoft to assist in performing a PITR (point in time restore) of the site prior the the owners group deletion only if it happens in the last 14 days.

The ideal solution would be not to have an option to delete Out of the Box SharePoint groups.

Please comment if you ever faced similar scenerio and had other solutions. 

## References

[Add O365 Group Owner to Site Collection Administration List](https://techcommunity.microsoft.com/t5/sharepoint/add-o365-group-owner-to-site-collection-administration-list/m-p/401027)

[https://learn.microsoft.com/en-us/sharepoint/dev/transform/modernize-connect-to-office365-group-permissions](https://learn.microsoft.com/en-us/sharepoint/dev/transform/modernize-connect-to-office365-group-permissions)

