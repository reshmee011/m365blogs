---
title: "Add M365 groups to SharePoint Groups"
date: 2024-04-21T17:32:36Z
tags: ["psm1","PowerShell","PnP","PowerShell Script Module"]
featured_image: '/posts/images/PowerShell-editing-psm1-reload/script.png'
omit_header_text: true
draft: true
---

$config = @{
    libraries = @(
        @{
            title= "Actuarial"
            owners = @(
                'user.pimms@contoso.co.uk'
            )
        }
        @{
            title= "Board Support"
            owners = @(
                'user.bas@contoso.co.uk'
            )
        }
        @{
            title= "Change"
            owners = @(
                'user.bas@contoso.co.uk'
            )
        }
        @{
            title= "Communications"
            owners = @(
                'user.bas@contoso.co.uk'
            )
        }
@{ title= "Finance"
owners = @(
    'user.pimms@contoso.co.uk'
)}
@{title="HR"
owners = @(
    'user.bas@contoso.co.uk'
)}
@{title="Investment"
owners = @(
    'user.carpenter@contoso.co.uk'
)}
@{title="IT"
owners = @(
    'user.will@contoso.co.uk'
)}
@{title="Legal"
owners = @(
    'user.bas@contoso.co.uk'
)}

 )
}
 
$siteTitle = "Commercial Services Team Site"
function Configure-LibraryPermissions($lib) {
    function Create-Group($title, $perm, $desc) {
        $groupExist = $groups | Where { $_.Title -eq $title }
        if ($groupExist -eq $null) {
            New-PnPGroup -Title $title -Owner "$siteTitle Owners" -Description $desc | Out-Null
            Write-Host "Created $title group"
        }
 
    # Add group to list as Edit
      Set-PnPListPermission -Identity $lib.title -Group $title -AddRole $perm | Out-Null
    # Remove group from site permissions
      Set-PnPWebPermission -Group $title -RemoveRole 'Read' -ErrorAction SilentlyContinue | Out-Null
}
 
$safeLibTitle = $lib.title -replace '["\/\\\[\]:\|<>\+=;,\?\*''@]', ''
 # Create Owners group
 if ($lib.owners -ne $null -and $lib.owners.Length -gt 0) {
    Create-Group "Custom Library Owners - $safeLibTitle" 'Full Control' "Add people to this group to grant access to the $($lib.title) library. Review regularly and remove people who no longer need access."
 
        $lib.owners | ForEach-Object {
        Add-PnPGroupMember -Group "Custom Library Owners - $safeLibTitle" -LoginName $_ | Out-Null
        Write-Host "Added $_ to Custom Library Owners - $safeLibTitle"
    }
  }  
 
}
 
$siteUrl = "https://contosoonline.sharepoint.com/sites/test"
Connect-PnPOnline -url $siteUrl -Interactive
 
$config.libraries | ForEach-Object {
    Configure-LibraryPermissions $_    
}
