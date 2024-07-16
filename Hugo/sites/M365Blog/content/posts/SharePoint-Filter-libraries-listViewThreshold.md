---
title: "How to Filter SharePoint Libraries to Return Less Than 5,000 Items"
date: 2024-07-15T06:32:50Z
tags: ["SharePoint Online","Search","Field","Column","Governance"]
featured_image: '/posts/images/SharePoint-Filter-libraries-listViewThreshold/LVT_morethan5000.png'
omit_header_text: true
draft: true
---


SharePoint Online has a list view threshold (LVT) of 5,000 items, which can cause performance issues if exceeded. This blog post will guide you through filtering your libraries to stay within this limit and avoid common problems associated with large lists.

The Problem
When a SharePoint library exceeds the 5,000-item threshold, various issues can arise. These include:
![LVT more than 5000](../images/SharePoint-Filter-libraries-listViewThreshold/LVT_morethan5000.png)


Issues
![Unable to browse to folder](../images/SharePoint-Filter-libraries-listViewThreshold/LVT-EmptyWhenBrowsedTo.png)

![WrongNumberofFilesInGroups](../images/SharePoint-Filter-libraries-listViewThreshold/WrongNumberOfFiles_PerGrouping.png)


![Groups Expanded](../images/SharePoint-Filter-libraries-listViewThreshold/Groups_Expanded.png)
![Groups collapsed](../images/SharePoint-Filter-libraries-listViewThreshold/grouping_setting_collapsed.png)

![Filter files modified less than 2 years](../images/SharePoint-Filter-libraries-listViewThreshold/FilterModifiedlessThan2Years.png)


Issues
Inability to Browse Folders

Navigating through folders becomes challenging and sometimes impossible.


Incorrect Number of Files in Groups

Grouping items may display incorrect file counts.


Groups Expanded and Collapsed Views

Expanded views of groups may show all files, while collapsed views do not reflect accurate counts.



Solution
To manage large libraries effectively, you can filter items to ensure that your views return fewer than 5,000 items. Here are some practical steps to achieve this:

Step 1: Create Indexed Columns
Create indexed columns to enhance performance. Commonly indexed columns include Modified, Created, Title, and ID.

Step 2: Apply Filters
Apply filters to your views to limit the number of items displayed. For instance, filter items modified in the last two years:



Go to the library and click on the gear icon (Settings).
Select Library Settings.
Under Views, choose a view to modify or create a new one.
In the Filter section, set criteria to limit the items. For example, Show items only when the following is true: Modified is greater than or equal to [Today]-730.
Step 3: Use Folders and Subfolders
Organize items into folders and subfolders to distribute content evenly and prevent any single folder from exceeding the threshold.

Step 4: Create Additional Views
Create multiple views with different filters to manage large libraries. For example, you can have views based on date ranges, document types, or departments.

Step 5: Enable Metadata Navigation
Enable metadata navigation and filtering to help users find items quickly without loading the entire list.

Conclusion
By following these steps, you can effectively manage large libraries in SharePoint Online and avoid issues related to the 5,000-item list view threshold. This ensures smoother navigation, accurate file counts, and better overall performance.