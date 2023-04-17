---
title: 'Nintex SharePoint Online Create SharePoint Group with Contribute permissions'
date: Tue, 03 May 2016 14:53:03 +0000
draft: false
tags: ['Nintex', 'REST', 'REST POST Nintex SharePoint Online', 'SharePoint', 'SharePoint 2013', 'SharePoint Online']
---

Nintex Workflow has actions "Web Request" and "Call HTTP Web Service" which can be used to make calls to the REST API of SharePoint Online/2013 to automate creation of SharePoint groups in a site collection. Add the following steps to the workflow

1.  Create a Text variable GroupName and Use "Set Workflow Variable" to assign it to a value.![GroupNameVariable](https://reshmeeauckloo.files.wordpress.com/2016/04/groupnamevariable.png)
2.  Get request digest from targeted site collection. Please note that workflow app permissions need to amended to give site collection full control access.        ![App Step to get request digest](https://reshmeeauckloo.files.wordpress.com/2016/04/app-step-to-get-request-digest.png)
    *   Add "App Step" action
    *   Add "Web Request" action. Fill in the following fields
        *   URL: <siteURL> /\_api/ContextInfo
        *   Method : POST
        *   Content type : application/x-www-form-urlencoded
        *   Header Name (Key) : Accept     Header value: application/json;odata=verbose
            
            ![WebRequestToGetRequestDigest](https://reshmeeauckloo.files.wordpress.com/2016/04/webrequesttogetrequestdigest.png)
        *   Body : Choose "Content" option  and enter "{}" in text field
        *   UserName: Credentials who has access to targeted site
        *   Password: password of above UserName
        *   Store response content in : Create a variable "digest token"![WebRequestToGetRequestDigest_2](https://reshmeeauckloo.files.wordpress.com/2016/04/webrequesttogetrequestdigest_2.png)
    *   Add "Query XML" action. Fill in the following fields
        *   XML source: Choose "Content" and pick variable "digest token"
        *   XPath query: /d:GetContextWebInformation/d:FormDigestValue
            
        *   Return result as :Text
        *   Query result in : create a text variable strDigestInfo![QueryXMLRequestDigest](https://reshmeeauckloo.files.wordpress.com/2016/04/queryxmlrequestdigest.png)
    *   Add "Log to History" action. Print the strDigestInfo to make sure valid request digest are used  ![QueryXMLRequestDigest](https://reshmeeauckloo.files.wordpress.com/2016/04/queryxmlrequestdigest.png)
3.  Create SharePoint Group using REST API ‍/\_api/web/sitegroups  ![AppStepToCreateSpGroup](https://reshmeeauckloo.files.wordpress.com/2016/04/appsteptocreatespgroup.png)
    *   Add "App Step" action. Optionally add "Log to History List" action to log status of workflow.
    *   Add "Build Dictionary" action to build "Request Headers".
        *    Add the following key value pair as Text- content-type:  application/json; odata=verbose - X-RequestDigest : {Variable:strDigestInfo}
        *   Save Output into RequestHeaders![RequestHeaders](https://reshmeeauckloo.files.wordpress.com/2016/04/requestheaders.png)
    *   Add "Build Dictionary" action to build "Metadata"
        *    Add the following key/value pair- key: type    Type:Text  Value: SP.Group ![BuildMetadataForGroup](https://reshmeeauckloo.files.wordpress.com/2016/04/buildmetadataforgroup.png)
    *   Add "Build Dictionary" action to build "Parameters"
        *    Add the following key/value pair- Key: \_\_metadata   Type:Dictionary   Value: Workflow Variable GroupMetadata - Key: Description  Type: Text  Value: Group created automatically from Nintex workflow - Key: Title  Type:Text  Value:{Variable:GroupName}
        *   Assign Output to a dictionary variable GroupParameters ![BuildDictionaryGroupParameters](https://reshmeeauckloo.files.wordpress.com/2016/04/builddictionarygroupparameters.png)
    *   Add "HTTP Web Service"action to create group
        *   Fill in Address : <SiteURL>/\_api/web/sitegroups
        *   Set Request Type to "HTTP Post"
        *   Set Request Headers to dictionary variable RequestHeaders
        *   Set Request Content to dictionary variable GroupParameters
        *   Set Response Headers to new dictionary variable GroupResponseHeaders
        *   Set Response Content to new dictionary variable GroupResponseContent
        *   Set Response Status Code to new text variable GroupResponseStatusCode![WebRequestToCreateGroup](https://reshmeeauckloo.files.wordpress.com/2016/04/webrequesttocreategroup1.png)
    *   Log to History list the response![LogResponseOfGroupCreated](https://reshmeeauckloo.files.wordpress.com/2016/04/logresponseofgroupcreated.png)
4.  Retrieve SharePoint Group using REST API ‍ ‍/\_api/web/sitegroups/getbyname ![AppStepToGetGroupCreated](https://reshmeeauckloo.files.wordpress.com/2016/04/appsteptogetgroupcreated.png)
    *   Add "Add Step" action
    *   Optionally add "Log History List" action to log workflow status.
    *   Add a "Web Request" action![WebRequestToGetGroupCreated](https://reshmeeauckloo.files.wordpress.com/2016/04/webrequesttogetgroupcreated.png)
        *   Set URL to <SiteURL>/\_api/web/sitegroups/getbyname('‍[{Variable:GroupName}](https://wus-des-wfo.nintex.com//href=%22#")‍')?$select=Id
        *   Choose  Get for Method
        *   Click Add Header
        *   Set Header Key to "Accept"
        *   Set Header Value to "application/json;odata=verbose"
        *   Set UserName to user who has access to site collection
        *   Set Password to password of specified user
        *   Set Store response content in a Text variable GroupRequestIDXMLAdd
    *   Add "Log History List" action to log response GroupRequestIDXMLAdd from web request. The response is in JSON format. Nintex does not have a JSON parser action to retrieve value of property. Use "Regular Expression" action as described in the next step is used to retrieve Id value
    *   Add "Regular Expression" action![RegularExpressionToGetID](https://reshmeeauckloo.files.wordpress.com/2016/04/regularexpressiontogetid.png)
        *   Set String to ‍{Variable:GroupRequestIDXML}
        *   Set String Operation to Extract
        *   Set pattern to (?<="Id":).\*?(?=})
        *   Save Output to text variable strGroupIDCol
    *   Add "Get Item from Collection" action.![GetGroupIDFromCollection](https://reshmeeauckloo.files.wordpress.com/2016/04/getgroupidfromcollection.png)
        *   Set Target Collection to variable strGroupIDCol
        *   Set Index to 0
        *   Set Output to SPGroupId
    *    Add "Log History List" action to log strGroupIDCol
5.  Add Contribute Role to new Group
    *   Add App Step action  ![AppStepToAddContributeRoleToGroup](https://reshmeeauckloo.files.wordpress.com/2016/04/appsteptoaddcontributeroletogroup2.png)
    *   Add "Call HTTP Web Service" Action ![WebRequestToAddContributeRole](https://reshmeeauckloo.files.wordpress.com/2016/04/webrequesttoaddcontributerole.png)
        *   Set URL to <siteURL>‍/\_api/web/roleassignments/addroleassignment(principalid=‍[{Variable:SPGroupId}](https://wus-des-wfo.nintex.com//href=%22#")‍,roleDefId=1073741827)
        *   Set Request Type to "HTTP Post"
        *   Set Request Headers to dictionary "RequestHeaders"
        *   Set Response Headers to new dictionary variable RoleAssignResponseHeaders
        *   Set Response Content to new dictionary variable RoleAssignmResponseContent
        *   Set Response Status Code to new text variable RoleAssignmResponseStatusCode
    *   Add "Log to History List" action to log response of web request

Save and Publish the workflow. If the workflow runs succesfully ![WorkflowHistory](https://reshmeeauckloo.files.wordpress.com/2016/04/workflowhistory.png)