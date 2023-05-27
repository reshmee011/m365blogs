---
title: 'Delete all list Item and file versions from site and sub sites using CSOM and PowerShell'
date: Sat, 22 Apr 2017 10:51:46 +0000
draft: false
tags: ['Uncategorized']
---

The versions property is not available from client object model on the **ListItem** class as with server object model.  I landed on the article [SharePoint - Get List Item Versions Using Client Object Model](http://prasadtechtactics.blogspot.co.uk/2012/09/sharepoint-get-list-item-versions-using.html) that describes how to get a list item versions property using the method GetFileByServerRelativeUrl by passing the FileUrl property. The trick is to build the list item url as **"sites/test/Lists/MyList/30\_.000”** where 30 is the item id for which the version history needs to be retrieved. Using that information I created a PowerShell to loop through all lists in the site collection and sub sites to delete all version history from lists. The script below targets a SharePoint tenant environment. Please note that I have used script [Load-CSOMProperties.ps1](https://gist.github.com/glapointe/cc75574a1d4a225f401b) from blog post [Loading Specific Values Using Lambda Expressions and the SharePoint CSOM API with Windows PowerShell](https://www.itunity.com/article/loading-specific-values-lambda-expressions-sharepoint-csom-api-windows-powershell-1249) to help with querying object properties like Lambda expressions in C#. The  lines below demonstrate use of the  [Load-CSOMProperties.ps1](https://gist.github.com/glapointe/cc75574a1d4a225f401b)  file which I copied in the same directory where the script DeleteAllVersionsFromListItems.ps1 is.

*   Load the  [Load-CSOMProperties.ps1](https://gist.github.com/glapointe/cc75574a1d4a225f401b) file

_         Set-Location $PSScriptRoot_ _        $pLoadCSOMProperties=(get-location).ToString()+"\\Load-CSOMProperties.ps1"_ _       . $pLoadCSOMProperties_

*    Retrieve ServerRelativeURL property from file object

_ Load-CSOMProperties -object $file -propertyNames @("ServerRelativeUrl"); _ You can download the script from [technet](https://gallery.technet.microsoft.com/Delete-all-list-Item-8cce86df?redir=0). https://gist.github.com/reshmee011/371dcfd27b9e7afc1817b7b587fc279a