---
title: "Add List Canvas App to Solution for possible ALM"
date: 2023-07-18T07:49:47+01:00
draft: true
---

Though it is not a complete solution 
```PowerShell
# Variables
$webUrl = "https://connectonline.ppfonline.co.uk"
$rootUrl = "https://connectonline.ppfonline.co.uk"
$libraryName = "Pages"
$outputFolder = "E:\PDFs"
$chromePath = "C:\Program Files\Google\Chrome\Application\chrome.exe";
# Load SharePoint snap-in
if((Get-PSSnapin "Microsoft.SharePoint.PowerShell" -ErrorAction SilentlyContinue) -eq $null) 
{
    Add-PSSnapin "Microsoft.SharePoint.PowerShell"
}

function exportToPDF($web)
{
    $library = $web.Lists[$libraryName]
    $folderUrl = "$outputFolder\$($web.Title -replace '[~#%&*{}|:<>?/|"]', '_'  )";
    # Create the output folder if it doesn't exist
    if (!(Test-Path -Path $folderUrl))
    {
        New-Item -ItemType Directory -Path $folderUrl
    }
$i=0;
# Loop through all pages in the library
foreach ($item in $library.Items)
{
    # Get the file and URL of the page
    $file = $item.File
    $fileUrl = $file.ServerRelativeUrl
    $pageUrl = "$rootUrl$fileUrl"

    # Build the output file path
    #$outputFile = "$outputFolder\$($file.Name).pdf"

    # Export the page to PDF using the Save As feature
    $outfile = "$folderUrl\" + $file.Name.ToString().Replace(".aspx","")+ "_"+ $i++ + ".pdf"
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless --disableGPU --print-to-pdf="$outfile" "$pageUrl"

    # Output the status
    Write-Host "Exported page $pageUrl to $outfile"
    Start-Sleep -Seconds 5
}

# Dispose of the web object
$web.Dispose()
}

# Get the web and library
# Get the root web
$rootWeb = Get-SPWeb $webUrl
   exportToPDF $rootWeb
<# Loop through each sub-web
foreach ($web in $rootWeb.Webs)
{
   exportToPDF $web
   foreach ($web1 in $web.Webs)
    {
      exportToPDF $web1
      Start-Sleep -Seconds 10
       foreach ($web2 in $web1.Webs)
       {
           exportToPDF $web2
             Start-Sleep -Seconds 10
            foreach ($web3 in $web2.Webs)
            {
              exportToPDF $web3
                 Start-Sleep -Seconds 10
            }
       }
    }
}

#>
