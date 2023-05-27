---
title: 'Office 365 - how to change settings to allow authenticated users to join meetings'
date: Fri, 19 Sep 2014 14:54:14 +0000
draft: false
tags: ['Uncategorized']
---

Today I was searching how to update lync settings to allow authenticated users to join meetings without waiting in meeting lobby. I stumbled across: http://technet.microsoft.com/en-us/library/gg398648.aspx The command below firsts get all meeting configuration settings in use where property AdmitAnonymousUsersByDefault is set to false and then update property PstnCallersBypassLobby to true. Get-CsMeetingConfiguration | Where-Object {$\_.AdmitAnonymousUsersByDefault -eq $False} | Set-CsMeetingConfiguration -PstnCallersBypassLobby $True