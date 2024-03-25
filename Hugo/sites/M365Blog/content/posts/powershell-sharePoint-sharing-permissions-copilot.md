---
title: "Harness sharing capabilities within your tenant and SharePoint site using PowerShell"
date: 2024-03-25T06:51:10Z
draft: true
---

# Harness sharing capabilities within your tenant and SharePoint site

This is why it is so important to use the permission models in SharePoint to ensure the right users or groups have the right access to the right content.

Users in your organization make choices that result in the oversharing of SharePoint Content. I have observed over the past few months that end users do not always pay attention to the permission of the site, library or folder where they are uploading files to.

Content, sometimes critical and confidential ends up in locations where other users may have access when they shouldn't. This may also include external users.

What leads to the oversharing of content is when the preference of sharing content is to large groups of people in a SharePoint location rather than individuals or smaller groups because it is just "easier".

A good first step in getting prepared to roll out Copilot for Microsoft 365 is to review site-level sharing controls and also to remove the "Everyone Except External Users" option in the people picker.

Copilot for Microsoft 365 will only EVER surface content to which individual users have at lease view / read permissions on.


## Set the Everyone Except External Users Option to False

Set-SPOTenant -ShowEveryoneExceptExternalUsersClaim $false

Set-PnPTenant -ShowEveryoneExceptExternalUsersClaim $false

[ShowEveryoneExceptExternalUsersClaim](https://pnp.github.io/powershell/cmdlets/Set-PnPTenant.html#-showeveryoneexceptexternalusersclaim)

```powerShell
Enables the administrator to hide the "Everyone except external users" claim in the People Picker. When users share an item with "Everyone except external users", it is accessible to all organization members in the tenant's Azure Active Directory, but not to any users who have previously accepted invitations.

The valid values are: True(default) - The Everyone except external users is displayed in People Picker. False - The Everyone except external users claim is not visible in People Picker.
```



This is an important topic to understand.

## At site level, disable default sharing 

## Restricts sharing to power users


## References
https://danielanderson.io/?ck_subscriber_id=2437811072
[ShowEveryoneExceptExternalUsersClaim](https://pnp.github.io/powershell/cmdlets/Set-PnPTenant.html#-showeveryoneexceptexternalusersclaim)
