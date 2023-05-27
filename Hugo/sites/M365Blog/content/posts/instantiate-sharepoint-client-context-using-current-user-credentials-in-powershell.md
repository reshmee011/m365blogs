---
title: 'Instantiate SharePoint Client Context using current user credentials in PowerShell'
date: Wed, 02 Nov 2016 19:10:01 +0000
draft: false
tags: ['ClientContext', 'CSOM', 'Current User Credentials', 'PowerShell', 'Security', 'SharePoint 2013 On Premises', 'SharePoint 2016 On Premises', 'Uncategorized', '[System.Net.CredentialCache]::DefaultNetworkCredentials']
---

In C# managed code, SharePoint Client Context can be created using System.Net.CredentialCache to pass logged in user credentials.```
ICredentials credentials = CredentialCache.DefaultCredentials;
clientContext.Credentials = credentials;
```I could not find anywhere how to achieve it in PowerShell. If using [PnP](https://github.com/OfficeDev/PnP-PowerShell/releases) PowerShell module, the switch parameter CurrentCredentials can be used with the cmdlet Connect-Online.```
Connect-SPOnline -Url “http://dev-sp-001a:1214/Teams/Legal” -CurrentCredentials 
$ctx= Get-SPOContext
```In most of my CSOM code without use of PnP I used to get current user name using [\[Environment\]::UserName](http://stackoverflow.com/questions/2085744/how-to-get-current-username-in-windows-powershell)```
[Environment]::UserName
```I used to prompt the current user to enter password```
$AdminPassword = Read-Host "Enter password: " -AsSecureString
```Lately I discovered I could use  system.net.credentialcache in PowerShell  to pass current user credentials when instantiating the ClientContext object.```
 $ctx=New-Object Microsoft.SharePoint.Client.ClientContext($siteUrl) 
 $Credentials = \[System.Net.CredentialCache\]::DefaultNetworkCredentials
 $ctx.Credentials = $Credentials;
 $web = $ctx.Web
 $ctx.Load($web);
 $ctx.ExecuteQuery();
```This means you can start Windows PowerShell as the user having appropriate permissions to the SharePoint Environment and run CSOM code without prompting credentials.