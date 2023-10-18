---
title: "Ensure Taxonomy Feature in SharePoint Sites Connected to Private/Shared Teams Channels"
date: 2023-10-18T09:57:45+01:00
tags: ["Taxonomy","Teams Channels","Managed Metadata","PowerShell"]
featured_image: '/posts/images/PowerShell_EnsureTaxonomyFeatureInTeamsChannelSites/TaxonomyDisabled.png'
draft: false
---

# Ensure Taxonomy Feature In SharePoint site connected to a Private/Shared Channel Sites

Taxonomy feature is not activated by default in SharePoint sites linked to a private or shared Teams Channels. 
When attempting to add content types with managed metadata columns, you may encounter an error message stating "Taxonomy disabled".

![Taxonomy disabled](../images/PowerShell_EnsureTaxonomyFeatureInTeamsChannelSites/TaxonomyDisabled.png)

To resolve this issue, you can enable the taxonomy feature with the ID **73ef14b1-13a9-416b-a9b5-ececa2b0604c** using the PowerShell cmdlet **Enable-PnPFeature** before adding the content types to the sites with the template TEAMCHANNEL#1.


```powershell
    $dsiteUrl = <Teams Channel SharePoint Url>
    $FeatureTaxonomy = "73ef14b1-13a9-416b-a9b5-ececa2b0604c" #enabled taxonomy

    Connect-PnPOnline -Url $siteUrl -Interactive
    Enable-PnPFeature -Identity $FeatureTaxonomy -Scope Site -Force
    # Add content types

    $list = New-Object Collections.Generic.List[string]
    $list.Add('0x0101AABBCCDDEEFF00112233445566778899AABB')
    $list.Add('0x0101FFEEDDCCBBAA99887766554433221100FFE')

    Add-PnPContentTypesFromContentTypeHub -ContentTypes $list -ErrorAction Continue
```

References
Feedback [Enable taxonomy Feature by default in sharepoint sites created via Private/Shared channel](https://feedbackportal.microsoft.com/feedback/idea/86b3778c-8b6d-ee11-a81c-000d3ae46fcb)
