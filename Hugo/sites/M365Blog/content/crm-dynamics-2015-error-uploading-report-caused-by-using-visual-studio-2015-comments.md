---
title: 'CRM Dynamics 2015 "Error Uploading Report" caused by using Visual Studio 2015'
date: Thu, 17 Nov 2016 17:35:45 +0000
draft: false
tags: ['BIDS 2013', 'BIDS 2015', 'CRM Dynamics 2015', 'Error Uploading Report', 'SSRS']
---


#### 
[sherry]( "sherryswall@hotmail.com") - <time datetime="2017-12-11 00:07:49">Dec 1, 2017</time>

this is a good solution, however i'll add more to it. If you want to find out exactly what the error is. Logon to machine which has sql server instance for the CRM. Check event viewer and see the errors for MSCRMReporting and it will tell you what the error is. In this case it is the version incompatibility.
<hr />
