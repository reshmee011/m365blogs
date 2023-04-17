---
title: 'Rest Api Generate Guest Link SharePoint Online'
date: Mon, 12 Sep 2016 17:36:15 +0000
draft: false
tags: ['CreateAnonymousLink', 'REST', 'SharePoint 2013', 'SharePoint Online']
---

The [CreateAnonymousLink](http://sharepointfieldnotes.blogspot.co.uk/2016/01/whats-new-in-sharepoint-2016-remote-api_28.html) method creates an anonymous link with option either to read or edit. It is quite useful to share documents with external users. The drawback is that anyone with the link can view/edit the document shared without sign-in. The method "[CreateAnonymousLink](http://sharepointfieldnotes.blogspot.co.uk/2016/01/whats-new-in-sharepoint-2016-remote-api_28.html) " accomplishes the same as "View Link - no sign-in required" and "Edit Link -no sign-in required" from the UI.![createanonymouslink](https://reshmeeauckloo.files.wordpress.com/2016/09/createanonymouslink.png) https://gist.github.com/reshmee011/1f08ae19491e5c094986dfdb55f29731 Once the link is generated, it can be sent by email to the relevant individuals.