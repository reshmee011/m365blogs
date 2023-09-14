---
title: "Teams Add-in Not Showing Within Outlook"
date: 2023-07-23T14:49:19+01:00
tags: ["Outlook", "Teams","Add-in"]
draft: false
featured_image: '/images/teams-addin-not-showing-in-outlook/TeamsAddInMissing.png'
---

# Teams Add-in Not Showing Within Outlook

The add-in for setting up a teams meeting within the outlook application was no longer showing. The add-in was not available to be enabled within Outlook as per [Troubleshoot the Teams Meeting add-in in Outlook for Windows](https://support.microsoft.com/en-gb/office/). Even after reinstalling Teams or creating a new profile within Outlook, it did not fix the issue. As a workaround I was using 

 1. Use Team calendar to create meeting
 2. Use Outlook Web App to create meetings

With Microsoft Support's help via a case we identified the issue was the missing following key:  

HKEY_CURRENT_USER\Software\Microsoft\Office\Outlook\Addins\TeamsAddin.FastConnect

The folder was exported from a working computer and then moved into the my damaged computer. The add-in was then available once Outlook had been restarted.

![Script Update](../images/teams-addin-not-showing-in-outlook/TeamsAddInMissing.png)

The fault was diagnosed by Microsoft Support as being an issue with a previous upgrade which removed the key.

References
[Troubleshoot the Teams Meeting add-in in Outlook for Windows](https://support.microsoft.com/en-gb/office/)
[Start Teams meeting in Outlook](https://techcommunity.microsoft.com/t5/microsoft-teams/start-teams-meeting-in-outlook/m-p/1335513)
