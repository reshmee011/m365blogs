---
title: "Troubleshooting SharePoint Storage Reports with PowerShell"
date: 2023-07-09T08:42:13+01:00
tags: ["SharePoint, PowerShell","Storage"]
draft: false
---

# Troubleshooting SharePoint Storage Reports with PowerShell

As the amount of data stored in SharePoint grows, it becomes important to monitor and manage storage usage. In this blog post, we will explore how to generate storage reports for SharePoint sites using PowerShell. These reports will provide insights into the storage usage of sites and individual files, including file versions and the size of the recycle bin. However the reports can only give a glimpse of at least 60-90% of storage.

![Tenant level report](../images/storage-breakdown-report/Preview_Site.png)
![Site level report](../images/storage-breakdown-report/Preview_File.png)

I noticed for two files where the storage usage from storage metrics was 1 MB despite the file size was ~ 20 Kb with only one version.

![StorageMetricsforFile](../images/storage-breakdown-report/StorageMetricsforFile.png)
![FileVersionSize](../images/storage-breakdown-report/FileVersionSize.png)

As per post [File size different in storage metrics than original even though file has only one version. Why so ?](https://answers.microsoft.com/en-us/msoffice/forum/all/file-size-different-in-storage-metrics-than/1e1a9300-5668-4aba-bc46-7c64c98cdbaf)

"If the files have many images, then we generate a larger thumbnail in the background which makes the file even bigger.

The storage metrics page shows the true storage value being used as a number of micro services run after upload to create thumbnail, previews, etc., which causes the increase in the storage used.

As confirmed from test environment, the file size increases in Storage metrics page irrespective of the versions sizes not adding up to the storage reflecting in the storage metrics page. 
Also, as observed the PowerPoint files did not have any versions, so even if versions are not there, there are other variables with the microservices running against the files.

This is by design and there is no way to predict how much space will be used after a document is uploaded. We can only say that the numbers shown in Storage Metrics do reflect the space being used by those documents."

Refer to the following posts for more info about storage within SharePoint.

[https://ms365thinking.blogspot.com/2022/02/pay-more-for-sharepoint-storage-than.html](https://ms365thinking.blogspot.com/2022/02/pay-more-for-sharepoint-storage-than.html)
[Deleting File Versions to reduce the SharePoint Storage Consumption](https://ms365thinking.blogspot.com/2023/05/deleting-file-versions-to-reduce.html)
[Sample on a report showing how much SharePoint Storage you can save by trimming document versions once the site is no longer active](https://pnp.github.io/script-samples/spo-generate-sp-storage-savings-report/README.html?tabs=pnpps)
[Total Size from Storage Metrics shows full size file version in SharePoint 2013](https://learn.microsoft.com/en-us/sharepoint/troubleshoot/administration/total-size-shows-full-size-file-version)
[SharePoint Storage metrics page is displaying incorrect information](https://answers.microsoft.com/en-us/msoffice/forum/all/sharepoint-storage-metrics-page-is-displaying/9ba4977e-6ad8-4298-8ece-621dfa6f0ae1)


## Prerequisites:
Before we begin, make sure you have the PnP PowerShell which can installed using instructions from [Installing PnP PowerShell](https://pnp.github.io/powershell/articles/installation.html)

Getting Started:
To generate SharePoint storage reports, we will use PnPPowerShell cmdlets in this blog post. Let's dive into the script that generates the reports.

```PowerShell
$SharePointAdminSiteURL = "https://contoso-admin.sharepoint.com"
$conn = Connect-PnPOnline -Url $SharePointAdminSiteURL -Interactive
# Set Variables
$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "\SiteStoragReport-" + $dateTime + ".csv"
$OutputSite = $directorypath + $fileName
$fileName = "\FileStorageReport-" + $dateTime + ".csv"
$OutPutFile = $directorypath + $fileName

$arraySite = New-Object System.Collections.ArrayList
$arrayFile = New-Object System.Collections.ArrayList
#Exclude certain libraries
$ExcludedLibraries = @()
function ReportStorageVersions($site) {
    try {
        $fileSizes = @(); 
        $fileSize = 0 
        $TotalVersionSize = 0
        $DocLibraries = Get-PnPList -Includes BaseType, Hidden, Title -Connection $siteconn | Where-Object { $_.BaseType -eq "DocumentLibrary" -and $_.Hidden -eq $False -and $_.Title -notin $ExcludedLibraries }
        $DocLibraries | ForEach-Object {
            Write-host "Processing Document Library:" $_.Title -f Yellow
            $library = $_
            $listItems = Get-PnPListItem -List $library.Title -Fields "ID" -PageSize 1000 -Connection $siteconn
            #Get file zize
            $listItems | ForEach-Object {
                $listitem = $_
                $fileVersionSize = 0
                $file = Get-PnPFile -Url $listitem["FileRef"] -AsFileObject -ErrorAction SilentlyContinue -Connection $siteconn  
                if ($file) {
                    $fileSize += $file.Length          
                    $elementFile = "" | Select-Object SiteUrl, siteName, siteStorage, FileRef,FileSize,TotalVersionSize,VersionCount,StartTime, EndTime
                    $elementFile.SiteUrl = $site.Url
                    $elementFile.siteName = $site.Title
                    $elementFile.siteStorage = "$siteStorage MB"
                    $elementFile.StartTime = (Get-Date).toString("dd-MM-yyyy HH:mm:ss")
                    $elementFile.FileRef  =   $listitem["FileRef"]
                    $fileversions = Get-PnPFileVersion -Url $listitem["FileRef"] -Connection $siteconn
                    if ($fileversions) {
                            # Calculate the total version size
                        $fileVersionSize = $fileversions | Measure-Object -Property Size -Sum | Select-Object -ExpandProperty Sum                                                   
                    }
                    $elementFile.FileSize = "$([Math]::Round(($file.Length/1MB),3)) MB" 
                    $elementFile.TotalVersionSize = "$([Math]::Round(($fileVersionSize/1MB),3)) MB"
                    $elementFile.VersionCount = $fileversions.Count
                    $totalVersionSize += $fileVersionSize
                    $elementFile.EndTime = (Get-Date).toString("dd-MM-yyyy HH:mm:ss")
                    $arrayFile.Add($elementFile) | Out-Null 
                }        
            }
        }
        $fileSizes += $fileSize
        $fileSizes += $totalVersionSize   

        return $fileSizes
    }
    catch {
        Write-Output " An exception was thrown: $($_.Exception.Message)" -ForegroundColor Red
    } 
}

# Get total storage use for this site collection
Get-PnPTenantSite -Connection $conn | Where-Object { ($_.Template -eq "GROUP#0" -or $_.Template -eq "SITEPAGEPUBLISHING#0")} | ForEach-Object {
    $site = $_
    $siteStorage = $site.StorageUsageCurrent
    Write-Host "Site storage: $siteStorage MB"
    $siteconn = Connect-PnPOnline -Url $site.Url -Interactive -ReturnConnection

    $element = "" | Select-Object SiteUrl, siteName, siteStorage, FileSize, StartTime,TotalVersionSize, RecycleBinSize,EndTime
    $element.SiteUrl = $site.Url
    $element.siteName = $site.Title
    $element.siteStorage = "$siteStorage MB"
    $element.StartTime = (Get-Date).toString("dd-MM-yyyy HH:mm:ss")
    $FileSizeVersions = ReportStorageVersions -site $site
    $element.FileSize = "$([Math]::Round(($FileSizeVersions[0]/1MB),3)) MB" 
    $element.TotalVersionSize = "$([Math]::Round(($FileSizeVersions[1]/1MB),3)) MB"
    $RecycleBinItemsSize = Get-PnPRecycleBinItem -Connection $siteconn | Measure-Object -Property Size -Sum | Select-Object -ExpandProperty Sum
    $element.RecycleBinSize = "$([Math]::Round(($RecycleBinItemsSize/1MB),3)) MB"
    $element.EndTime = (Get-Date).toString("dd-MM-yyyy HH:mm:ss")
    $arraySite.Add($element) | Out-Null 
}  

$arraySite | Export-Csv -Path $OutputSite -NoTypeInformation -Force 
$arrayFile | Export-Csv -Path $OutputFile -NoTypeInformation -Force

```

## Explanation:

- Gets the connection to the to the SharePoint admin centre Connect-PnPOnline .
- Sets variables to set the current date, the path for the output files, and create empty ArrayList objects to hold the site and file storage information.
- Defines a custom function called ReportStorageVersions that takes a SharePoint site as a parameter and iterates through document libraries, calculating the file sizes and version sizes.
    Uses Get-PnPList cmdlet to retrieve the document libraries, excluding any libraries specified in the $ExcludedLibraries array.
    Uses Get-PnPListItem to retrieve the list items and iterate through them to calculate the file sizes and version sizes using the Get-PnPFile and Get-PnPFileVersion cmdlets.
    The calculated storage information is stored in custom objects and added to the $arrayFile ArrayList.
- Uses Get-PnPTenantSite cmdlet to retrieve the SharePoint sites within the tenant and filter them based on the site templates we are interested in (e.g., "GROUP#0" for Microsoft Teams sites). The query can be amended as appropriately.
- Retrieves the site storage usage by connecting to the site using Connect-PnPOnline.
- Creates a custom object to store the site storage information and call the ReportStorageVersions function to retrieve the file sizes and version sizes.
- Calculates the size of the recycle bin using the Get-PnPRecycleBinItem cmdlet. The site storage information, including file sizes, version sizes, and recycle bin size, is added to the $arraySite ArrayList.
- Exports the $arraySite and $arrayFile ArrayLists to CSV files using the Export-Csv cmdlet.

## Conclusion:

In this blog post, we explored how to generate storage reports for SharePoint sites using PowerShell. By leveraging PnP PowerShell cmdlets, site storage usage, file sizes, version sizes, and recycle bin size could be calculated. These reports provide valuable insights into the storage consumption within SharePoint and can help administrators monitor and manage storage effectively. 