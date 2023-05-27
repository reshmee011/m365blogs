---
title: 'Import/Export Columns and Content Types using PnP PowerShell'
date: Fri, 24 Jun 2016 11:52:05 +0000
draft: false
tags: ['PnP', 'PnP Migration Site Columns Content Types SharePoint 2013 SharePoint Online', 'PowerShell', 'SharePoint 2013', 'SharePoint Online', 'Uncategorized']
---

It is useful to migrate Site Columns and Content Types from one Site Collection to another under the following scenarios maintaining the original GUIDS.

1.  SharePoint On Premise environment to SharePoint Online
2.  Within SharePoint Online, migrate to another tenant
3.  If Content Hub Feature is not used, migrate the contents to another site collection
4.  Keep a list for governance/change tracking purpose, i.e. checking in TFS

This article covers export/import of site columns and content types using [SharePointPnP2016PowerShell](https://github.com/OfficeDev/PnP-PowerShell/releases) commands which needs to be installed to be able to use the relevant commands. The following can be used to connect to SharePoint 2013 Site collection. Connect-SPOnline -Url $SiteUrl -CurrentCredentials If script is run for SharePoint 2013 premises and you are logged in as the user already have access to the  site collection, use the parameter -CurrentCredentials. If the parameter is omitted you will be prompted to enter your credentials which you might need to do if connecting to SharePoint Online. The site columns and content types need to be exported as XML first before being imported back to a different site collection.

### Export Site Columns to XML

Use command Get-SPOField to retrieve all fields from the site collection. If the field is in a group, e.g. "Custom Group", the field schemaXML property is copied to the XML file which is created in the location the script is run. The script can be downloaded from tech net article [Export Column using SharePointPnP2016PowerShell to XML](https://gallery.technet.microsoft.com/Export-Column-using-5070ce1d)

### Export Site Content Types to XML

Use command Get-SPOContentType to retrieve all content types in the sitecollection. If the field is in a grou p,e.g. "Portfolio DB", the content type schemaXML property is copied to the XML file which is created in the location the script is run. The script can be downloaded from tech net article  [Export Content Types using SharePointPnP2016PowerShell to XML](https://gallery.technet.microsoft.com/Export-Content-Types-using-108696f6)

### Import Site Columns from XML

Use command   Add-SPOFieldFromXml -FieldXml $\_.OuterXml  to add site column to site collection. For some site columns exported, there was a version tag which was causing the error " The object has been updated by another user since it was last fetched"  to be thrown when imported. The version tag was removed from the XML using method RemoveAttribute. $\_.RemoveAttribute("Version") There are some challenges associating the termset to a managed metadata site column. There are some additional steps needed - Import the SharePoint client dlls Add-Type -Path "c:\\Program Files\\Common Files\\microsoft shared\\Web Server Extensions\\15\\ISAPI\\Microsoft.SharePoint.Client.dll" Add-Type -Path "c:\\Program Files\\Common Files\\microsoft shared\\Web Server Extensions\\15\\ISAPI\\Microsoft.SharePoint.Client.Runtime.dll" Add-Type -Path "c:\\Program Files\\Common Files\\microsoft shared\\Web Server Extensions\\15\\ISAPI\\Microsoft.SharePoint.Client.Taxonomy.dll" - Query the site column added to site collection Get-SPOField  -Identity $\_.ID - Cast the field object into Microsoft.SharePoint.Client.Taxonomy.TaxonomyField $taxonomyField= \[Microsoft.SharePoint.Client.ClientContext\].GetMethod("CastTo").MakeGenericMethod(\[Microsoft.SharePoint.Client.Taxonomy.TaxonomyField\]).Invoke($Context, $field) - Update the properties  SsId and TermSetId $taxonomyField.SspId = \[Guid\]$termStoreIdEle.Value.InnerText; $taxonomyField.TermSetId = $termId; The script can be downloaded from tech net article  [Import Site Columns using SharePointPnP2016PowerShell to XML](https://gallery.technet.microsoft.com/Import-Columns-using-2716bc12)

### Import Content Types from XML

Use command Get-SPOContentType -Identity $\_.Name  to retrieve content type by name in the sitecollection. If the content type does not exist , use command below to create content type Add-SPOContentType -ContentTypeId $\_.ID -Group $\_.Group -Name $\_.Name -Description $\_.Description To add the fields to content type use Add-SPOFieldToContentType -Field $\_.Name -ContentType $spContentType.Name If the field is required use the switchParameter -Required Add-SPOFieldToContentType -Field $\_.Name -ContentType $spContentType.Name   -Required If the field is hidden use the  switchParameter -Hidden Add-SPOFieldToContentType -Field $\_.Name -ContentType $spContentType.Name   -Hidden The script can be downloaded from tech net article  [Import Content Types using SharePointPnP2016PowerShell to XML](https://gallery.technet.microsoft.com/Import-Content-Types-using-66fcf69c) The full code is shown below https://gist.github.com/reshmee011/cb83653a8926dfd84495b78fbb43c1f2