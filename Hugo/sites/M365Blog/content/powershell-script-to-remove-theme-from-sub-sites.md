---
title: 'PowerShell Script to Remove Theme from Sub Sites'
date: Fri, 15 Jan 2016 16:48:56 +0000
draft: false
tags: ['PowerShell', 'Project Server 2013', 'SharePoint 2013', 'Uncategorized']
---

I had a requirement to apply a custom css file to a SharePoint collection on which there was a custom composed look with background image applied. I used the composed look option to reset the theme to a default without background image on the main site collection URL. Unfortunately sub sites were still displaying the background image. I tried setting the theme of the Sub Site using function [ApplyTo](http://sharepoint.stackexchange.com/questions/78872/how-do-you-apply-a-theme-with-powershell-in-2013) or [ApplyTheme](https://msdn.microsoft.com/EN-US/library/jj251358(v=office.15).aspx) without luck. I finally decided to remove theme from the sub sites which works the same way as resetting to default. The PowerShell script to remove theme from sub sites is $url = "http://inm.test/local" $site = Get-SPSite -Identity $url foreach ($web in $site.AllWebs) {   Write-Host "Web Title: " $web.Title   $web.allowunsafeupdates = $true   $theme = \[Microsoft.SharePoint.Utilities.ThmxTheme\]::RemoveThemeFromWeb($web,$false)   $web.Update()   $web.allowunsafeupdates = $false   $web.Dispose() } $site.Dispose(); The background image on all sub sites is cleared.