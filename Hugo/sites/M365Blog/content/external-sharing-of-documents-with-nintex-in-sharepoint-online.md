---
title: 'External Sharing of Documents with Nintex in SharePoint Online'
date: Tue, 20 Sep 2016 16:53:55 +0000
draft: false
tags: ['ExternalSharing Nintex SharePoint Online', 'Nintex', 'REST', 'SharePoint Online', 'Uncategorized']
---

The "Office 365 update item permissions" step available in Nintex Workflow updates item permissions. However if external users are given permissions using this step, it fails with message "failed to resolve user ". The[ issue with Office 365 update item permissions](https://community.nintex.com/thread/12513) is that it does not allow updating permissions for external users.

With the [SharePoint 2016 Remote API Sharing](http://sharepointfieldnotes.blogspot.co.uk/2015/12/whats-new-in-sharepoint-2016-remote-api.html) with external users can be achieved using method UpdateDocumentSharingInfo from class SPDocumentSharingManager which fortunately can be called using REST.

The tenant and site collection sharing property needs to be updated to either option "Allow external users who accept sharing invitations and sign in as authenticated users" or "Allow sharing with all external users, and by using anonymous links" to be able to add external users. If the first option is selected the user needs to be given access to the site.

![allowsharingwithallexternalusers](https://reshmeeauckloo.files.wordpress.com/2016/09/allowsharingwithallexternalusers.png) The workflow will work against a list having columns Email and SharePointLink ![listcolumnconfig](https://reshmeeauckloo.files.wordpress.com/2016/09/listcolumnconfig.png) The first step in the workflow is to get the RequestDigest from targeted site collection. ![App Step to get request digest](https://reshmeeauckloo.files.wordpress.com/2016/04/app-step-to-get-request-digest.png) The steps to add in the workflow are as below

*   Add "App Step" action
*   Add "Web Request" action. Fill in the following fields
    *   **URL**: <siteURL> /\_api/ContextInfo
    *   **Method** : POST
    *   **Content type** : text/xml![contextinforequest](https://reshmeeauckloo.files.wordpress.com/2016/09/contextinforequest.png)
    *   **Body** : Choose "Content" option  and enter "{}" in text field
    *   **UserName**: Credentials who has access to targeted site
    *   **Password**: password of above UserName
    *   **Store response content in** : Create a variable "RequestDigest"
*   Add "Query XML" action. Fill in the following fields
    *   **XML source**: Choose "Content" and pick variable "digest token"
    *   **XPath query**: /d:GetContextWebInformation/d:FormDigestValue![getrequestdigest](https://reshmeeauckloo.files.wordpress.com/2016/09/getrequestdigest.png)
    *   **Return result as** :Text
    *   **Query result in** : create a text variable strDigestInfo
*   Add "Log to History List" step. Print the strDigestInfo to make sure valid request digest are used.

Please note that workflow app permissions need to amended to give site collection full control access. The second step is to call the **UpdateDocumentSharingInfo** REST API method to grant appropriate access to the external user. ![appsteptoaddreadpermissiontodocument](https://reshmeeauckloo.files.wordpress.com/2016/09/appsteptoaddreadpermissiontodocument.png)

*   Add "App Step" action
*   Add Build Dictionary step.  Rename is to "Request Headers".
    *   Add **key** "accept" and Value "application/json;odata=verbose"
    *   Add **key** "content-type" and Value  "application/json;odata=verbose"
    *   Add **key** "X-RequestDigest" and Value ‍[{Variable:strDigestInfo}](https://neu-des-wfo.nintexo365.net//href=%22#")‍
    *   Add **Output** to variable RequestHeadersPerm.

Request Headers ![build-request-headers](https://reshmeeauckloo.files.wordpress.com/2016/09/build-request-headers.png)

*   Add Build Dictionary step.  Rename is to "Build Metadata".
    *   Add **key** "type" and value "SP.Sharing.UserRoleAssignment".
    *   **Output** in variable MetadaPerm.

![buildmetadata](https://reshmeeauckloo.files.wordpress.com/2016/09/buildmetadata.png)

*   Add Build Dictionary step.  Rename is to "Build UserRoleAssignments".
    *   Add **key** "\_\_metadata" and Value of type Dictionary to variable MetadataPerm
    *   Add **key** "Role" and value of type Integer 1. The Role property represents the level of permission you want to grant. Possible values are  1 =  View, 2 =  Edit, 3 = Owner, 0 = None.
    *   Add **UserId** to ‍[{Current Item:Email}](https://neu-des-wfo.nintexo365.net//href=%22#")‍
    *   Add **Output** to UserRoleAssignments

![build-userroleassignments](https://reshmeeauckloo.files.wordpress.com/2016/09/build-userroleassignments.png)

*   Add Item to Collection Step
    *   Set **Target collection** to  collection variable RoleAssignmentCol
    *   Set **Index** 0
    *   Set **Value** to dictionary variable UserAssignmentts
    *   Set **Output** to RoleAssignmentCol

![roleassignmentstocollection](https://reshmeeauckloo.files.wordpress.com/2016/09/roleassignmentstocollection.png)

*   Add a **Build  Dictionary** step and label to "Build Parameters"
    *   Add **Key** "userRoleAssignments" and Value of type Dictionary to variable RoleAssignmentCol
    *   Add **Key** "resourceAddress" and Value ‍[{Workflow Context:Current site URL}](https://neu-des-wfo.nintexo365.net//href=%22#")‍‍[{Current Item:SharePointLink}](https://neu-des-wfo.nintexo365.net//href=%22#")‍
    *   Add **Key** "validateExistingPermissions" and Value of type Boolean set to No
    *   Add **key** "additiveMode" and Value of type Boolean set to Yes
    *   Add **Key** "sendServerManagedNotification" and Value of type Boolean set to Yes
    *   Add **Key** "customMessage" and Value "Document has been shared with you"
    *   Add Key "includeAnonymousLinksInNotification" and Value of type Boolean set to Yes
    *   Add Key "propagateAcl" and Value of type Boolean set to Yes

  ![buildparameters](https://reshmeeauckloo.files.wordpress.com/2016/09/buildparameters.png)

*   Add a Call Http Web Service step to assign permissions using SP.Sharing.DocumentSharingManager.UpdateDocumentSharingInfo![listcolumnconfig](https://reshmeeauckloo.files.wordpress.com/2016/09/listcolumnconfig.png)
    *   Set **Address** to ‍[{Workflow Context:Current site URL}](https://neu-des-wfo.nintexo365.net//href=%22#")‍/\_api/SP.Sharing.DocumentSharingManager.UpdateDocumentSharingInfo
    *   Set **Request Type** to "Http Post"
    *   Set **Request Headers** to variable RequestHeadersPerm
    *   Set **Request Content** to variable ParametersPerm
    *   Set **Response Headers** to variable ResponseContentPerm
    *   Set **Response Content **to variable ResponseHeadersPerm
    *   Set **Response Status Code** to ResponseStatusCode

  ![callhttpwebservice_updatedocumentsharinginfo](https://reshmeeauckloo.files.wordpress.com/2016/09/callhttpwebservice_updatedocumentsharinginfo.png) You can use the **Log to History List** step to log the ResponseStatusCode,  ResponseContentPerm and ResponseHeadersPerm. An email will be sent to the external user when run with a link to access the resource. ![emailgeneratedwithlink](https://reshmeeauckloo.files.wordpress.com/2016/09/emailgeneratedwithlink.png)