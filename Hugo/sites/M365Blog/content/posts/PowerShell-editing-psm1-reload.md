---
title: "Working with psm1 Files in PowerShell"
date: 2024-02-07T17:32:36Z
tags: ["psm1","PowerShell","PnP"]
featured_image: '/posts/images/PowerShell-editing-psm1-reload/script.png'
draft: true
---

# Working with PowerShell Script Module Files

Have you ever utilized a psm1 file to create reusable functions across multiple ps1 files? If so, you might have encountered the frustration of edits not reflecting immediately due to module caching.

I recently faced this issue when updating a psm1 file; despite making changes, the updates didn't take effect immediately. The issue stemmed from module caching.

The only solution I found to load the updated psm1 file was to terminate the PowerShell session and start a new one. This hiccup highlighted the importance of understanding how PowerShell handles module caching and the necessity of refreshing the session to apply changes.

If you prefer not to install the psm1 module permanently, you can call it using the 'using' keyword:

An example of functions within the psm1 file saved as sp-common.psm1

```PowerShell
using module PnP.PowerShell

class SpViewInsert {
    [string]$ViewName
    [string]$AddAfter
}

class SpField {
    [string]$ListName
    [string]$FieldName
    [string]$FieldType
    [string]$FieldXml
    [SpViewInsert[]]$AddToViews
}

function Confirm-Connection($Connection) {
    if ($null -eq $Connection -or $Connection.GetType().Name -ne "PnPConnection") {
        Write-Warning "Connection missing or not a valid PnPConnection"
        Exit
    }
}

function Add-FieldToView([PnP.PowerShell.Commands.Base.PnPConnection]$connection, [string]$listName, [string]$fieldName, [SpViewInsert]$viewInsert) {
    $view = Get-PnPView -Connection $connection -List $listName -Identity $viewInsert.ViewName
    $existingFields = New-Object System.Collections.ArrayList
    $view.ViewFields | ForEach-Object { $null = $existingFields.Add($_) }
    $addAfterIndex = $existingFields.IndexOf($viewInsert.AddAfter)
    if ($addAfterIndex -gt -1) {
        $existingFields.Insert($addAfterIndex + 1, $fieldName)
        $null = Set-PnPView -Connection $connection -List $listName -Identity $viewInsert.ViewName -Fields $existingFields
        Write-Host "--Added field '$($fieldName)' to '$($viewInsert.ViewName)' view on '$($listName)' list/library"
    }
}

function Add-SharePointListFields([PnP.PowerShell.Commands.Base.PnPConnection]$connection, [SpField[]]$fields) {
    $fields | ForEach-Object {
        $newField = $_
        if ($null -ne (Get-PnPField -Connection $connection -List $newField.ListName -Identity $newField.FieldName -ErrorAction Ignore)) {
            Write-Warning "Field '$($newField.FieldName)' already exists on '$($newField.ListName)' list/library"
        }
        else {
            $null = Add-PnPFieldFromXml -Connection $connection -List $newField.ListName -FieldXml $newField.FieldXml -ErrorAction Break
            Write-Host "Added field '$($newField.FieldName)' to '$($newField.ListName)' list/library"

            if ($null -ne $newField.AddToViews) {
                $newField.AddToViews | ForEach-Object {
                    $addToView = $_
                    Add-FieldToView $connection $newField.ListName $newField.fieldName $addToView
                }
            }
        }
    }
}

Export-ModuleMember -Function Confirm-Connection
Export-ModuleMember -Function Add-SharePointListFields
```

Instead of installing the psm1, `Using` can be used to load psm1 file to use.

```PowerShell
#load the sp-common.psm1
using module ..\..\sp-common.psm1
[CmdletBinding()]
Param(
    [Parameter(Mandatory = $true)][PnP.PowerShell.Commands.Base.PnPConnection]$Connection
)

[SpField[]]$newFields = @(
    @{
        ListName   = "Payments"
        FieldName  = "PaymentType"
        FieldXml   = '<Field Type="Choice" DisplayName="Match Type" Required="FALSE" EnforceUniqueValues="FALSE" Indexed="FALSE" Format="Dropdown" FillInChoice="FALSE" ID="{1aaaaaaa-eaaa-4aaa-baaa-eaaaaaaa}" SourceID="{{listid:Payments}}" StaticName="PaymentType" Name="PaymentType" ColName="nvarchar10" RowOrdinal="0" CustomFormatter="" Version="1">
                        <CHOICES>
                            <CHOICE>Auto</CHOICE>
                            <CHOICE>Manual</CHOICE>
                            </CHOICES>
                        </Field>'
        AddToViews = @(
            @{
                ViewName = "All Items"
                AddAfter = "PaymentType"
            }
        )
    }
)

Confirm-Connection $Connection
Add-SharePointListFields $Connection $newFields
```

This approach allows you to utilize the module without installing it permanently, facilitating easier testing and experimentation.

## References

[How to Write a PowerShell Script Module](https://learn.microsoft.com/en-us/powershell/scripting/developer/module/how-to-write-a-powershell-script-module?view=powershell-7.4)
