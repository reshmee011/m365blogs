---
title: "Effective permissions of end user using across sites PowerShell"
date: 2024-07-10T16:16:09Z
tags: ["SharePoint","PnP","PowerShell","Effective permissions" ,"Security","Permissions"]
featured_image: '/posts/images/powershell-get-effectivepermissions-for-site-list-item/GetEffectivePermission.png'
omit_header_text: true
draft: true
---

Permissions granted to a user either directly or through a shared link , SharePoint Group or M365 group to any list/libraries or file/item can be checked using the effectivepermissions endpoint

The script using the effecivepermissions endpoint to identify any permissions assigned to the site, list/library and item/file/folder level. 

{{< gist reshmee011 7e9e09d016e0384a6aff31d7d10804e8 >}}

