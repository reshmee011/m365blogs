---
title: "Get sharing link information using PnP PowerShell for files"
date: 2024-07-18T23:17:16+01:00
draft: true
---


```powershell
 $SharingLinks = [Microsoft.SharePoint.Client.ObjectSharingInformation]::GetObjectSharingInformation($Ctx, $Item, $false, $false, $false, $true, $true, $true, $true)
            $Ctx.Load($SharingInfo)
            $Ctx.ExecuteQuery()
```

The full script can be found on link [Oversight of Sharing Information in SharePoint sites using PowerShell and CSOM, REST and PnP PowerShell]

However it failed to return all sharing info excluding those blocking download of files.

For the particular folder, there were 7 links defined 
![links](../images/powershell_getsharinginformation_csom_graph/CSOM_sharinglink.png)

However using the CSOM option returns only 5 links. 

