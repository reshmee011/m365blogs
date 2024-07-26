---
title: "Update Choice values of List Items in SharePoint List"
date: 2024-07-13T06:39:48Z
tags: ["PowerShell","Choice","SharePoint","List"]
featured_image: '/posts/images/powershell-update-choice-value-from-old-to-new/csv.png'
omit_header_text: true
draft: true
---


function Delete-ARData {
    param (
        [Parameter(Mandatory = $true)
        ][string] $SourceSiteUrl,
        [Parameter(Mandatory = $true)]
        [string]$SourceFolderPath,

        [Parameter(Mandatory = $true)]
        [string]$DestinationSiteUrl,
        [Parameter(Mandatory = $true)]
        [string]$DestinationFolderPath
    )

    #List Parameters
    $RecoveriesList = "Recoveries"
    $RemittancesList = "Remittances"
    $RemittancesLibrary = "Remittance Documents"

  
     # Generate a unique log file name using today's date and time
     $todayDate = Get-Date -Format "yyyy-MM-dd_HHmm"
 
     $informationLogFileName = "copy-file-information_$todayDate.log"
     $informationLogFilePath = Join-Path -Path $PSScriptRoot -ChildPath $informationLogFileName
   # Tenant URL
   $tenantUrl = "https://contoso.sharepoint.com"

   $startTime = Get-Date -Format "yyyy-MM-dd HH:mm"
   $informationMessage = "Delete Process Started '$startTime'."
   Write-Host $informationMessage -ForegroundColor Green
   Add-Content -Path $informationLogFilePath -Value $informationMessage

   # Connect to the source and destination SharePoint sites
   Connect-PnPOnline -Url $SourceSiteUrl -Interactive
   $sourceConn = Get-PnPConnection 
   Connect-PnPOnline -Url $DestinationSiteUrl -Interactive
   $destinationConn = Get-PnPConnection

   $sourceRelativeFolerPath = $SourceSiteUrl.replace($tenantUrl, "") + $SourceFolderPath
   $destinationRelativeFolderPath = $DestinationSiteUrl.replace($tenantUrl, "") + $DestinationFolderPath

     #Get Source Data
  # Get source files
  $sourceFiles = Get-PnPFolderItem  -FolderSiteRelativeUrl $SourceFolderPath -ItemType File -Recursive -Connection $sourceConn
  $informationMessage = "Total '$($sourceFiles.count)' files."
  Write-Host $informationMessage -ForegroundColor Green
  Add-Content -Path $informationLogFilePath -Value $informationMessage

  foreach ($file in $sourceFiles) {
     # Get sub folder Path
     $subFolderPath = $file.ServerRelativeUrl.Replace($sourceRelativeFolerPath, "").replace($file.Name, "")

     # Get destination relative folder path with sub folder
     $newDestinationRelativeFolderPath = $destinationRelativeFolderPath + $subFolderPath.TrimEnd("/")
     try {

        #check where file exists before deleting file
        $fileUrl = $newDestinationRelativeFolderPath + "/" + $file.Name
        $fileAslistItem =  Get-PnPFile -Url $fileUrl -AsListItem -Connection $destinationConn
        #Get Destination Data from temp library
        if($fileAslistItem)
        {
        #Get remittances Docs from remittance docs library using $file.Name
        $remFiles = Find-PnPFile -List $RemittancesLibrary -Match   "*$($file.Name)" -Connection $destinationConn
        $remFiles | ForEach-Object {
           $remFileAsItemId =  (Get-PnPFile -Url $_.ServerRelativeUrl -AsListItem -Connection $destinationConn).Id
            #Get/Delete remittances Data from Remittance lists
            Get-PnPListItem -List $RemittancesList -PageSize 5000 -Connection $destinationConn| Where-Object {$_.FieldValues.DocumentSourceReference -eq $remFileAsItemId }| ForEach-Object{
              Remove-PnPListItem -List $RemittancesList -Identity $_.Id -Connection $destinationConn -Force
              $informationMessage = "Deleting Remittance ID $($_.Id) for sourceref $($remFileAsItemId)"
              Write-Host $informationMessage -ForegroundColor Green
              Add-Content -Path $informationLogFilePath -Value $informationMessage
            }

                      
            #Get/Delete Payments Data from Recoveries lists
            $recoveriesItems = Get-PnPListItem -List $RecoveriesList -PageSize 5000 -Connection $destinationConn| Where-Object {$_.FieldValues.DocumentSourceReference -eq $remFileAsItemId } | ForEach-Object {
              Remove-PnPListItem -List $RecoveriesList -Identity $_.Id -Connection $destinationConn -Force
              $informationMessage = "Deleting Recoveries ID $($_.Id) for sourceref $($remFileAsItemId)"
              Write-Host $informationMessage -ForegroundColor Green
              Add-Content -Path $informationLogFilePath -Value $informationMessage
            }
          #delete rem doc
          Remove-PnPListItem -List $RemittancesLibrary -Identity $remFileAsItemId -Connection $destinationConn -Force
          $informationMessage = "Deleting rem document ID $($remFileAsItemId)"
          Write-Host $informationMessage -ForegroundColor Green
          Add-Content -Path $informationLogFilePath -Value $informationMessage
        }
        #delete temp doc
              Remove-PnPFile -ServerRelativeUrl $fileUrl -Connection $destinationConn -Force
              $informationMessage = "Deleting temp library file $($fileUrl)"
              Write-Host $informationMessage -ForegroundColor Green
              Add-Content -Path $informationLogFilePath -Value $informationMessage
     }
     else{
        $informationMessage = "File '$($file.Name)' does not exist in '$newDestinationRelativeFolderPath'"
        Add-Content -Path $informationLogFilePath -Value $informationMessage
      } 
    }
    catch {
        $errorMessage = "Error: Deleting file '$($file.Name)' to '$newDestinationRelativeFolderPath' - $($_.Exception.Message)"
        Write-Host $errorMessage -ForegroundColor Red
        Add-Content -Path $errorLogFilePath -Value $errorMessage
    }
  }
}


Delete-ARData `
-SourceSiteUrl "https://contoso.sharepoint.com/teams/t-app-ar" `
-SourceFolderPath "/Test Data/UAT Testing V3/Apr 2023/Temp Library/ReAssure/PPF" `
-DestinationSiteUrl "https://contoso.sharepoint.com/teams/u-app-ar" `
-DestinationFolderPath "/TempLibrary/ReAssure/PPF" `


