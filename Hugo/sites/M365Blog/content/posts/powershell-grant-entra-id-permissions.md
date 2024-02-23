---
title: "Powershell Grant Entra Id Permissions"
date: 2024-02-21T06:48:06Z
draft: true
---


Using the CLI for Microsoft 365
To get a list of all the permissions granted to your tenant, use the following command:

m365 login
m365 spo serviceprincipal grant list
Then, to add a permission to the SharePoint Online Entra ID app, such as the Mail.Read Microsoft Graph permission, use the grant add command:

m365 spo serviceprincipal grant add --resource 'Microsoft Graph' --scope 'Mail.Read'
If you need to remove the permission, you can use the grant revoke command:

m365 spo serviceprincipal grant list
# review the response to find the grant you want to remove and get it's ID

m365 spo serviceprincipal grant revoke --id 50NAzUm3C0K9B6p8ORLtIsQccg4rMERGvFGRtBsk2fA
You can also use the permissionrequest command to list, approve, and deny permission requests that were submitted using the declarative approach:

m365 spo serviceprincipal permissionrequest list
# get the ID of the request...

# ... to approve it...
m365 spo serviceprincipal permissionrequest approve --id 4dc4c043-25ee-40f2-81d3-b3bf63da7538

# ... or reject it it...
m365 spo serviceprincipal permissionrequest deny --id 4dc4c043-25ee-40f2-81d3-b3bf63da7538
Using the SharePoint Online PowerShell
To get a list of all the permissions granted to your tenant, use the following command:

Connect-SPOService -Url https://voitanos-admin.sharepoint.com
Get-SPOTenantServicePrincipalPermissionGrants
Then, to add a permission to the SharePoint Online Entra ID app, such as the Mail.Read Microsoft Graph permission, use the Approve-SPOTenantServicePrincipalPermissionGrant cmdlet:

Approve-SPOTenantServicePrincipalPermissionGrant --Resource 'Microsoft Graph' --Scope 'Mail.Read'
If you need to remove the permission, you can use the grant revoke command:

Get-SPOTenantServicePrincipalPermissionGrants
# review the response to find the grant you want to remove and get it's ID

Revoke-SPOTenantServicePrincipalPermission --ObjectId 50NAzUm3C0K9B6p8ORLtIsQccg4rMERGvFGRtBsk2fA
You can also use the Get-SPOTenantServicePrincipalPermissionRequests cmdlet to list, approve, and deny permission requests that were submitted using the declarative approach:

Get-SPOTenantServicePrincipalPermissionRequests
# get the ID of the request...

# ... to approve it...
Approve-SPOTenantServicePrincipalPermissionRequest --RequestId 4dc4c043-25ee-40f2-81d3-b3bf63da7538

# ... or reject it it...
Deny-SPOTenantServicePrincipalPermissionRequest --RequestId 4dc4c043-25ee-40f2-81d3-b3bf63da7538

## References

[Beware of Declarative Permissions in SharePoint Framework Projects](https://www.voitanos.io/blog/consider-avoiding-declarative-permissions-with-azure-ad-services-in-sharepoint-framework-projects/?utm_source=convertkit&utm_medium=email&utm_campaign=%F0%9F%98%B1+Beware+of+Declarative+Permissions+in+SharePoint+Framework+Projects%21%20-%2013133930)