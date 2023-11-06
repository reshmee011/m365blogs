---
title: "PowerShell_GetAllFilesWithNoCheckedInVersion"
date: 2023-11-06T07:08:24Z
draft: true
---

# Get All Checked Out Files including those with checked in versions

```PowerShell
[1:27 PM] Reshmee Auckloo
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
 
$m365Sites = Get-PnPTenantSite -Detailed | Where-Object {($_.Url -like '*/TEAM-*' -or  $_.Url -like '*/ACT-*' -or  $_.Url -like '*/PROJ-*' -or $_.Template -eq 'TEAMCHANNEL#1') -and $_.Template -ne 'RedirectSite#0' }
#$m365Sites = Get-PnPTenantSite -Detailed | Where-Object {($_.Url -like '*Comm*') -and $_.Template -ne 'RedirectSite#0' }
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
        # write-host  $_.ServerRelativePath.DecodedUrl
         $user = (Get-PnPUser -Identity $_.CheckedoutById -ErrorAction Ignore) ?? $_.CheckedoutById
         #write-host  $user.Title    
 
        $ExportVw = New-Object PSObject
        $ExportVw | Add-Member -MemberType NoteProperty -name "Site URL" -value $siteUrl
           
        $ExportVw | Add-Member -MemberType NoteProperty -name "File Url" -value $_.ServerRelativePath.DecodedUrl
        $ExportVw | Add-Member -MemberType NoteProperty -name "Checked Out By" -value $user.Title
        $ExportVw | Add-Member -MemberType NoteProperty -name "No Checked in version" -value "Yes"
        $filesCollection += $ExportVw
        #Take over the Checked-Out File
#        $_.TakeOverCheckOut()
 #       $Ctx.ExecuteQuery()
        #Check-in the file
  #      $CheckInFileURL = [String]::Concat($List.ParentWebUrl, $_.ServerRelativePath.DecodedUrl.Replace($List.ParentWebUrl, [string]::Empty))
   #     Set-PnPFileCheckedIn -Url $CheckInFileURL -CheckinType MajorCheckIn -Comment "Checked-In by the Script at $(Get-Date)"  
       # Write-Host "Check-in the File: $CheckInFileURL"  
    }
 
$alldocs = (Get-PnPListItem -List $_  -PageSize 1000 | where-object{ $null -ne $_.FieldValues.CheckoutUser} )
 
$alldocs | ForEach-Object {
        #write-host $_.FieldValues.FileRef
        #write-host $_.FieldValues.CheckoutUser.LookupValue
 
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

