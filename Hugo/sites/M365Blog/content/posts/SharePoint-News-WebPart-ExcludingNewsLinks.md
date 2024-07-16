---
title: "Exclude News Links from News WebPart in SharePoint Online"
date: 2024-07-15T16:16:09Z
tags: ["SharePoint","Newslink","Search","News WebPart"]
featured_image: '/posts/images/SharePoint-News-WebPart-ExcludingNewsLinks/ExcludeNewsLinkFromNewsWebpart.png'
omit_header_text: true
draft: false
---

SharePoint Online offers the useful feature of news links to avoid duplication of news articles and promote existing posts across different sites. This is especially beneficial when news creation is enabled for various departments or directorates, allowing them to share significant news organization-wide.

However, the News WebPart might display both the news links and the original articles, resulting in duplicated content.

To manage this, you can utilize the `OriginalSourceUrl` column, which stores the link to the source news article.

![Original Source Url](../images/SharePoint-News-WebPart-ExcludingNewsLinks/OriginalSourceUrl_Column.png)

All SharePoint columns are indexed, and the managed property for the OriginalSourceUrl column is OriginalSourceUrIOWSTEXT.

To exclude news links from the News WebPart, follow these steps:

* Edit the News WebPart.
* Select the managed property OriginalSourceUrIOWSTEXT.
![Original Source Ows Test](../images/SharePoint-News-WebPart-ExcludingNewsLinks/OriginalSourceUrlOWSText.png)
* Set the field to Doesn't contain and specify the domain name (e.g., contoso).

![Original Source Url](../images/SharePoint-News-WebPart-ExcludingNewsLinks/ExcludeNewsLinkFromNewsWebpart.png)

This will filter out the news links from the News WebPart, ensuring only the original news articles are displayed.