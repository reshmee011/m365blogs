---
title: 'SharePoint REST Get all items __next'
date: Fri, 04 Dec 2015 00:14:41 +0000
draft: false
tags: ['REST', 'SharePoint 2013', 'SharePoint Online']
---

When a REST query is run against a SharePoint list, not all items are returned if it contains more than 100 items. The default server paging is 100. It is a good practice to check the \_\_next property of the JSON results to make sure all items are returned. Below is a code snippet running from a SharePoint hosted add-in. `var url = appUrl + "/_api/SP.AppContextSite(@target)/Web/Lists/getByTitle('Repairs')/Items?$select=Title&@target='" + hostUrl + "'"; jQuery.getScript(scriptbase + "SP.RequestExecutor.js", getRepairDetails); function getRepairDetails() { var executor = new SP.RequestExecutor(appUrl); executor.executeAsync( { url: url, method: "GET", dataType: "json", headers: { Accept: "application/json;odata=verbose" }, success: function (data) { var response = JSON.parse(data.body); message.append(String.format("Retrieved {0} items", response.d.results.length)); message.append("  
"); if (response.d.__next) { url = response.d.__next; getRepairDetails(); } }, error: function (data, errorCode, errorMessage) { alert(errorMessage); } } );`