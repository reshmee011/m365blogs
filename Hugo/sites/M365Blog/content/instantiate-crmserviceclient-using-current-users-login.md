---
title: 'Instantiate CRMServiceClient using current user''s login'
date: Thu, 10 Nov 2016 18:12:33 +0000
draft: false
tags: ['Connection', 'CRM Dynamics 2015', 'CRMServiceClient', 'Current User's creden', 'Get-CrmConnection', 'PowerShell', 'Uncategorized']
---

The[Get-CrmConnection](https://technet.microsoft.com/en-us/library/dn756303.aspx) method can be used to  return connection to a CRM instance. The syntax to call the method is```
Parameter Set: OnLine
Get-CrmConnection \[-OnLineType\] <OnlineType> \[\[-Credential\] <PSCredential> \] 
\[-DeploymentRegion\] <String> \[\[-ProfileName\] <String> \] -OrganizationName <String> \[ <CommonParameters>\]

Parameter Set: OnPrem
Get-CrmConnection \[-ServerUrl\] <Uri> \[\[-Credential\] <PSCredential> \] 
\[-OrganizationName\] <String> \[\[-HomeRealmUrl\] <Uri> \] \[\[-ProfileName\] <String> \] 
\[ <CommonParameters>\]

Parameter Set: UIOnly
Get-CrmConnection \[\[-InteractiveMode\]\] \[ <CommonParameters>\]
```I wanted to get the crm connection with the current user's credentials without any prompts. [The first and second options required the object PSCredential which can't be created using logged current user's credentials](http://stackoverflow.com/questions/27449166/pscredentials-from-current-user). The third option with the switch InteractiveMode  displays a dialog box prompting to enter connection details. All three options were not appropriate for the requirement. The method returns the object [**Microsoft.Xrm.Tooling.**](https://msdn.microsoft.com/en-us/library/dn742291(v=crm.7).aspx)**[CrmServiceClient](https://msdn.microsoft.com/en-us/library/dn742291(v=crm.7).aspx).** From the msdn article, it can be constructed using the NetworkCredential object. The constructor's definition in C#```
public CrmServiceClient(
	NetworkCredential credential,
	AuthenticationType authType,
	string hostName,
	string port,
	string orgName,
	bool useUniqueInstance = false,
	bool useSsl = false,
	OrganizationDetail orgDetail = null
)
```  In PowerShell, the current user's credentials can be retrieved using \[System.Net.CredentialCache\]::DefaultNetworkCredentials. There is no way the \[System.Net.CredentialCache\]::DefaultNetworkCredentials can be converted to the PSCredential object. The below syntax can be used to create the Microsoft.Xrm.Tooling.Connector.CrmServiceClient using logged in user's credentials. Replace the variables $serverName, $serverPort, $organizationName with the respective values.```
 $crmConnection = New-Object \`
 -TypeName Microsoft.Xrm.Tooling.Connector.CrmServiceClient \`
 -ArgumentList (\[System.Net.CredentialCache\]::DefaultNetworkCredentials), 
 (\[Microsoft.Xrm.Tooling.Connector.AuthenticationType\]::AD),
 $serverName,
 $serverPort, 
 $organizationName, 
 $False,
 $False,
 (\[Microsoft.Xrm.Sdk.Discovery.OrganizationDetail\]$null)
```