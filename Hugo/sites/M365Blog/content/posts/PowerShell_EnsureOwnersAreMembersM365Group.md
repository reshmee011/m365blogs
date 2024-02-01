---
title: "Ensuring Owners Are Members"
date: 2023-10-17T09:57:15+01:00
tags: ["M365 Group","Owners","Members","PowerShell", "Permissions"]
featured_image: '/posts/images/PowerShell_ensureownersaremembers/GroupMembership.png'
draft: false
---

# Ensure Owners Are Members M365Group

Microsoft 365 (M365) Groups serve as a central hub for collaboration across various M365 applications like Teams, Planner, SharePoint, and more. While M365 roles include Owners, Members, and Guests, it's crucial to understand that being an owner doesn't always inherit member privileges. In this article, we'll explore why it's imperative to have M365 Group owners also serve as active members for seamless group management and productivity.

## The Impact of Owners Not Being Members

Sometimes, M365 Group owners may not be added as members automatically, potentially leading to various issues that can impede productivity and efficiency. One way to identify this is by checking the total members count, but the actual members, including owners, might be different. 
Members count from top right of SharePoint site

![Members](../images/PowerShell_ensureownersaremembers/members.png)

Group membership after clicking on members icon

![Group Membership](../images/PowerShell_ensureownersaremembers/GroupMembership.png)

This discrepancy may arise from various methods of managing M365 group permissions, such as through the Teams admin center, Microsoft Teams, SharePoint admin center, SharePoint connected sites, Planner, or scripting using PowerShell.

One might expect that being an owner would automatically grant the same privileges as being a member. However, it's important to note that behind the scenes, each M365 group is linked to a separate email address for group members, which does not include the owners.

Ensuring m365 owners are m365 members allow access to the different m365 apps 
    - The shared Outlook inbox
    - SharePoint document library or data shared to the m365 group members. You can grant access to a m365 group members and not to m365 group owners 
    - Team channels 

The script below ensures that all owners are members to the M365 group.

```powerShell
$AdminCenterURL="https://contoso-admin.sharepoint.com/"# Connect to SharePoint Online admin center
Connect-PnPOnline -Url $AdminCenterURL -Interactive
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "m365OwnersNotMembers-" + $dateTime + ".csv"
$OutPutView = $directorypath + "\Logs\"+ $fileName
# Array to Hold Result - PSObjects
$m365GroupCollection = @()
#Write-host $"$ownerName not part of member in $siteUrl";
$m365Sites = Get-PnPTenantSite -Detailed | Where-Object {($_.Url -like '*/TEST-*'-or $_.Template -eq 'TEAMCHANNEL#1') -and $_.Template -ne 'RedirectSite#0' }
$m365Sites | ForEach-Object {   
    $groupId = $_.GroupId;
    $siteUrl = $_.Url;
     if($_.Template -eq "GROUP#0")
     {
       #if owner is not part of m365 group member
    (Get-PnPMicrosoft365GroupOwner -Identity $groupId -ErrorAction Ignore) | foreach-object {
       $owner = $_;
        if(!(Get-PnPMicrosoft365GroupMember -Identity $groupId  -ErrorAction Ignore| Where-Object {$_.DisplayName -eq $owner.DisplayName}))
        {
            $ExportVw = New-Object PSObject
            $ExportVw | Add-Member -MemberType NoteProperty -name "Site URL" -value $siteUrl
           $ExportVw | Add-Member -MemberType NoteProperty -name "Owner Name" -value $owner.DisplayName
           $m365GroupCollection += $ExportVw
           Add-PnPMicrosoft365GroupMember -Identity $groupId  -Users $owner.Email
           Write-host $"$owner.DisplayName has been added as member in $siteUrl";
        }
    }
}
}
# Export the result array to CSV file
$m365GroupCollection | sort-object "Group Name" |Export-CSV $OutPutView -Force -NoTypeInformation
```

## Adding End Users as Members Before Owners

It's advisable to add end users as members before assigning them as owners, whether through interfaces or scripting.

```powerShell
  Add-PnPMicrosoft365GroupMember -Identity $site.GroupId -Users 'test@contoso.onmicrosoft.com'
  Add-PnPMicrosoft365GroupOwner -Identity $site.GroupId -Users 'test@contoso.onmicrosoft.com'
```

## References

[Overview of Microsoft 365 Groups for administrators](https://learn.microsoft.com/en-us/microsoft-365/admin/create-groups/office-365-groups?view=o365-worldwide&wt.mc_id=MVP_308367)

