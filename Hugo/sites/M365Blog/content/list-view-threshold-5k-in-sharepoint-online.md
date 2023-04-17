---
title: 'List View Threshold 5k in SharePoint Online'
date: Fri, 07 May 2021 16:11:01 +0000
draft: false
tags: ['ListViewThreshold', 'LVT', 'SharePoint Online', 'Uncategorized']
---

[![](https://s-ssl.wordpress.com/i/spotify-badge.svg)](https://open.spotify.com/show/5W6pXcgUr64vmKuvexGtgI)

The list view threshold is 5000 in SharePoint Online and still can not be changed despite it has been raised in the User Voice for improvements.

I have created more than 5000 files in a library with the use of Power Automate to test what works and does not work.

I was surprised by navigating to the list or library I did not get any errors and everything seems to be working. However if I switch to classic view I get the message

"This view cannot be displayed because it exceeds the list view threshold (5000 items) enforced by the administrator.  
To view items, try selecting another view or creating a new view. If you do not have sufficient permissions to create views for this list, ask your administrator to modify the view so that it conforms to the list view threshold."

Fortunately "exit the classic view" made the error disappear. Views exceeding the list view threshold of 5000 k work in the modern experience. As you filter and sort by a column , indexes are automatically created in the list indexed columns. In the screenshot below the Severity column has been added automatically when I was sorting the library using the column.

[![](https://reshmeeauckloo.files.wordpress.com/2021/05/automaticallyindexedcolumns.png?w=960)](https://reshmeeauckloo.files.wordpress.com/2021/05/automaticallyindexedcolumns.png)

However automatic creation of indexes is limited to lists or libraries with less than 20 k items, otherwise you may encounter errors. In modern experience, automatic indexes are created with columns used for filtering and sorting in saved views or when sorting.

Also list view threshold error can happen if filtering or sorting by columns of type lookup, people and managed metadata.

Source: https://support.microsoft.com/en-us/office/manage-large-lists-and-libraries-b8588dae-9387-48c2-9248-c24122f07c59

"**Note:** Displaying 12 or more columns of the following types can cause a list view threshold error: people, lookup, and managed metadata. Displaying columns of other types will not. "