---
title: 'Instantiate SharePoint Client Context using current user credentials in PowerShell'
date: Wed, 02 Nov 2016 19:10:01 +0000
draft: false
tags: ['ClientContext', 'CSOM', 'Current User Credentials', 'PowerShell', 'Security', 'SharePoint 2013 On Premises', 'SharePoint 2016 On Premises', 'Uncategorized', '[System.Net.CredentialCache]::DefaultNetworkCredentials']
---


#### 
[M]( "mharley@healthwise.org") - <time datetime="2017-10-10 19:33:12">Oct 2, 2017</time>

The PnP cmdlets have been updated... Connect-SPOnline -Url “http://dev-sp-001a:1214/Teams/Legal” -CurrentCredentials $ctx= Get-SPOContext Should now be: Connect-PnPOnline -Url “http://dev-sp-001a:1214/Teams/Legal” $ctx= Get-PnPContext Thanks for posting!
<hr />
