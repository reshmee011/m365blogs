---
title: "PowerShell Date Format With adddays with an example to run a CLI for M365 cmdlet"
date: 2024-01-31T11:58:02Z
draft: true
---
# PowerShell Date Format With adddays with an example to run a CLI for M365 cmdlet

I was trying to run the following CLI for M365 cmdlet to retrieve 365 Audit log 


```PowerShell
$days= 3
 $activities += m365 purview auditlog list --contentType SharePoint --startTime (Get-date).adddays(-$days) --endTime (Get-date).adddays(-($days-1))  --output 'json' | ConvertFrom-Json
```


```PowerShell
 $activities += m365 purview auditlog list --contentType SharePoint --startTime ((Get-date).adddays(-$days) | Get-Date -uFormat '%Y-%m-%d') --endTime ((Get-date).adddays(-($days-1)) | Get-Date -uFormat '%Y-%m-%d') --output 'json' | ConvertFrom-Json
```
PowerShell Date Format With adddays with an example to run a CLI for M365 cmdlet


## References

[Get-Date](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/get-date?view=powershell-7.4??wt.mc_id=MVP_308367)
[AddDays with Format for dates][https://robdy.io/adddays-with-format-for-dates/]
[https://stackoverflow.com/questions/45689471/windows-iso-8601-timestamp]
