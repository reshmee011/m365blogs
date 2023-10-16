---
title: "PowerShell_EnsureOwnersAreMembersM365Group"
date: 2023-10-13T09:57:15+01:00
tags: ["M365","Owners","Members","PowerShell"]
draft: true
---

# Ensure Owners Are Members M365Group

Owners without members for M365 group connection sites might experience several issues. The script below ensures that all owners are members to the M365 group.
Owners are added to M365 group members but sometimes the process does not happen.

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
           #Add-PnPMicrosoft365GroupMember -Identity $groupId  -Users $owner.Email
           Write-host $"$owner.DisplayName has been added as member in $siteUrl";
        }
    }
}
}
# Export the result array to CSV file
$m365GroupCollection | sort-object "Group Name" |Export-CSV $OutPutView -Force -NoTypeInformation
```
It is a good idea to add end user as member before adding as owner. 

```powerShell
  Add-PnPMicrosoft365GroupMember -Identity $site.GroupId -Users 'test@contoso.onmicrosoft.com'
  Add-PnPMicrosoft365GroupOwner -Identity $site.GroupId -Users 'test@contoso.onmicrosoft.com'
```
