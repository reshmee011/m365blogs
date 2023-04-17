---
title: 'Nintex Workflow SharePoint Online Create Sub Site using Call HTTP Web Service Action'
date: Sat, 28 Nov 2015 22:36:21 +0000
draft: false
tags: ['Nintex', 'Nintex Workflow', 'SharePoint Online', 'Uncategorized']
---

In SharePoint Online, Nintex Workflow has an action named Call HTTP Web Service. The action can be used to call SharePoint REST API, i.e. /\_api/web/webinfos/add endpoint to create web site. The workflow app needs to be given full control at the site collection. The app permissions for workflow can be configured by following the steps below (refer to https://msdn.microsoft.com/en-us/library/office/jj822159(v=office.15).aspx for more info)

1.  Allow workflow to use app permissions

 a. Navigate to site settings

 b. Under Site Actions, click Manage site features

c. Locate the Workflows can app permissions and click Activate if it is not activated yet.

![WorkflowCan use app permissions](https://reshmeeauckloo.files.wordpress.com/2015/11/workflowcan-use-app-permissions.jpg)2\. Allow workflow to use app permissions

3.Navigate to site settings> site app permissions

[![Site App Permissions](https://reshmeeauckloo.files.wordpress.com/2015/11/site-app-permissions.jpg?w=300)](https://reshmeeauckloo.files.wordpress.com/2015/11/site-app-permissions.jpg) Note down the workflow app id which is between i:0i.t|ms.sp.ext| and @

4\. Navigate to <SiteURL>/\_layouts/15/appinv.aspx

5\. Enter workflow app id  and click on Lookup button 6. Enter the following in Permission Request XML

<AppPermissionRequests> <AppPermissionRequest Scope="[http://sharepoint/content/sitecollection](http://sharepoint/content/sitecollection "http://sharepoint/content/sitecollection")" Right="FullControl" /> </AppPermissionRequests>    [![GrantPermissions](https://reshmeeauckloo.files.wordpress.com/2015/11/grantpermissions.jpg)](https://reshmeeauckloo.files.wordpress.com/2015/11/grantpermissions.jpg)

7.Click on Create 8.Click on Trust the app.

The logic of the Nintex Workflow was built using the [Consuming the SharePoint 2013 REST Service from SharePoint Designer](http://blog.vgrem.com/2014/05/08/consuming-the-sharepoint-2013-rest-service-from-sharepoint-designer/) that describes how to configure Call HTTP Web Service Action in order to create web site. Create a new Nintex Workflow against a custom list with Title column only using the Nintex Workflow Designer

1.  Use "Set Workflow Variable" action to create and assign variable RequestURL <SiteURL>/\_api/web/webinfos/add

![1.Set Workflow Variable RequestURL](https://reshmeeauckloo.files.wordpress.com/2015/11/1-set-workflow-variable-requesturl1.jpg)

2. Use "Set Workflow Variable" action to create and assign Title

![4.Set Workflow Variable Title](https://reshmeeauckloo.files.wordpress.com/2015/11/4-set-workflow-variable-title1.jpg)

3\. Use "Set Workflow Variable" action to create and assign SiteURL the expected URL

![3.Set Workflow Variable SiteUrl](https://reshmeeauckloo.files.wordpress.com/2015/11/3-set-workflow-variable-siteurl1.jpg)

4. Use "Set Workflow Variable" action to create and assign variable SiteTemplate

![2.Set Workflow Variable Site Template](https://reshmeeauckloo.files.wordpress.com/2015/11/2-set-workflow-variable-site-template.jpg)

5. Use action "Build Dictionary" to build Dictionary RequestHeaders.

![5.Build Dictionary RequestHeader](https://reshmeeauckloo.files.wordpress.com/2015/11/5-build-dictionary-requestheader.jpg) Add key pair values as described in the table below

Name

Type

Value

 Accept

Text

 application/json;odata=verbose

 Content-Type

Text

 application/json;odata=verbose

Table 1. Request Headers for a RequestHeaders dictionary variable 6. Use action "Build Dictionary" to build Dictionary MetaData.![6.Build Dictionary Metadata](https://reshmeeauckloo.files.wordpress.com/2015/11/6-build-dictionary-metadata.jpg) Add key pair values as described in the table below

Name

Type

Value

 type

Text

 SP.WebInfoCreationInformation

Table 2.  Metadata dictionary variable 7. Use action "Build Dictionary" to  build Dictionary Parameters. ![7.Build Dictionary Parameters](https://reshmeeauckloo.files.wordpress.com/2015/11/7-build-dictionary-parameters.jpg) Add key pair values as described in the table below

Name

Type

Value

 \_\_metadata

Dictionary

Workflow Variable:Metadata

 Url

Text

Workflow Variable:Title

 Title

Text

Workflow Variable:Title

 Description

Text

Workflow Variable:Title

 Language

Text

 Value:1033

 WebTemplate

Text

 Workflow Variable: sts

 UseIniquePermissions

 Boolean

 Value:No

Table 3.  Parameters dictionary variable 8. Create App Step ![9.1App Step](https://reshmeeauckloo.files.wordpress.com/2015/11/9-1app-step.jpg) 9. Add and Configure the HTTP Web Service ![9.Call HTTP Web Service](https://reshmeeauckloo.files.wordpress.com/2015/11/9-call-http-web-service.jpg)

Property Name

Value

 Address

Set value to variable RequestUrl that was created in step 1

 RequestType

Set value to HTTP POST

 RequestHeaders

Set value to dictionary variable RequestHeaders that we’ve constructed to store request header properties in step 5

 ResponseContent

Create a new dictionary variable named ResponseContent to store the body of this HTTP request

ResponseHeaders

Create a new dictionary variable ResponseHeaders to store the response headers of this HTTP request.

ResponseStatusCode

Create a new text variable named ResponseStatusCode  to store the response status code of this HTTP request.

Table 4. HTTP Web Service Action configuration Publish the workflow against the list and run workflow against an item in the list to create a subsite with the title specified. ![Workflow](https://reshmeeauckloo.files.wordpress.com/2015/11/workflow.jpg)