---
title: 'REST or CSOM with JavaScript to find if user is in particular SharePoint group'
date: Fri, 19 Sep 2014 15:49:31 +0000
draft: false
tags: ['CSOM', 'JavaScript', 'REST', 'SharePoint', 'SharePoint 2013']
---

Initially I used REST call to find out whether the user is in a particular SharePoint group in SharePoint 2013. Though it works fine when the logged in user is an administrator or part of the queried group, the rest call to /\_api/web/sitegroups/getbyid(xx)/users keeps prompting for user name and password for non-administrator users throwing access denied exception at the end. After a while investigating, I could not find anything to resolve the rest call issue and I decided to find to alternatives way to find out if the user is in a particular group. I came across the example using CSOM and it works without prompting for user name and password. Using CSOM to check whether user is in group http://sharepoint.stackexchange.com/questions/86912/check-if-user-is-in-a-specified-group