---
title: "Oversight of Sharing Information in SharePoint sites using PowerShell and CSOM, REST and PnP PowerShell"
date: 2024-05-05T06:57:18+01:00
tags: ["SharePoint","SharingLinks","PowerShell","Sites", "Security","Copilot for M365", "Governance","CSOM","REST", "PnP","Microsoft Graph" ]
featured_image: '/posts/images/powershell-get-sharing-links-sharepoint/report.png'
draft: false
toc: true
---

# Oversight of Sharing Information in SharePoint sites using PowerShell and CSOM, REST and PnP PowerShell

:::note
test notes
:::

Effective oversight of sharing links and sharing information are paramount to ensuring data security, compliance, and optimal collaboration experiences. 

As organisations migrate to M365 environments, they inherit powerful collaboration tools that facilitate seamless sharing of documents and resources. However, without proper governance, these capabilities can lead to unintended consequences such as data breaches, compliance violations, and loss of intellectual property.

Sharing is a powerful feature for collaboration. However depending on how items, files or folders are shared, a sharing link might be created or unique permissions on these items are created.
![ManageLinks](../images/powershell-get-sharing-links-sharepoint/ManageLinks.png)

For Copilot for M365 implementations, ensuring there is no oversharing is a critical aspect of safeguarding sensitive information and maintaining regulatory compliance.

