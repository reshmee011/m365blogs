---
title: "Exclude News Link from News WebPart SharePoint Online"
date: 2024-07-15T16:16:09Z
tags: ["SharePoint","Newslink","Search","News WebPart"]
featured_image: '/posts/images/SharePoint-News-WebPart-ExcludingNewsLinks/Sample.png'
omit_header_text: true
draft: true
---

# Exclude News Link from News WebPart SharePoint Online

If ever news link are within your SharePoint Online. News link are a useful feature to avoid duplication of news articles and a great way to promote an existing news post onto a different site. Imagine self news creation are enabled for the business across their own department/directorate sites and some of those need to be promoted to the whole organisation. A news link is the useful feature that can be leveraged. 

However the new webpart can show both the news links and original news article duplicating the exact news.

There is a column `OriginalSourceUrl` which stores the link of the source news article. 

![Original Source Url](../images/SharePoint-News-WebPart-ExcludingNewsLinks/OriginalSourceUrl_Column.png)

All SharePoint columns are indexed and  the managed property of the indexed column is `OriginalSourceUrIOWSTEXT`. 

To exclude news links from news web part, edit the webpart and select managed property 

![Original Source Url](../images/SharePoint-News-WebPart-ExcludingNewsLinks/OriginalSourceUrlOWSText.png)

Use the field `OriginalSourceUrIOWSTEXT` `Doesn't contain` <domainName> e.g. contoso


![Original Source Url](../images/SharePoint-News-WebPart-ExcludingNewsLinks/ExcludeNewsLinkFromNewsWebpart.png)