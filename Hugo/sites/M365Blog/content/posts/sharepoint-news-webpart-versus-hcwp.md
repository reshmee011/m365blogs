
---
title: "SharePoint Highlighted Content Web Part versus News Web Part: Author versus Editor"
date: 2024-07-25T00:39:57+01:00
tags: ["SharePoint", "News webpart", "Highlighted Content Web Part"]
featured_image: '/posts/images/sharepoint-news-webpart-versus-hcwp/news_managedProperty.png'
omit_header_text: true
draft: false
---

The Highlighted Content Web Part and the News Web Part in SharePoint can be used to display news on a page. Both webparts serve different purposes and have distinct features. One particular difference is the display of author in News Webpart and display of editor in the Highlighted Content Web Part.

The Highlighted Content Web Part displays editing details like editor and last updated date with the news information. Unfortunately there is no option to change it to the author name.

Let's explore the `Highlighted Content Web Part` versus `News webpart` on a high level view.

## Highlighted Content Web Part

Highlighted Content Web Part dynamically displays content from various sources like document libraries, sites, site collections, or all sites.

* Content Types: It can show documents, pages, news, videos, images, etc.
* Configuration: It allows filtering, sorting, and custom filters

![Managed Property filtering ](../images/sharepoint-news-webpart-versus-hcwp/HCWP.png)

* Layouts: Offers multiple layouts such as Cards, List, Carousel, Compact, and Filmstrip1.

* Use Case: Ideal for aggregating and displaying a variety of content types from multiple locations on a single page.

It can be used to display news content. One shortfall is it displays the editor instead of the news author.

![HCWP displays editoe](../images/sharepoint-news-webpart-versus-hcwp/hcwp_Editor.png)

## News Web Part

News web part is designed to display news articles and updates.

* Content Types: Shows news posts only.

* Configuration: Limited to filtering news posts by source and any metadata.

![Managed Property filtering ](../images/sharepoint-news-webpart-versus-hcwp/news_managedProperty.png)

* Layouts: Provides layouts like Top Story, Carousel, and List3.

![News Layouts](../images/sharepoint-news-webpart-versus-hcwp/news_layout.png)

* Use Case: Best for highlighting news and updates in a visually appealing format.

The good news is that it displays the author instead of the editor.

![News webpart displays author](../images/sharepoint-news-webpart-versus-hcwp/news_webpart.png)


Other options that can be considered to display news are custom web parts built using SPFX or configuring the PnP Modern WebParts.

## References

[How to use News WebPart in SP Online to read from different lists?](https://sharepoint.stackexchange.com/questions/287926/how-to-use-news-webpart-in-sp-online-to-read-from-different-lists)

[Use the Highlighted content web part](https://support.microsoft.com/en-us/office/use-the-highlighted-content-web-part-e34199b0-ff1a-47fb-8f4d-dbcaed329efd)

[Advanced Highlighted Content Web Part](https://learn.microsoft.com/en-us/microsoft-365/community/highlighted-content-web-part)