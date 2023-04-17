---
title: 'Duplicate Rule Detection Export and publish in CRM Dynamics 2015'
date: Tue, 05 Apr 2016 16:40:37 +0000
draft: false
tags: ['CRM', 'CRM PowerShell Publish Duplicate Rule', 'PowerShell', 'Uncategorized']
---

I have set up custom duplicate rule detection on some entities' fields on a DEV CRM 2015 instance. I needed to export the settings to be able to import into other environments. I used the link [Export Duplicate Detection Rules from one organization and import to another organization in CRM 2013](https://blogs.msdn.microsoft.com/arpita/2014/10/15/export-duplicate-detection-rules-from-one-organization-and-import-to-another-organization-in-crm-2013/) to export the rules. However the CRMDataMigrationUtility.exe SDK tool exports only unpublished duplicate detection rules which I had to do to be able to export the settings. This means the detection rule has to be published afterwards. Instead of doing it manually, I have written a PowerShell script which can publish a specific rule or all rules for an entity. The code is below. https://gist.github.com/reshmee011/737dfc6bfd15568098c2f71588bc7a2a After running the script the following message is displayed. ![DuplicateRuleSetResult](https://reshmeeauckloo.files.wordpress.com/2016/04/duplicaterulesetresult.png)