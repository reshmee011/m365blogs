---
title: 'SharePoint, Approve Selected List Items'
date: Thu, 17 Dec 2015 18:57:00 +0000
draft: false
tags: ['JavaScript', 'JSOM', 'SharePoint 2013', 'SharePoint2013 List View Operations', 'Uncategorized']
---

SharePoint provides Out of the box JavaScript Libraries. The List Views functions are described in the post with an example how to apply some of the functions CountSelectedItems(ctx); return count of selected items in view SelectRowByIndex(ctx,1,true); selects first item in list view DeselectAllItems(ctx, tab.rows, false);deselects all items in view SelectRibbonTab("Ribbon.ListItem",true); displays the listitem ribbon The code below can be copied into a script editor web part on a list view page. https://gist.github.com/reshmee011/b36395eb08654baa4a8e6e3c9f83e6f4 I created a list "Test Approve Items" with columns "Status", and "Title" with 4 list items. ![ListViewOperations.JPG](https://reshmeeauckloo.files.wordpress.com/2015/12/listviewoperations.jpg) When "Select All" button is clicked, all items in the list view get selected. When "Deselect All" is clicked, all items in the list view get deselected. When "Show Ribbon" is clicked, the ribbon is shown ![ribbon.JPG](https://reshmeeauckloo.files.wordpress.com/2015/12/ribbon.jpg) When "Set Approve" is clicked after selected some items, the status changes to "Approve".