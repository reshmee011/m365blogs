---
title: 'Nintex DocSign Enable Multiple Recipients'
date: Sat, 30 Apr 2016 10:04:39 +0000
draft: false
tags: ['DocSign', 'JSON', 'Nintex', 'REST', 'SharePoint Online', 'Uncategorized']
---

Doc Sign is a useful secure way of signing documents which is legally binding. Nintex provides the DocuSign send document action that integrates DocuSign service. A trial account for PROD can be created but the trial limit is 3 documents. For development or evaluation purposes, a free developer account can be created on [DocuSign](https://www.docusign.com/developer-center) site. Follow the instructions from [The Quick Quide to Doc Sign ](https://www.docusign.com/developer-center/api-overview#quickstart) to configure your dev account. Part of the process is to create an integrator key which is needed to call DocuSign REST API. Also create a template which can be referred in the REST API call. The issue with the  Nintex "DocuSign populate document" action is that it does not support multiple recipients. It tells DocuSign to send one recipient only. The workaround as suggested in the [Nintex forum](https://community.nintex.com/message/31841) is to repeat the action but the document is duplicated and  tracing history is lost. The option is to use the "Web Request" action from Nintex to send to multiple recipients. To make a REST CALL to Doc Sign you need the following info

1.  Account ID: Can be found after you login to DocSign by clicking on the arrow down next to your picture. The number displayed after your email address is the account id to use.       ![DocSignAccountID](https://reshmeeauckloo.files.wordpress.com/2016/04/docsignaccountid.png)
2.  Integrator Key: From the menu displayed next to your profile picture, click on "Go to Admin"> "API and Keys" under Integration section. If there is not integrator key, click on "Add Integrator key" else note down the integrator key.![IntegratorKey](https://reshmeeauckloo.files.wordpress.com/2016/04/integratorkey.png)
3.  Template ID: Click on "Templates" link and click on the _i_ next to the template a![DocSignTemplateID](https://reshmeeauckloo.files.wordpress.com/2016/04/docsigntemplateid.png)

In the Nintex Workflow add a "Web Request" Action ![MultipleRecipientsDocSign](https://reshmeeauckloo.files.wordpress.com/2016/04/multiplerecipientsdocsign1.png) Set the following fields

1.  URL: https://demo.docusign.net/restapi/v2/accounts/{account id}/envelopes
2.  Method: Select POST and the following headers
    *   Content Type : application/json;odata=verbose
    *   X-DocuSign-Authentication : {"Username":"{DocSign Account}","Password":"{DocSign password}","IntegratorKey":"{key}"}
3.  Body : Choose Content and  copy the following  JSON request{  "emailSubject": "Please sign", "emailBlurb": "Please sign...thanks!", "status": "sent", "compositeTemplates": \[ {    "serverTemplates": \[ {   "sequence" : 1, "templateId": "{template id}"    }\], "inlineTemplates": \[    { "sequence" : 2, "recipients": { "signers" : \[{ "email": "{test@em.com}", "name": "Test1 Test1", "recipientId": "1", "roleName": "Signer", "routingOrder": "1" }, { "email": "test2@yahoo.co.uk", "name": "Test2 Test 2", "recipientId": "2", "roleName": "Signer 2", "routingOrder": "2" } \] } }\] }\]  }
4.  Store response content in : Create a variable DocSignWebRequestResponse
5.  Store http status code in : Create a variable DocSignWebRequestStatusCode
6.  Store response headers in: Create a variable DocSignWebRequestHeaders
7.  Store response cookies in: Create a variable DocSignWebRequestResponseCookies

The JSON format of the response is { "envelopeId": "84385783-fa5b-4b5e-8ef5-02702b505fb2", "uri": "/envelopes/84385783-fa5b-4b5e-8ef5-02702b505fb2", "statusDateTime": "2016-04-27T06:30:50.7530000Z", "status": "sent" } Add "Extract Substring of String from Index with Length" action to get the envelope id from variable DocSignWebRequestResponse. ![GetEnvelopeIDfromResponse](https://reshmeeauckloo.files.wordpress.com/2016/04/getenvelopeidfromresponse.png) Set the following fields

1.  String: {Variable: DocSignWebRequestResponse}
2.  Index: 20
3.  Number of Characters: 36
4.  Output: Text variable EnvelopeID

You can add a "DocuSign retrieve envelope status" action specifying the envelopeID in a loop to get status of the envelope id until it is completed with a particular time interval. Fill in the following fields

1.  Authorizing user: {DocSign UserName}
2.  Envelope ID: {Variable:EnvelopeID}
3.  Status: Save into variable status
4.  Message: Save into variable message

The Status will have value "In Process" if still waiting for signatures, else the status will be completed. ![DocSignRetriveEnvelopeStatus](https://reshmeeauckloo.files.wordpress.com/2016/04/docsignretriveenvelopestatus.png)