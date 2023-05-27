---
title: 'Migrate SharePoint Workflow 2013 as visio file'
date: Thu, 24 Dec 2015 16:34:37 +0000
draft: false
tags: ['SharePoint 2013', 'Workflow', 'Workflow SharePoint 2013 Migration']
---

I had to migrate a SharePoint Designer 2013 workflow to an environment. The workflow was exported to visio as a .vwi file. Unfortunately when the I tried to import the file using the "Import as visio" option from SharePoint Designer, the message was appearing "This workflow cannot be imported because it was created in SharePoint Designer for a different site, or the original workflow has been moved or deleted." I stumbled upon the blog below how to fix the migration issue https://aniketahmed.wordpress.com/2013/08/02/305/ Below are the steps

1.  Change the extension of the exported workflow from .vwi to zip
2.  Open the .zip file and remove **workflow.xoml.wfconfig.xml** file
3.  Rename .zip file back to .vwi
4.  From SharePoint Designer 2013, open the site where the workflow will be migrated.
5.  Click on Workflows tab.
6.  Click Import from Visio and browse to select the file and click Next.
7.  If the workflow is a reusable workflow, pick the reusable workflow option else if the workflow is a list workflow , pick the "List Workflow" option and select the list.
8.  Click Save.
9.  Click Publish to make the workflow available to the site.

If the workflow does not compile and throw the error "Errors were found while compiling the workflow. The workflow files were saved but cannot be run. Microsoft.SharePoint.SPException: App Management Shared Service Proxy is not installed...." The App Management Service application needs to be created and started. from Central Administration. Also the App Management Service proxy has to be added to the default proxy group by following the steps described in blog. http://www.c-sharpcorner.com/UploadFile/anavijai/error-app-management-shared-service-proxy-is-not-installed/