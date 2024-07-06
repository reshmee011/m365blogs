---
title: "Managing Service Principal Permission Requests using PowerShell"
date: 2024-04-03T08:25:29Z
tags: ["SPFx","PowerShell", "Permission", "Service Principal","CLIforM365","PnP PowerShell","SPO PowerShell","Declarative","Explicit"]
featured_image: '/posts/images/powershell-grant-entra-id-permissions/PendingRequests.png'
omit_header_text: true
draft: false
---

# Managing Service Principal Permission Requests using PowerShell

Permission to the "SharePoint Online Client" service principal can be granted either in declarative method within SPFx solutions or directly.
This post explores how to handle both declarative and direct permission grants using **SPO PowerShell** , **ClI for M365** and **PnP PowerShell**.

## PnP PowerShell

### Get all service principal permission grants

Gets the collection of permission grants for the "SharePoint Online Client" service principal using the cmdlet **Get-PnPTenantServicePrincipalPermissionGrants** 

```PowerShell
    Connect-PnPOnline -Url https://contoso-admin.sharepoint.com -Interactive
    Get-PnPTenantServicePrincipalPermissionGrants
```

### Add Microsoft Graph permission explicitly 

To explicitly add a permission to the SharePoint Online Entra ID app, such as the Mail.Read Microsoft Graph permission, use the **Grant-PnPTenantServicePrincipalPermissionGrant** cmdlet:

```PowerShell
Grant-PnPTenantServicePrincipalPermissionGrant --Resource 'Microsoft Graph' --Scope 'Mail.Read'
```

### Revoke Permission permission explicitly

Use the **Revoke-PnPTenantServicePrincipalPermission** to revoke a permission that was previously granted to the "SharePoint Online CLient" service principal.

```PowerShell
Revoke-PnPTenantServicePrincipalPermission --ObjectId 50NAzUm3C0K9B6p8ORLtIsQccg4rMERGvFGRtBsk2fA
```

### Get tenant service principal permission requests

Use the **Get-PnPTenantServicePrincipalPermissionRequests** cmdlet to list, approve, and deny permission requests that were submitted using the declarative approach:

```PowerShell
    Get-PnPTenantServicePrincipalPermissionRequests
```

###  Approve Tenant Service Principal Permission Request

```PowerShell
  Approve-PnPTenantServicePrincipalPermissionRequest --RequestId b4993f61-036d-465b-bc2f-6ef12696c26f
```

### Reject/Deny Tenant Service Principal Permission Request

```PowerShell
 Deny-PnPTenantServicePrincipalPermissionRequest --RequestId 4a36e6a4-61df-450c-a914-6c6efdaa2dd0
```

## CLI for M365

### Get a list of Service Principal permission grants

```powershell
    m365 login
    m365 spo serviceprincipal grant list
```

### Add a permission to the SharePoint Online Entra ID app

Add a permission to the SharePoint Online Entra ID app, such as the Mail.Read Microsoft Graph permission, use the grant add command:

```PowerShell
  m365 spo serviceprincipal grant add --resource 'Microsoft Graph' --scope 'Mail.Read'
```

### Remove permission with grant remove

Remove the permission using the grant revoke command:

```PowerShell 
m365 spo serviceprincipal grant list
```

Note down the id permissions need to be revoked.

```PowerShell 
m365 spo serviceprincipal grant revoke --id 50NAzUm3C0K9B6p8ORLtIsQccg4rMERGvFGRtBsk2fA
```

### Get all ServicePrincipal permission requests

Use the permissionrequest command to list, approve, and deny permission requests that were submitted using the declarative approach:

```PowerShell 
 m365 spo serviceprincipal permissionrequest list
```

Note the ID of the request to approve or reject/deny

### Approve serviceprincipal permissionrequest

```PowerShell 
 m365 spo serviceprincipal permissionrequest approve --id 4dc4c043-25ee-40f2-81d3-b3bf63da7538
```

### Reject serviceprincipal permissionrequest

```PowerShell 
 m365 spo serviceprincipal permissionrequest deny --id 4dc4c043-25ee-40f2-81d3-b3bf63da7538
```

## SPO PowerShell

To get a list of all the permissions granted to your tenant, use the following command:

```PowerShell
    Connect-SPOService -Url https://contoso-admin.sharepoint.com
    Get-SPOTenantServicePrincipalPermissionGrants
```

Review the response to find the grant you want to remove and get it's ID

[Get-SPOTenantServicePrincipalPermissionGrants](https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/get-spotenantserviceprincipalpermissiongrants?view=sharepoint-ps&wt.mc_id=MVP_308367)

![UnknownError](../images/powershell-get-sharing-links-sharepoint/ErrorGetGrants.png)

### Explicitly add permission to "SharePoint Online Client"
 
Add a permission to the SharePoint Online Entra ID app, such as the Mail.Read Microsoft Graph permission, using the **Approve-SPOTenantServicePrincipalPermissionGrant** cmdlet:

```PowerShell
 Approve-SPOTenantServicePrincipalPermissionGrant --Resource 'Microsoft Graph' --Scope 'Mail.Read'
```

[Approve-SPOTenantServicePrincipalPermissionGrant](https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/approve-spotenantserviceprincipalpermissiongrant?view=sharepoint-ps&wt.mc_id=MVP_308367)


### Revoke service principal permission grants

Revokes a permission that was previously granted to the "SharePoint Online Client" service principal.

```PowerShell
# list all serviceprincipal permission grants
Get-SPOTenantServicePrincipalPermissionGrants
```

Review the response to find the grant you want to remove and get it's ID

```PowerShell
Revoke-SPOTenantServicePrincipalPermission --ObjectId 50NAzUm3C0K9B6p8ORLtIsQccg4rMERGvFGRtBsk2fA
```

[Revoke-SPOTenantServicePrincipalPermission](https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/revoke-spotenantserviceprincipalpermission?view=sharepoint-ps&wt.mc_id=MVP_308367)

### Get Service Principal Permission requests in declarative way

Use the **Get-SPOTenantServicePrincipalPermissionRequests** cmdlet to list, approve, and deny permission requests that were submitted using the declarative approach:

```PowerShell
    Get-SPOTenantServicePrincipalPermissionRequests
```

![Tenant ServicePrincipal Permission Requests](../images/powershell-get-sharing-links-sharepoint/TenantServicePrincipalPermissionRequests.png)

Get the ID of the request to approve it...

**Approve Request**
Approves a permission request for the current tenant's "SharePoint Online Client" service principal

```PowerShell
Approve-SPOTenantServicePrincipalPermissionRequest --RequestId 4dc4c043-25ee-40f2-81d3-b3bf63da7538
```
**Deny Request**
Denies a permission request for the current tenant's "SharePoint Online Client" service principal.

```PowerShell
Deny-SPOTenantServicePrincipalPermissionRequest --RequestId 4dc4c043-25ee-40f2-81d3-b3bf63da7538
```

## References

[Beware of Declarative Permissions in SharePoint Framework Projects](https://www.voitanos.io/blog/consider-avoiding-declarative-permissions-with-azure-ad-services-in-sharepoint-framework-projects)