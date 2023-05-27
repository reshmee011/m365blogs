---
title: 'Prevent members in ''Modern'' M365 group connected team sites to delete libraries'
date: Sat, 24 Jul 2021 23:06:38 +0000
draft: false
tags: ['Uncategorized']
---

It is not possible for M365 to change the permissions of members from Edit to Contribute to prevent deletion or uncontrolled creation of lists and libraries. The option to "Edit User Permissions" is greyed out.

[![](https://reshmeeauckloo.files.wordpress.com/2021/07/image.png?w=577)](https://reshmeeauckloo.files.wordpress.com/2021/07/image.png)

One option would be to create a custom SharePoint Group for contributors but anyone added to the custom SharePoint Group would not be part of the M365 group which means they won't be able to contribute via Teams or any other M365 services.

One undocumented approach is to rename the Edit role to "Edit without manage lists" and amend the permissions to remove the ability to create or delete lists and libraries.

1.  Click on Permissions Levels from the classic advanced permissions page
2.  Click on Edit Permission
3.  Update the Name to "Edit without manage lists" and uncheck "Manage Lists"

[![](https://reshmeeauckloo.files.wordpress.com/2021/07/image-1.png?w=1024)](https://reshmeeauckloo.files.wordpress.com/2021/07/image-1.png)

All users in the members group will be granted "Edit without manage lists" permissions.

In a non m365 connected team site Members can easily be changed to contributors as per the blog post [Prevent document library deletion | CaPa Creative Ltd](https://capacreative.co.uk/2018/09/17/prevent-document-library-deletion/)