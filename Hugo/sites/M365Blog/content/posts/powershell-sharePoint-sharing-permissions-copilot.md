---
title: "Harness sharing capabilities within your tenant and SharePoint site using PowerShell"
date: 2024-03-25T06:51:10Z
draft: true
---

# Harness sharing capabilities within your tenant and SharePoint site



## Set the Everyone Except External Users Option to False

Set-SPOTenant -ShowEveryoneExceptExternalUsersClaim $false

Set-PnPTenant -ShowEveryoneExceptExternalUsersClaim $false

[ShowEveryoneExceptExternalUsersClaim](https://pnp.github.io/powershell/cmdlets/Set-PnPTenant.html#-showeveryoneexceptexternalusersclaim)

```powerShell
Enables the administrator to hide the "Everyone except external users" claim in the People Picker. When users share an item with "Everyone except external users", it is accessible to all organization members in the tenant's Azure Active Directory, but not to any users who have previously accepted invitations.

The valid values are: True(default) - The Everyone except external users is displayed in People Picker. False - The Everyone except external users claim is not visible in People Picker.
```

There are other tenant wide level settings but I think it would have been great to be able to set those at the library or site level and have site power users have greater control over them 


This is an important topic to understand.

## At site level, disable default sharing 

## Restricts sharing to power users

## shring links
SHARING LINKS from power automate

. Create a sharing link for a file or folder
· Specify:
. The Item ID
· Link Type (View and edit, View only)
· Link Scope (Anonymous, People in
your org)
· Link Expiration date (optional)
· Link is then available in Dynamic
Content to be able to send via email or
Teams
. Make sure that sharing is enabled from
the admin level in Microsoft 365.

## References
https://danielanderson.io/?ck_subscriber_id=2437811072
[ShowEveryoneExceptExternalUsersClaim](https://pnp.github.io/powershell/cmdlets/Set-PnPTenant.html#-showeveryoneexceptexternalusersclaim)
[SPOSharingSettings](https://microsoft365dsc.com/resources/sharepoint/SPOSharingSettings/)
