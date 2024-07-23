---
title: "Power Automate get user details"
date: 2024-07-23T09:53:05+01:00
tags: ["Power Automate","Get User Details"]
featured_image: '/posts/images/powerauomate-get-user-details/GetAuthorDetails.png'
omit_header_text: true
draft: true
---

1. Http SharePoint Action 

Site Address: @triggerOutputs()?['body/SiteUrl']
Method: Get
Uri : /_api/web/_api/Web/GetUserById(body('Parse_Page_Details_JSON')?['d']?['AuthorId'])

![Get User Details](../images/powerauomate-get-user-details/GetAuthorDetails.png)

2. Parse JSON Actin
Content:
Schema:

3. Use User Details in any subsequent actions