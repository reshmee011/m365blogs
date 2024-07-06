---
title: "Get Total Count of SharePoint Files, Folders, and Items with PnP PowerShell"
date: 2024-07-05T07:17:21+01:00
tags: ["PowerShell", "Inventory","SharePoint Online","Libraries","Lists"]
featured_image: '/posts/images/powershell-get-totalcount-of-files-folders-items-in-tenant/example.png'
draft: false
---

# Get Total Count of SharePoint Files, Folders, and Items with PnP PowerShell

This PowerShell script powered by PnP PowerShell can help to get total count of files, folders, and list items across SharePoint tenant. This script is invaluable for administrators looking to perform audits, verify data migrations, or simply keep tabs on the content sprawl within their environments. The use case of this script was to get a total number of items that would be ingested into a third party application Records365 (provided by RecordPoint) to ensure the number tally for compliance purposes and identify any gaps.

## Script Overview

The script connects to SharePoint Online tenant and iterates over each site collection, excluding specific system sites. It tallies the number of files, folders, and list items, excluding items from system lists and libraries. The results are compiled into a CSV report for easy analysis.

### Prerequisites

- SharePoint Online Management Shell
- PnP PowerShell Module

### Script Breakdown

{{< gist reshmee011 88b3db1d88a0aed18f8edd0e93bdc755 >}}

![example output](../images/powershell-get-totalcount-of-files-folders-items-in-tenant/example.png)

This script provides a detailed snapshot of your SharePoint content, helping you manage your digital estate more effectively. Whether you're conducting a routine audit or preparing for a migration, this tool simplifies the process of inventorying your SharePoint assets.

