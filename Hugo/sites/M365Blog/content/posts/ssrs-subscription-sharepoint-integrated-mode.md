---
title: 'SSRS Subscription SharePoint Integrated Mode'
date: Thu, 24 Dec 2015 16:12:07 +0000
draft: false
tags: ['SharePoint 2013', 'SSRS', 'SSRS SharePoint Subscription Integrated mode', 'Uncategorized']
---

Subscription option is available on the SSRS report item to allow the run the report automatically. The user need to be logged in as site administrator to create subscription. Follow the steps below

*   Create a document library (named for example “Extracts” to store the report extracts.
*   Click on “Manage Subscriptions” from the drop down options available against the SSRS report item.

![ManageSubscri](https://reshmeeauckloo.files.wordpress.com/2015/12/managesubscri.jpg)

*   Click on “Add Subscription”.

![UpdateFields](https://reshmeeauckloo.files.wordpress.com/2015/12/updatefields.jpg)

*   Update the fields
    *   Delivery Extension = SharePoint Document Library
    *   Document Library = The document library created in first step.
    *   File Name = Name you want to give to the report exported.
    *   Output Format = You can choose Excel 2003 or some other formats.
    *   Overwrite existing file or create new version of file = Checked
    *   Delivery Event
        *   Click on “Configure” to create a schedule. Example of a schedule can be At 8:30 a.m. every Thursday.