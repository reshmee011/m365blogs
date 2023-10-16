---
title: "Ensure Taxonomy Feature In Teams Channel Sites"
date: 2023-10-13T09:57:45+01:00
tags: ["Taxonomy","Teams Channels","Managed Metadata","PowerShell"]
draft: true
---

# Ensure Taxonomy Feature In Teams Channel Sites

Taxonomy feature no enabled in Channel associated sites. Attempting to add content types with managed metadata columns failed with message "Taxonomy disabled"


```powershell
    $dsiteUrl = <Teams Channel SharePoint Url>
    $FeatureTaxonomy = "73ef14b1-13a9-416b-a9b5-ececa2b0604c" #enabled taxonomy

    Connect-PnPOnline -Url $siteUrl -Interactive
    Enable-PnPFeature -Identity $FeatureTaxonomy -Scope Site -Force
    # Add content types

    $list = New-Object Collections.Generic.List[string]

    $list.Add('0x010100236CAD7E91D6CA4FAD041395A8952B71')

    $list.Add('0x010100AF32BA6AA57E2843A130C0B439CDE045')

    Add-PnPContentTypesFromContentTypeHub -ContentTypes $list -ErrorAction Continue
```