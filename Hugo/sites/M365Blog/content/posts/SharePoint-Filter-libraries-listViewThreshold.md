---
title: "How to Filter SharePoint Libraries to Return Less Than 5,000 Items"
date: 2024-07-16T06:32:50Z
tags: ["SharePoint Online","Search","Field","Column","Governance"]
featured_image: '/posts/images/SharePoint-Filter-libraries-listViewThreshold/LVT_morethan5000.png'
omit_header_text: true
draft: false
---

SharePoint Online has a list view threshold (LVT) of 5,000 items, which can cause performance issues if exceeded. 

![LVT more than 5000](../images/SharePoint-Filter-libraries-listViewThreshold/LVT_morethan5000.png)

This post provides a workaround through filtering libraries/lists to stay within this limit and avoid common problems associated with large lists.

## The Problem

When a SharePoint library exceeds the 5,000-item threshold, various issues can arise. These include:

### Inability to Browse Folders

* Navigating through folders becomes challenging and sometimes impossible through desktop office apps (Excel, Word,etc..)
![Unable to browse to folder](../images/SharePoint-Filter-libraries-listViewThreshold/LVT-EmptyWhenBrowsedTo.png)

* Incorrect Number of Files in Groups

![WrongNumberofFilesInGroups](../images/SharePoint-Filter-libraries-listViewThreshold/WrongNumberOfFiles_PerGrouping.png)

* Groups show as expanded even specified to be collapsed

Group Setting set as collapsed within the view
![Groups collapsed](../images/SharePoint-Filter-libraries-listViewThreshold/grouping_setting_collapsed.png)

However Groups load as expanded
![Groups Expanded](../images/SharePoint-Filter-libraries-listViewThreshold/Groups_Expanded.png)

## Solution

To manage large libraries effectively, you can filter items to ensure that your views return fewer than 5,000 items. Here are some practical steps to achieve this:

* Step 1: Create Indexed Columns
Create indexed columns to enhance performance. Commonly indexed columns include Modified, Created, Title, and ID.

* Step 2: Apply Filters
Apply filters to your views to limit the number of items displayed. For instance, filter items modified in the last two years:


- Go to the library and click on the gear icon (Settings).
- Select Library Settings.
- Under Views, choose a view to modify or create a new one.
   In the Filter section, set criteria to limit the items. For example, Show items only when the following is true: Modified is greater than or equal to [Today]-730.

![Filter files modified less than 2 years](../images/SharePoint-Filter-libraries-listViewThreshold/FilterModifiedlessThan2Years.png)

* Step 3: Use Folders and Subfolders
Organize items into folders and subfolders to distribute content evenly and prevent any single folder from exceeding the threshold.

## Conclusion

By following these steps, large libraries in SharePoint Online can be managed and avoid issues related to the 5,000-item list view threshold. This ensures smoother navigation, accurate file counts, and better overall performance.