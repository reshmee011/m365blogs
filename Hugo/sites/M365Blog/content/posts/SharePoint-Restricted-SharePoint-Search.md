---
title: "Restrict certain SharePoint sites from tenant search and Copilot for M365 using PowerShell"
date: 2024-06-08T09:55:41+01:00
tags: ["SharePoint Restricted Search","Copilot for M365","Governance","PowerShell","PnP PowerShell"]
featured_image: '/posts/images/SharePoint-Restricted-SharePoint-Search/rss_enabled.png'
draft: false
---

# Restrict certain SharePoint sites from tenant search and Copilot for M365 using PowerShell

Excluding certain SharePoint sites from search would mean the contents from the excluded sites won't be available to M365 tenant search and **Copilot for M365** using Restricted SharePoint Search feature.

The downsides using this feature are:

1. **Limited Findability**: By excluding certain SharePoint sites from search, you limit the findability of data. Users would need to know the specific sites where the data resides in order to search for it. This can hinder efficient data retrieval and records management.


2. **Limited Number of Allowed Sites**: As of the time of writing the blog post, only 100 sites can be added to the list of allowed sites. This might not be sufficient for large tenants with requirements to add more than 100 sites to the allowed sites.* 

The other option is to disable site indexing to prevent results from appearing in all contextual search (site,library,list or tenant) which does not have any limit to the number of sites to be excluded. Refer to the script [Enable/Disable Search Crawling on Sites and Libraries](https://pnp.github.io/script-samples/spo-enable-disable-search-crawling/README.html?tabs=pnpps). 

3. **No Selective Disabling**: Currently, there's no option to selectively disable the feature for specific applications like Copilot for M365 while keeping it enabled for tenant search. This lack of granular control can be a limitation for some organizations.

4. **License Requirement**: To use Restricted SharePoint Search, the tenant needs to have a Copilot for M365 license enabled. Without this license, using any feature would result in a "license not assigned" message.

![license required](../images/SharePoint-Restricted-SharePoint-Search/rss-licenserequired.png)


Until the right controls are in place, sensitive sites may be excluded by turning on **Restricted SharePoint Search** using [PowerShell Scripts for Restricted SharePoint Search](https://learn.microsoft.com/en-us/sharepoint/restricted-sharepoint-search-admin-scripts?wt.mc_id=MVP_308367) and using an allowed list to specify which sites to allow **Copilot for M365** to use.


Please refer to [Microsoft Syntex - SharePoint Advanced Management overview](https://learn.microsoft.com/en-us/sharepoint/advanced-management?wt.mc_id=MVP_308367) and [Manage SharePoint Premium Settings Using PowerShell to protect data in Copilot for M365 Rollout](https://reshmeeauckloo.com/posts/powershell-sharepoint-premium-settings/) for applying controls on content for governance.

1. **Enable Restricted SharePoint Search Mode**

Restricted SharePoint Search can be enable on the tenant using the cmdlets.

**SPO PowerShell**

```PowerShell
Set-SPOTenantRestrictedSearchMode -Mode Enabled 
```

To check the state of Restricted SharePoint Search

```powershell
Get-SPOTenantRestrictedSearchMode 
```

![rss state not enabled](../images/SharePoint-Restricted-SharePoint-Search/RestrictedSearchModeNotSet.png)

**PnP PowerShell**

```PowerShell
Set-PnPTenantRestrictedSearchMode -Mode Enabled 
```

To check the state of Restricted SharePoint Search

```powershell
Get-PnPTenantRestrictedSearchMode 
```

![rss state mode enabled](../images/SharePoint-Restricted-SharePoint-Search/PnPRSSMode.png)

2. **Add the sites to the allowed list via csv file or in list array of urls**

Sample csv to use with no headers and only one column for the site url 

```csv
https://reshmeeauckloo.sharepoint.com/sites/Company311
https://reshmeeauckloo.sharepoint.com/sites/contosoportal
```

**SPO PowerShell**

```Powershell
Add-SPOTenantRestrictedSearchAllowedList  -SitesListFileUrl C:\Users\admin\Downloads\UrlList.csv
```

```Powershell
Add-SPOTenantRestrictedSearchAllowedList -SitesList  @("https://reshmeeauckloo.sharepoint.com/sites/Company311","https://reshmeeauckloo.sharepoint.com/sites/contosoportal") 
```

**PnP PowerShell**

This cmdlet is dependent on the PR [new cmdlet for Add-PnPTenantRestrictedSearchAllowedList](https://github.com/pnp/powershell/pull/3993) to be merged. 

```Powershell
Add-PnPTenantRestrictedSearchAllowedList  -SitesListFileUrl C:\Users\admin\Downloads\UrlList.csv
```

```Powershell
Add-PnPTenantRestrictedSearchAllowedList -SitesList  @("https://reshmeeauckloo.sharepoint.com/sites/Company311","https://reshmeeauckloo.sharepoint.com/sites/contosoportal") 
```

The result of enabling Restricted SharePoint Search will result in the message appearing at the top of **Copilot for M365**.

![rss enabled](../images/SharePoint-Restricted-SharePoint-Search/rss_enabled.png)

**Your organisation's admin has restricted Copilot from accessing certain SharePoint sites. This limits the content Copilot can search and reference when responding to your prompts. Learn more**

The same message appears under for tenant level search "Search in SharePoint" with files not within the allowed list returned.

![search restricted for tenant ](../images/SharePoint-Restricted-SharePoint-Search/Rss_OrganisationSearch.png)

Click on [Learn More](https://support.microsoft.com/en-gb/office/why-am-i-seeing-limited-results-29c9d8da-30d0-4ec2-a41f-5f2d93b509e4)

This content below is copied from the **Learn More** above.

```html
Your search and your Copilot experience are showing you limited results because your organization’s administrator has decided to restrict the SharePoint sites that appear in the organization-wide search results and Copilot experiences. When your administrator makes this decision, only the following organization content will show up in your organization-wide search and your Copilot experiences:

A curated list of  SharePoint sites set up by your organization’s administrator

Content from SharePoint sites you frequently visit

Your files from OneDrive, chats, emails, and calendars you have access to 

Files that were shared directly with you 

Files you’ve viewed, edited, or created 

To have more results included in search and Copilot experiences, please get in touch with your administrator to provide access to additional sites.
```
Please note this does not prevent referencing a file not from the allowed list of sites for **Copilot for M365** to reason on it, it just won't surface it otherwise.

![file referenced from outside the allowed list of sites](../images/SharePoint-Restricted-SharePoint-Search/FilereferencedFrom_not_allowedListofsites.png)

3. **Retrieves existing list of URLs in the allowed list**

**SPO PowerShell**
```powershell
 Get-SPOTenantRestrictedSearchAllowedList
```

**PnP PowerShell**

This cmdlet is dependent on the PR [New cmdlet for Get-PnPTenantRestrictedSearchAllowedList](https://github.com/pnp/powershell/pull/3997) to be merged.

```powershell
 Get-PnPTenantRestrictedSearchAllowedList
```
If Restricted SharePoint Search is not enabled

4. Removes sites from the allowed list

**SPO PowerShell**

```powershell
Remove-SPOTenantRestrictedSearchAllowedList -SitesList <List[string]> [<CommonParameters>]
```

## Potential issues

If the Url is specified as [https://reshmeeauckloo.sharepoint.com/sites/Company311](https://reshmeeauckloo.sharepoint.com/sites/Company311) as mentioned in the docs [PowerShell Scripts for Restricted SharePoint Search](https://learn.microsoft.com/en-us/sharepoint/restricted-sharepoint-search-admin-scripts?wt.mc_id=MVP_308367), error message may be thrown.

**Add-PnPTenantRestrictedSearchAllowedList: Could not find site with URL [https://reshmeeauckloo.sharepoint.com/sites/Company311](https://reshmeeauckloo.sharepoint.com/sites/Company311). Verify if the site exists and URL is valid.**

![Could not find url](../images/SharePoint-Restricted-SharePoint-Search/rss_add_couldnotfindsitewithurl.png)

Keep it simple with just the URL, otherwise you will be presented with message.

## Conclusion

Restricted SharePoint Search can be a powerful tool for organizations that need to control which SharePoint sites are accessible to tenant search and Copilot for M365. It provides a layer of security and governance by allowing administrators to specify an allowed list of sites.

However, it's important to be aware of the limitations and potential issues. These include limited findability, a cap on the number of allowed sites, lack of selective disabling, and license requirements. Additionally, the process of enabling and managing this feature requires the use of PowerShell scripts, which may require a certain level of technical expertise.

Restricted SharePoint Search needs to be used as a temporary measure for Copilot for M365 rollout and disable it once appropriate controls are in place for governance, compliance and security. 

## References

[Prepare content for Microsoft Copilot w/ SharePoint Content Governance | M365 Community Conference](https://www.youtube.com/watch?v=B5VRu9q6sZ8&t=404s)

[PowerShell Scripts for Restricted SharePoint Search](https://learn.microsoft.com/en-us/sharepoint/restricted-sharepoint-search-admin-scripts?wt.mc_id=MVP_308367)

