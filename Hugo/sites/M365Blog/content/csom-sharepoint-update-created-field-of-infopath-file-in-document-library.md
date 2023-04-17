---
title: 'CSOM SharePoint Update Created Field of InfoPath file in Document Library'
date: Wed, 09 Dec 2015 11:08:46 +0000
draft: false
tags: ['CSOM', 'SharePoint', 'SharePoint CSOM', 'Uncategorized']
---

While doing a bulk import of InfoPath documents into document library using a CSV file  the "Created" list out of the box date field is set to today's date and time. The "Created" date needed to be set to another date property "Date Raised". The InfoPath form can be uploaded following the steps described in the post https://reshmeeauckloo.wordpress.com/2015/11/26/programmatically-create-infopath-form-using-csom-sharepoint-2010/ The InfoPath item can be retrieved specifying the URL of the file and field "Created" set to another date. `Microsoft.SharePoint.Client.File newFile = clientContext.Web.GetFileByServerRelativeUrl(fileUrl); ListItem item = newFile.ListItemAllFields; newFile .CheckOut(); DateTime dateRaised; DateTime.TryParseExact ("24/04/1998", "d/M/yyyy",        CultureInfo.InvariantCulture, DateTimeStyles.None, out dateRaised); item["Created"] = dateRaised.ToString("yyyy-MM-dd"); item.Update(); newFile.CheckIn("Date Created Updated",CheckinType.MajorCheckIn); clientContext.ExecuteQuery();`