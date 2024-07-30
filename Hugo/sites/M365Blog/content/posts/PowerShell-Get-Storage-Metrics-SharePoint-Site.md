---
title: "Getting Storage Metrics for a SharePoint site"
date: 2024-07-29T06:57:18+01:00
tags: ["SharePoint","Storage Metrics","File","Folder","Site", "Library" ]
featured_image: '/posts/images/PowerShell-Get-Storage-Metrics-SharePoint-Site/StorageMetrics_Script_Output.png'
omit_header_text: true
draft: false
toc: true
---

Gaining an overview what takes space in a SharePoint is important to monitor any big files or huge number of versions.

Unfortunately the storagemetrics link won't work for libraries with more than 5k files without nested folders, yeps it might be against information architecture, creating folders is an option to avoid the list view threshold error

![Storage Metrics Error for folder more than 5k](../images/PowerShell-Get-Storage-Metrics-SharePoint-Site/Error_StorageMetrics_folder_morethan5k.png)

{{< gist reshmee011 c7c6d2efb14f6a622774e1c737585e53 >}}