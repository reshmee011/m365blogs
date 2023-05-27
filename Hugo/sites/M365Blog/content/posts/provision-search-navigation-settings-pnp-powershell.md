---
title: 'Provision Search Navigation Settings PnP PowerShell'
date: Fri, 30 Sep 2016 16:51:24 +0000
draft: false
tags: ['Add-SPONavigationNode', 'PnP', 'PnP PowerShell SharePoint Search Navigation Settings', 'PowerShell', 'Remove-SPONavigationNode', 'SharePoint 2013', 'SharePoint Online']
---

I was investigating ways how to provision a search centre site with all the configurations automatically. I tried to package all configurations from an already configured Search Centre using "Save site as template", [however it did not work as Publishing Features are activated in a Search Centre.](https://support.microsoft.com/en-gb/kb/2492356) There are different configuration elements in a search centre site

*   Search Navigation Settings
*   Custom Search Display Templates
*   Custom Search Results Pages which can be created using the function [Add-PublishingPage ](http://sharepoint.stackexchange.com/questions/127922/exception-calling-executequery-with-0-arguments-the-remote-server-return)
*   Configure web parts in the search results page
    *   Refinement WebPart
    *   Search Results WebPart

This blog post focuses on updating the Search Navigation Settings. I have been experimenting using [PnP](https://github.com/OfficeDev/PnP-PowerShell/releases) method Add-SPONavigationNode. Prior to PnP September 2016 release, when the method was used, it was throwing the error message _[Add-SPONavigationNode : Invalid URI: The format of the URI could not be determined.](https://github.com/OfficeDev/PnP-PowerShell/issues/396)_ Fortunately it was fixed with PnP September 2016 release. I have created the method Update-SearchNavigationNode() to handle the update .

1.  The Search Navigation Nodes are retrieved using Id 1040
2.  All Existing Navigation Nodes are deleted using PnP method Remove-SPONavigationNode
3.  The Search Navigation Nodes are added using method  Add-SPONavigationNode  providing the values for parameters Title, Url,Location and Header.

`Add-SPONavigationNode -Title "site directory " -Url "/search/Pages/SomePageName.aspx" -Location "SearchNav" -Header "SearchNav"`

https://gist.github.com/reshmee011/ff202242804f11f363c9cec3eda0aedd