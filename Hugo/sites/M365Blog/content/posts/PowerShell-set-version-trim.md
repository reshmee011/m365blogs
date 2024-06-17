---
title: "PowerShell Set Version Trim"
date: 2024-06-17T06:43:47+01:00
draft: true
---

```PowerShell
PS C:\Users\reshm> set-pnptenantsite -Identity https://reshmeeauckloo.sharepoint.com/sites/Company311 -EnableAutoExpirationVersionTrim  $false  -ExpireVersionsAfterDays 50
Set-PnPTenantSite: EnableVersionExpirationSetting is disabled. Please run Set-SPOTenant to enable file version expiration feature.
```

```powershell
PS C:\Users\reshm> set-pnptenantsite -Identity https://reshmeeauckloo.sharepoint.com/sites/Company311 -ExpireVersionsAfterDays 50                 
Set-PnPTenantSite: You must specify a value for "EnableAutoExpirationVersionTrim" if "ExpireVersionsAfterDays", "MajorVersionLimit", "MajorWithMinorVersionsLimit" and "ApplyToNewDocumentLibraries" are specified.
```

**SPO PowerShell**

Are you sure you want to perform this action?
The setting for new document libraries takes effect immediately. Please run Get-SPOSite to check the newly set values on properties EnableAutoExpirationVersionTrim, ExpireVersionsAfterDays, MajorVersionLimit.The setting for existing document libraries may take 24 hours to take effect. Please run Get-
SPOSiteVersionPolicyJobProgress to check the progress. The setting for existing libraries does not trim existing versions to meet the newly set limits.



![Prompt to confirm update for setting](../images/PowerShell-set-version-trim/ConfirmPrompt_To_Update_Version_Setting.png)


Are you sure you want to perform this action?
The setting for new document libraries takes effect immediately. Please run Get-SPOSite to check the newly set values on properties EnableAutoExpirationVersionTrim, ExpireVersionsAfterDays, MajorVersionLimit. The setting for existing document libraries may take 24 hours to take effect. Please run Get-
SPOSiteVersionPolicyJobProgress to check the progress. The setting for existing libraries does not trim existing versions to meet the newly set limits.

![Prompt to confirm enabling setting at site](../images/PowerShell-set-version-trim/ConfirmPrompt_To_Enable_VersionTrim.png)