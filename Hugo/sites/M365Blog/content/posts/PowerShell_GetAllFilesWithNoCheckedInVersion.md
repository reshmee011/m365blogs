---
title: "Discovering All Checked Out Files including those with no checked in versions with PnP PowerShell"
date: 2023-11-15T07:08:24Z
tags: ["OneDrive","Checked out","PowerShell", "PnP", "Mandatory Metadata"]
featured_image: '/posts/images/PowerShell_GetAllFilesWithNoCheckedInVersion/listofCheckedOutFiles.png'
omit_header_text: true
draft: false
---

# Discovering All Checked Out Files including those with no checked in versions

There are scenarios when files uploaded won't have "checked-in version" which will make the files visible only to their uploader. Two possible scenarios that can lead to the situation:

1. **Mandatory Metadata Requirements**: When there are mandatory fields configured on the libraries and end users use Onedrive as a medium to upload the files to SharePoint via a shortcut to OneDrive. End users are not prompted to fill in mandatory fields, instead they are presented with a series of warnings and errors.

![OneDrive](../images/PowerShell_GetAllFilesWithNoCheckedInVersion/SelectOneDriveLoc.png)

The experience could be 

- Warning to check in when closing the file.

![WarningtoCheckin](../images/PowerShell_GetAllFilesWithNoCheckedInVersion/WarningtoCheckin.png)

- Check in option for end user to select
![CheckinOption](../images/PowerShell_GetAllFilesWithNoCheckedInVersion/CheckinOption.png)

- Dreaded message "File is deleted" appears.
![file is deleted](../images/PowerShell_GetAllFilesWithNoCheckedInVersion/Messagefileisdeleted.png)

2. **Overlooked Check-Ins**:  When **Require Check Out** option under versioning settings of any library is set to "Yes" and the uploader forget to check in the file.

![file is deleted](../images/PowerShell_GetAllFilesWithNoCheckedInVersion/RequireDocsToBeCheckedOut.png)

## Taking ownership

As a site administrator, ownership of files can be taken for the files that have no checked-in version. To take control, Go to the “Manage files which have no checked-in version” link under Library Settings.

![Manage Field with no checked in versions](../images/PowerShell_GetAllFilesWithNoCheckedInVersion/ManageFilesWithNoChekedinVersion.png)

List of check out files 
![List of Checked Out Files](../images/PowerShell_GetAllFilesWithNoCheckedInVersion/listofCheckedOutFiles.png)

## Issues with files with no checked in versions

Files which have no checked in versions have the following issues
- Invisibility: They evade search results, remaining hidden from intended audiences.
- Backup Issues: These files are not backed up, risking data loss.
- Mass update of those files failed

## Script to discover checked out files using PnP PowerShell

The script will help find all files with no checked in versions and also files left in checked out state by end users.

```PowerShell
#Set Parameters
$AdminCenterURL="https://contoso-admin.sharepoint.com/"
Connect-PnPOnline -Url $AdminCenterURL -Interactive
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "checkedoutfiles-" + $dateTime + ".csv"
$OutPutView = $directorypath + "\Logs\"+ $fileName
# Array to Hold Result - PSObjects
$filesCollection = @()
 
#Array to Skip System Lists and Libraries
$SystemLists = @("Converted Forms", "Master Page Gallery", "Customized Reports", "Form Templates", "List Template Gallery", "Theme Gallery","Apps for SharePoint",
                            "Reporting Templates", "Solution Gallery", "Style Library", "Web Part Gallery","Site Assets", "wfpub", "Site Pages", "Images", "MicroFeed","Pages")
 
$m365Sites = Get-PnPTenantSite -Detailed | Where-Object {($_.Url -like '*/intranet-*' -or  $_.Url -like '*/team-*' -or $_.Template -eq 'TEAMCHANNEL#1') -and $_.Template -ne 'RedirectSite#0' }
$m365Sites | ForEach-Object {
$siteUrl = $_.Url;    
Connect-PnPOnline -Url $siteUrl -Interactive
 
$Ctx = Get-PnPContext
#Get the List
write-host  $siteUrl  
 Get-PnPList  | Where {$_.Hidden -eq $false -and $SystemLists -notcontains $_.Title -and $_.BaseTemplate -eq 101 } | ForEach-Object {
#Get All Checked-Out Files
$CheckedOutFiles = $_.GetCheckedOutFiles()
$Ctx.Load($CheckedOutFiles)
$Ctx.ExecuteQuery()
#Check-in All Files Checked out to the User
$CheckedOutFiles | ForEach-Object {

        $user = (Get-PnPUser -Identity $_.CheckedoutById -ErrorAction Ignore) ?? $_.CheckedoutById
        $ExportVw = New-Object PSObject
        $ExportVw | Add-Member -MemberType NoteProperty -name "Site URL" -value $siteUrl
        $ExportVw | Add-Member -MemberType NoteProperty -name "File Url" -value $_.ServerRelativePath.DecodedUrl
        $ExportVw | Add-Member -MemberType NoteProperty -name "Checked Out By" -value $user.Title
        $ExportVw | Add-Member -MemberType NoteProperty -name "No Checked in version" -value "Yes"
        $filesCollection += $ExportVw
    }
 
$alldocs = (Get-PnPListItem -List $_  -PageSize 1000 | where-object{ $null -ne $_.FieldValues.CheckoutUser} )
 
$alldocs | ForEach-Object {
        $ExportVw = New-Object PSObject
        $ExportVw | Add-Member -MemberType NoteProperty -name "Site URL" -value $siteUrl
        $ExportVw | Add-Member -MemberType NoteProperty -name "File Url" -value $_.FieldValues.FileRef
        $ExportVw | Add-Member -MemberType NoteProperty -name "Checked Out By" -value $_.FieldValues.CheckoutUser.LookupValue
        $ExportVw | Add-Member -MemberType NoteProperty -name "No Checked in version" -value "No"
        $filesCollection += $ExportVw
  }
 }
}
# Export the result array to CSV file
$filesCollection | sort-object "File Url" |Export-CSV $OutPutView -Force -NoTypeInformation
```

## Conclusion

Uncovering files without checked in versions will help with data integrity, compliance and records management in SharePoint. By employing PnP PowerShell and administrative actions, these files can be salvaged.

## References

[SharePoint Online: Manage Files Which Have No Checked in Version using PowerShell](https://www.sharepointdiary.com/2017/08/sharepoint-online-manage-files-which-have-no-checked-in-version-using-powershell.html)