---
title: 'Modern SharePoint Permission Management : update site members to contribute'
date: Sat, 11 Sep 2021 02:16:33 +0000
draft: false
tags: ['Uncategorized']
---

In SharePoint Online when a team site is automatically created, the Site Members and Site Owners SharePoint groups are created by default with Edit and Full Control permissions. However the permissions levels of Site Members can not be edited from its default Edit value to Contribute for example.

You can argue we can create a custom SharePoint group with contribute permissions to restrict users from being able to manage lists and libraries however users added to custom SharePoint groups won't benefit from other services in M365 ecosystem like teams, mailbox, etc... The best practice using SharePoint team sites is to use the M365 groups to edit permissions for full collaboration experience using connected services like teams. This add challenges to business who would like to have controlled contribute access to site members.

Though it might not be best practice to edit out of the box permission levels like edit , full control, etc... Permissions levels can be edit on the classic advanced permission settings page. In the example below I have edited the OOB Edit permission level to exclude "Manage Lists" and remove it Edit\_ExcludeManageLists.

[![](https://reshmeeauckloo.files.wordpress.com/2021/05/image.png?w=1024)](https://reshmeeauckloo.files.wordpress.com/2021/05/image.png)