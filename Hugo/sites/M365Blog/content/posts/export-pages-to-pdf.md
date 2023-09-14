---
title: "Export SharePoint Pages to PDF"
date: 2023-07-18T07:49:47+01:00
draft: true
tags: ["Export to PDF", "Chrome","SharePoint Page"]
---

# Export SharePoint Pages to PDF

There may be scenerios where you might want to take screenshots of all your pages in your SharePoint site. For instance before revamping a site, you might take screenshots of all pages to compare against updated site to show end client progress or difference the new design brings or could be simply for backing up the pages before archiving the site.

## Prerequisites
Google Chrome installed
PnP PowerShell installed

Full list of chrom flags https://github.com/GoogleChrome/chrome-launcher/blob/main/docs/chrome-flags-for-tools.md

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
```