An extract from [Announcing SharePoint advanced management innovations for the AI and Copilot era](https://techcommunity.microsoft.com/t5/sharepoint-premium-blog/announcing-sharepoint-advanced-management-innovations-for-the-ai/ba-p/4126366?WT.mc_id=5005104&ck_subscriber_id=2673998245)

"With Copilot and AI, security has become a concern. Not because Copilot allows people to access anything more than they could previously; it just allows them to find information they have access to faster. A term used sometimes in SharePoint is "Security by obscurity"; hide stuff and hope people don't find it. That doesn't work as well anymore with Copilot. It surfaces data more broadly and quickly."

Refer to [Microsoft Copilot for Microsoft 365 - best practices with SharePoint](https://learn.microsoft.com/en-us/SharePoint/sharepoint-copilot-best-practices?wt.mc_id=MVP_308367).

However, manually tracking down these links across multiple sites and libraries can be a daunting task. There are few options available, each with its own limitations.

1. [Report on file and folder sharing in a SharePoint site](https://learn.microsoft.com/nl-nl/sharepoint/sharing-reports?wt.mc_id=MVP_308367) allows to report on external sharing per site only.
 
2. [Data access governance reports for SharePoint sites](https://learn.microsoft.com/en-us/sharepoint/data-access-governance-reports?wt.mc_id=MVP_308367) provide a very high level view of the sharing links without details on which folder or item the sharing link was created. At the time of writing this blog post in April/May 2024, Data Access Governance reports show new sharing links in the past 28 days, which makes it very difficult to find content that was shared using an Everyone Except External Users or Anyone links more than a month ago.te
3. [Use sharing auditing in the audit log](https://learn.microsoft.com/en-us/purview/audit-log-sharing?view=o365-worldwide&tabs=microsoft-purview-portal#how-to-identify-resources-shared-with-external-users?wt.mc_id=MVP_308367) is restricted to the filter criteria used, which may not retrieve all sharing links.

I have not been able to determine why unique permissions are sometimes created without generating a sharing link. One of the scenerios I noticed is the sharing link is only created when "Copy Link" is clicked on.

![CopyLinks](../images/powershell-get-sharing-links-sharepoint/linkcopied.png)

There have been changes to sharing as per MC706173, please refer for more info[M365 Changelog: (Updated) Invite people you choose in the Share control in Microsoft 365 apps](https://petri.com/microsoft-changelog/m365-changelog-invite-people-you-choose-in-the-share-control-in-microsoft-365-apps/) though the full changes are not clear.

So reporting on sharing links might not be enough and look into drilling into unique permissions applied to each file, folder or item. I have provided two flavours of scripts using CSOM, PnP cmdlets and REST API

## PowerShell Script overview using the CSOM method GetObjectSharingInformation

This PowerShell script automates the process of retrieving sharing information in SharePoint Online on file, folder or item empowering administrators to efficiently audit sharing. The script was adapted from 
[How to get a list of shared links in a SharePoint Online document library? Any PowerShell or other way?](https://learn.microsoft.com/en-us/answers/questions/992330/how-to-get-a-list-of-shared-links-in-a-sharepoint) using the CSOM method GetObjectSharingInformation

{{< gist reshmee011 24bf63cc81ccae604b563a7c312f001f >}}

Some explanation of the script

### Parameters

$tenantUrl: Input parameter to specify the URL of the SharePoint tenant collection.
$dateTime: Current date and time in a specific format for generating unique file names.
$directorypath: Path to the directory containing the script.
$fileName: Name of the CSV file for storing the sharing links report.

### Connection to SharePoint Online

The script connects to the SharePoint Online environment using the Connect-PnPOnline cmdlet, prompting the user to enter the tenant collection URL interactively.

### Retrieval of Sharing Links

The getSharingLink function retrieves sharing links for a given object (site or list item) using the ObjectSharingInformation class. It collects relevant information such as the URL of the shared link, access type, expiration date, and more.

### Exclusion of Certain Libraries

The script excludes specified libraries (e.g., system libraries) from the sharing links audit using the $ExcludedLists array.

### Iteration through Sites and Libraries

Using Get-PnPTenantSite, the script retrieves a list of SharePoint sites matching predefined criteria (e.g., Communication sites, Teams sites). It then iterates through each site and its document libraries to gather sharing link information.

### Exporting the Report

Finally, the collected sharing link data is exported to a CSV file with a timestamped filename in the designated directory.

## Using REST EndPoint

The REST endpoint that can be used to return only sharing links, refer to [Externally Sharing – GetSharingInformation REST API](https://sharepoint.stackexchange.com/questions/309303/get-all-shared-links-to-a-specific-user-in-tenant)

 (siteUrl)/_api/web/Lists(listid)/GetItemById(itemid)/GetSharingInformation?$Expand=permissionsInformation,pickerSettings"

{{< gist reshmee011 be60dcf4e73c250aa408441423835c62 >}}

### Output of the REST EndPoint call
![RESTOutput](../images/powershell-get-sharing-links-sharepoint/RESTOutput.png)

## Using Get-PnPFileSharingLink and Get-PnPFolderSharingLink

The Get-PnPFileSharingLink and Get-PnPFolderSharingLink returns all sharing links created only at file and folder level. 
![ActualLinks](../images/powershell-get-sharing-links-sharepoint/ActualSharingLinks.png)

There is no cmdlet to return sharing link at item level yet.

The cmdlets Get-PnPFileSharingLink and Get-PnPFolderSharingLink uses the Graph EndPoint

https://graph.microsoft.com/v1.0/sites/{SiteId}/drives/{VroomDriveID}/items/{VroomItemID}/permissions?$filter=Link ne null

{{< gist reshmee011 f9ff790bd6ff00824f30e8b46d39d6dc >}}

However using Get-PnPFileSharingLink and Get-PnPFolderSharingLink return only sharing links and not all sharing instances without a link.

Example output

![PnPOutput](../images/powershell-get-sharing-links-sharepoint/PnP_Output.png)

## Conclusion

This PowerShell script can help with compliance or simply optimise security practices for effective SharePoint management. There are different ways to retrieve sharing information. The CSOM method wins returning all sharing information while the REST and PnP cmdlets (Graph) return only sharing links.

## References

[Microsoft Copilot for Microsoft 365 - best practices with SharePoint](https://learn.microsoft.com/en-us/SharePoint/sharepoint-copilot-best-practices?wt.mc_id=MVP_308367)

[Microsoft Purview data security and compliance protections for Microsoft Copilot](https://learn.microsoft.com/en-us/purview/ai-microsoft-purview?wt.mc_id=MVP_308367)

[How to get a list of shared links in a SharePoint Online document library? Any PowerShell or other way?](https://learn.microsoft.com/en-us/answers/questions/992330/how-to-get-a-list-of-shared-links-in-a-sharepoint?wt.mc_id=MVP_308367)

[How to check links shared in SharePoint Online or OneDrive for Business?](https://erik365.blog/2023/03/16/how-to-check-links-shared-in-sharepoint-online-or-onedrive-for-business/#:~:text=In%20the%20SharePoint%20Online%20report,on%20your%20Microsoft%20365%20users)

[Data access governance reports for SharePoint sites](https://learn.microsoft.com/en-us/sharepoint/data-access-governance-reports?wt.mc_id=MVP_308367)

[Use sharing auditing in the audit log](https://learn.microsoft.com/en-us/purview/audit-log-sharing?view=o365-worldwide&tabs=microsoft-purview-portal#how-to-identify-resources-shared-with-external-users?wt.mc_id=MVP_308367) 
