---
title: 'ustom Layout Results Page to allow sorting and filtering by property'
date: Tue, 24 Nov 2015 00:17:48 +0000
draft: false
tags: ['Result source', 'Search', 'SharePoint 2013', 'SharePoint Online', 'SharePoint Search Display Template ResultsPage', 'Uncategorized']
---

Content query webpart does not give the same flexibility as search results page to allow user to filter and sort by property. SharePoint Online provides out of the box display template which can be used in results page. However if custom property needs to be displayed or a custom display format is needed , then a new search result page can be created. Assuming news items need to be displayed in the following format with filters Country and Subject. ![SearchResultPage](https://reshmeeauckloo.files.wordpress.com/2015/11/searchresultpage.jpg)

1.  Map Managed Metadata fields to managed properties                  Country and Subject are managed metadata fields. Two crawled properties are created per managed metadata fields. For example field country has two crawled properties ows\_taxId\_Country and ows\_Country. The ows\_FieldName format stores the friendly label and is not mapped by default. The latter can be mapped to the out of the box refinable string mapped properties.

![RefinableStrings](https://reshmeeauckloo.files.wordpress.com/2015/11/refinablestrings.jpg) 2.  Create  Result Source for the news ![NewsResultSource.png](https://reshmeeauckloo.files.wordpress.com/2015/11/newsresultsource.png) 3.Create a new result display template "Item\_NewsArticle.html" by following the guide http://www.ableblue.com/blog/archive/2013/06/05/introduction-to-sharepoint-2013-display-templates/ The Item\_NewsArticle.js gets generated automatically. [Download Item\_NewsArticle.html](https://onedrive.live.com/redir?resid=DAD16970816E89BC!82206&authkey=!AL0yNVL5Ga4mOnE&ithint=file%2chtml)

1.  On the news article results page, ensure the search results web part uses the template "New Article Item" and the query use the " All News Article" result source created in previous steps.