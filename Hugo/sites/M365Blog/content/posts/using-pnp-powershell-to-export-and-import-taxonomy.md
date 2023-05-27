---
title: 'Using PnP PowerShell to export and import Taxonomy'
date: Wed, 02 Nov 2016 20:43:35 +0000
draft: false
tags: ['CSOM', 'Debug PnP', 'Enable Trace on PnP', 'Export-SPOTaxonomy', 'Git', 'Import-SPOTaxonomy', 'PnP', 'PowerShell', 'Set-SPOTraceLog', 'SharePoint 2013', 'SharePoint 2016', 'SharePoint Online', 'Taxonomy', 'Terms', 'TermSet']
---

[PnP](https://github.com/OfficeDev/PnP-PowerShell/releases) provides export taxonomy cmdlet which allows to export all term groups, term sets, terms and child terms to a file with one line of code```
Connect-SPOnline -Url <siteurl> -CurrentCredentials 
Export-SPOTaxonomy -IncludeID -Path "C:\\temp\\Metadata\\terms.txt"
```[PnP](https://github.com/OfficeDev/PnP-PowerShell/releases) also provides an equivalent import taxonomy cmdlet to import all term groups, term sets, terms and child terms from a file with one line of code```
Connect-SPOnline -Url <siteurl> -CurrentCredentials
Import-SPOTaxonomy -Path "C:\\temp\\Metadata\\terms.txt"
```However if IncludeId switch parameter is used with the export, the import would fail if the [October 2016 PnP PowerShell release](https://github.com/OfficeDev/PnP-PowerShell/releases/tag/2.8.1610.1) is used. The error message appears```
Import-SPOTaxonomy : Exception on line 5: Failed to read from or write to database. 
Refresh and try again. If the problem persists, please contact the administrator
```I decided to enable Trace on the PnP to get debug messages using the Set-SPOTraceLog```
  Set-SPOTraceLog -On -Level Debug
```Debug messages appear to monitor progress of execution but was not helpful. ![errorimportingterm](https://reshmeeauckloo.files.wordpress.com/2016/11/errorimportingterm.png) I tried to google the message, found out that the error happened as the code is trying to create a term or term set with an unique identifier (GUID)  which already exists in the taxonomy store. I inspected the taxonomy file exported and found that terms having child terms were wrongly formatted.In the example below the term group EDRMS is repeated 3 times.```
EDRMS;#383eab6e-a944-430d-9a18-f8b872322c2c|EDRMS;#383eab6e-a944-430d-9a18-f8b872322c2c|Directorate;#a99db510-121a-407b-87ec-0b4911e59469|EDRMS;#383eab6e-a944-430d-9a18-f8b872322c2c|Directorate;#a99db510-121a-407b-87ec-0b4911e59469|Finance;#fbab08b3-bc7d-4363-a1c4-1f5d4a466b07|Credit and Collections;#0ec6cefe-c92c-4dfe-81d4-1fe56183f8a2
```The Export-SPOTaxonomy was wrongly formatting child terms in the export file. I raised the issue [532](https://github.com/OfficeDev/PnP-PowerShell/issues/) on PnP PowerShell project. However I was not sure how quick the issue was going to get fixed so decided [to debug PnP powershell command](https://veenstra.me.uk/2016/10/11/office-365-how-do-debug-pnp-powershell-commands/) and found that the issue was with the [core PnP project ](https://github.com/OfficeDev/PnP-sites-core). I fixed it with pull request [Taxonomy import export](https://github.com/OfficeDev/PnP-Sites-Core/pull/832) which got merged into pull request ["Fixed issue with subterms not exported correctly when using ExportAll" ](https://github.com/OfficeDev/PnP-Sites-Core/pull/833). With the PnP PowerShell November 2016 release, you will be able to use PnP to import and export taxonomy.