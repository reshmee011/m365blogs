---
title: 'Convert UTC Date returned from SharePoint API to Local time using JavaScript and C#'
date: Thu, 19 Nov 2015 12:44:37 +0000
draft: false
tags: ['REST', 'SharePoint 2013', 'SharePoint Online']
---

SharePoint Online stores all date fields in UTC format and convert it to appropriate time zone whenever it is retrieved by a user. However when using SharePoint API, the date returned is in UTC format and have to be converted to local time. Using the JavaScript function below the local time can be returned. self.toLocalDate = function (utc\_date) { //The getTimezoneOffset() value is in minutes var offset = new Date().getTimezoneOffset(); //offset will be in minutes. Add/subtract the minutes from your date return utc\_date.setMinutes(utc\_date.getMinutes() + offset); } In C#, the function below can be used System.TimeZone.CurrentTimeZone.ToLocalTime(date) Further articles to help understand time zone in SharePoint Online http://www.techgrowingpains.com/2012/05/sharepoint-time-zone-confusion-2/ https://yetanothersharepointblog.wordpress.com/2013/07/14/timezone-issues-when-working-with-dates-in-sharepoints-rest-services/