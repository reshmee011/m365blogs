---
title: 'Update User Profile Property using Web Service Request action in Nintex'
date: Wed, 28 Oct 2015 17:43:52 +0000
draft: false
tags: ['Nintex', 'Nintex Workflow', 'SharePoint', 'SharePoint 2013', 'Web Service']
---

In SharePoint Online, Nintex workflow can be used to update a user profile property. Before using Nintex Workflow to update user it needs to be given the right permissions

1.  Allow workflow to use app permissions

 a. Navigate to site settings

 b. Under Site Actions, click Manage site features

c. Locate the Workflows can app permissions and click Activate if it is not activated yet.

![WorkflowCan use app permissions](https://reshmeeauckloo.files.wordpress.com/2015/11/workflowcan-use-app-permissions.jpg)2\. Allow workflow to use app permissions

3.Navigate to site settings> site app permissions

[![Site App Permissions](https://reshmeeauckloo.files.wordpress.com/2015/11/site-app-permissions.jpg?w=300)](https://reshmeeauckloo.files.wordpress.com/2015/11/site-app-permissions.jpg) Note down the workflow app id which is between i:0i.t|ms.sp.ext| and @

4\. Navigate to <SiteURL>/\_layouts/15/appinv.aspx

5\. Enter workflow app id  and click on Lookup button 6. Enter the following in Permission Request XML

  <AppPermissionRequests AllowAppOnlyPolicy="true" > <AppPermissionRequest Scope="[http://sharepoint/content/sitecollection](http://sharepoint/content/sitecollection "http://sharepoint/content/sitecollection")" Right="FullControl" /> <AppPermissionRequest Scope="[http://sharepoint/social/tenant](http://sharepoint/social/tenant)" Right="FullControl" /> </AppPermissionRequests> 7.Click on Create 8.Click on Trust the app. Unfortunately the REST API does not provide the capability to update user profile property of any user except the pictureurl property of currently login user. The SharePoint UserProfileService.asmx is still available. It can be used in a Web Request Action to update a user profile property. The example below updates the property "About Me". [![UpdateUserProfileProperty](https://reshmeeauckloo.files.wordpress.com/2015/10/updateuserprofileproperty.jpg?w=300)](https://reshmeeauckloo.files.wordpress.com/2015/10/updateuserprofileproperty.jpg) Set the following in the Web Request call. Url: https://tenant-admin.sharepoint.com/\_vti\_bin/userprofileservice.asmx SoapAction: http://microsoft.com/webservices/SharePointPortalServer/UserProfileService/ModifyUserPropertyByAccountName Body: <?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">  <soap:Body>    <ModifyUserPropertyByAccountName xmlns="http://microsoft.com/webservices/SharePointPortalServer/UserProfileService">      <accountName>‎‏i:0#.f|membership|test[@domain.com](mailto:test@domain.com)</accountName>      <newData>        <PropertyData>          <IsPrivacyChanged>false</IsPrivacyChanged>          <IsValueChanged>true</IsValueChanged>          <Name>AboutMe</Name>          <Privacy>NotSet</Privacy>          <Values>            <ValueData><Value xsi:type="xsd:string">Weekend finally</Value></ValueData></Values>        </PropertyData>     </newData>    </ModifyUserPropertyByAccountName>  </soap:Body></soap:Envelope> UserName: account having tenant admin rights Password: password of the user specified above Store Response Content in: Create a variable to store response content Store http status code in : Create a variable to store status code Store response headers in: Create a variable to store response headers Store response cookies in: Create a variable to store response cookies   To check whether user profile properties have been updated correctly use the api <site collection url>[/\_api/SP.UserProfiles.PeopleManager/GetPropertiesFor(accountName=@v)?@v=](https://paperblade.sharepoint.com/sites/dev/artelia/_api/SP.UserProfiles.PeopleManager/GetPropertiesFor(accountName=@v)?@v= "https://paperblade.sharepoint.com/sites/dev/artelia/_api/SP.UserProfiles.PeopleManager/GetPropertiesFor(accountName=@v)?@v=")'<accountname>' or access the user information list <siteurl>/\_layouts/15/people.aspx?MembershipGroupId=0 and click on the name you want to view data.