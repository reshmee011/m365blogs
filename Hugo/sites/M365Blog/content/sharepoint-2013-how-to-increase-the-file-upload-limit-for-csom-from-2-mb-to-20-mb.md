---
title: 'SharePoint 2013: How to increase the file upload limit for CSOM from 2 Mb to 20 Mb'
date: Tue, 23 Feb 2016 21:26:24 +0000
draft: false
tags: ['CSOM', 'PowerShell', 'SharePoint 2013']
---

In SharePoint 2013, the default file size limit set in Central Administration is for uploading single file manually in SharePoint libraries or using server side code. When I was trying to upload a file more than 2 MB using CSOM, the following error was thrown _The request message is too large. The server does not allow messages that are larger than 2097152 bytes._ I tried the fix mentioned in the post http://sharepoint.stackexchange.com/questions/83196/sharepoint-2013-client-object-model-file-larger-than-2-mb-exception```
$ws = [Microsoft.SharePoint.Administration.SPWebService]::ContentService $ws.ClientRequestServiceSettings.MaxReceivedMessageSize = 20971520 $ws.ClientRequestServiceSettings.MaxParseMessageSize  = 20971520 $ws.Update()
```I did reset IIS without much success. Stopping the SharePoint Timer Service before updating the values worked for me. The PowerShell script below stops timer service on each server on the farm, updates the client request service settings to increase size limit to 20 MB (20\*1024\*1024 = 20971520) and starts the timer service after the update.

\# Add Sharepoint Powershell Module

Set-ExecutionPolicy -ExecutionPolicy:Bypass -Force -Confirm:$false

Clear-Host

Add-PsSnapin Microsoft.SharePoint.PowerShell -ErrorAction SilentlyContinue

\[array\]$servers= Get-SPServer | ? {$\_.Role -eq "Application"}

foreach ($server in $servers)

{

    Write-Host "Restarting Timer Service on $server"

    $Service = Get-WmiObject -Computer $[server.name](http://server.name/) Win32\_Service -Filter "Name='SPTimerV4'"

    if ($Service -ne $null)

    {

        $Service.InvokeMethod('StopService',$null)

        Start-Sleep -s 8

        $ws = \[Microsoft.SharePoint.Administration.SPWebService\]::ContentService

        $ws.ClientRequestServiceSettings.MaxReceivedMessageSize = 20971520

        $ws.ClientRequestServiceSettings.MaxParseMessageSize = 20971520

        $ws.Update()

        $service.InvokeMethod('StartService',$null)

        Start-Sleep -s 5

        Write-Host -ForegroundColor Green "Timer Job successfully restarted on $server"

    }

    else

    {

        Write-Host -ForegroundColor Red "Could not find SharePoint Timer Service on $server"

    }

}