---
title: "SharePoint Search: Excluding Columns from Search Results"
date: 2024-05-01T06:32:50Z
tags: ["SharePoint Online","Search","Field","Column","Governance"]
featured_image: '/posts/images/SharePoint-Exclude-Columns-from-Search/SearchableColumns.png'
draft: false
---

# SharePoint Search: Excluding Columns from Search Results

SharePoint empowers users to manage and organize vast amounts of data efficiently. However, not all data within a SharePoint site might need to be searchable. Do you miss the functionality to control visibility of sensitive or irrelevant information in column.

## The Challenge

You might have encountered instances where you need certain columns in your SharePoint lists or libraries to be excluded from search results. This could be due to various reasons, such as confidentiality concerns or simply to streamline search output. However, achieving this goal can sometimes be less straightforward than expected.

## Exploring Solutions

### Using PowerShell

PnP PowerShell to update make the column non searchable, unfortunately using PnP PowerShell is failing.
![script-to-set-nocrawl-fail](../images/SharePoint-Exclude-Columns-from-Search/script-to-set-nocrawl-fail.png)

### Activating SharePoint Server Publishing Infrastructure

The link for searchable columns even as site owner from Site information > View all site settings as per article [Enable content on a site to be searchable](https://learn.microsoft.com/en-us/sharepoint/make-site-content-searchable) is missing by default.

![link-to-make-column-nonsearchable-missing](../images/SharePoint-Exclude-Columns-from-Search/link-to-make-column-nonsearchable-missing.png)

The solution involves activating the SharePoint Server Publishing Infrastructure feature. Follow these steps:

Navigate to Site Settings.
1. Under Site Actions, select Manage site features.
2. Look for SharePoint Server Publishing Infrastructure and activate it.
3. Upon activation, you'll gain access to additional settings, including the ability to manage searchable columns.

![Activate Publishing feature](../images/SharePoint-Exclude-Columns-from-Search/SPServerPublishingInfrastructure.png)

Link is visible

![SearchableColumns](../images/SharePoint-Exclude-Columns-from-Search/SearchableColumns.png)

#### Important Considerations

Before proceeding with the activation of the SharePoint Server Publishing Infrastructure feature, it's crucial to consider the following:

- Consultation: Always consult with your organisation's Microsoft resources or IT experts before making significant changes.
- Feature Scope: Determine whether you need "searchable columns" enabled for this specific site or across all sites. It's recommended to enable this feature only on relevant sites to avoid unnecessary complications.
- Permanent Activation: Once activated, refrain from deactivating this feature. Deactivation can lead to issues, as it generates various artifacts that might disrupt functionality if removed later.

## Conclusion

It would have ideal to have the functionality to exclude a column from search results without relying on the SharePoint Server Publishing Infrastructure. Activate the feature with caution if it is a requirement to exclude column from search results.

## References

[Enable content on a site to be searchable](https://learn.microsoft.com/en-us/sharepoint/make-site-content-searchable)