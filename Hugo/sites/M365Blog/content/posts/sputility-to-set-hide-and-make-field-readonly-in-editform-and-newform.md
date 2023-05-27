---
title: 'SPUtility to set , hide and make field readonly in EditForm and NewForm'
date: Thu, 24 Dec 2015 17:44:17 +0000
draft: false
tags: ['Uncategorized']
---

I was trying to work out how to set, hide and make fields read-only in EditForm.aspx and NewForm.aspx depending on conditions. I stumbled on SPUtility.js library from https://sputility.codeplex.com/ It allows to manipulate fields on EditForm.aspx and NewForm.aspx **Set a Text field's value** SPUtility.GetSPField('Title').SetValue('Test Set Field'); **Set a Text field's value to today's date and make it read only** vardate \= new Date(); SPUtility.GetSPField('Date Realised').SetDate(date.getFullYear(), date.getMonth() + 1, date.getDate()).MakeReadOnly(); **Hide a field from view** SPUtility.GetSPField('test hide').Hide();