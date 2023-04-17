---
title: 'Search Display Template js file not created when HTML file for Item_ThreeLines is created'
date: Mon, 14 Mar 2016 23:41:15 +0000
draft: false
tags: ['HTML', 'JavaScript', 'Publishing Feature JavaScript ThreeLines', 'Search', 'Search DisplayTemplate js SharePoint Online SharePoint 365', 'SharePoint', 'SharePoint 2013', 'SharePoint Online', 'Uncategorized']
---

On a SharePoint Online site collection where Publishing feature was initially turned on and subsequently turned off, js file was not automatically created/updated whenever a display template was created/edited. We wanted to have a search content web part with three lines instead of two lines. We created a new display template by following the steps below.

*   Navigate to Site Settings>Master Pages>Display Templates>Content Web Parts.
*   Download the file Item\_TwoLines.html
*   Rename it to Item\_ThreeLines.html
*   Edit title to Three Lines and wherever reference to Line 2 is made add corresponding reference to Line 3

https://gist.github.com/reshmee011/7091aabc9038a41a2378

*   Import the file into the  Site Settings>Master Pages>Display Templates>Content Web Parts library

However the js file was not created automatically since publishing feature was turned off. The only option was to create the html display template in a site collection with publishing feature turned on, wait until the js is created automatically and migrate the js file and HTML file into the site collection where publishing feature is turned off. If the properties below are not set automatically, manually set the highlighted properties while uploading. ![Item_ThreeLinesjs](https://reshmeeauckloo.files.wordpress.com/2016/03/item_threelinesjs.png) The Item Template is accessible from any search web part added to the non publishing site collection ![3_lines_Content_By_Search_WP.png](https://reshmeeauckloo.files.wordpress.com/2016/03/3_lines_content_by_search_wp.png)