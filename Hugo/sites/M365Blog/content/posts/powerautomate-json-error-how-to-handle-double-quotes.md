---
title: "JSON Data Handling in Power Automate: Double Quotes"
date: 2024-07-24T09:53:05+01:00
tags: ["Power Automate", "JSON", "Data Handling","Coalesce"]
featured_image: '/posts/images/powerautomate-json-error-how-to-handle-double-quotes/InternalError_Json_cause.png'
omit_header_text: true
draft: false
---

When using Power Automate action `Send an Http request to SharePoint` specially to send data in the body to a REST API call, for instance to create a news link from a SharePoint List Item, you might encounter issues if the body contains double quotes in the title or description. This blog post will walk you through the problem and provide solutions to handle double quotes in JSON strings.

## Problem Overview

Using the Send an Http request to SharePoint action to create a news link can fail if the JSON body contains double quotes. Here's an example of a problematic JSON body:

The body sent to the action

```json
{"BannerImageUrl":"https://contoso.sharepoint.com/_layouts/15/images/sitepagethumbnail.png","Description":"test","IsBannerImageUrlExternal":false,"OriginalSourceUrl":"https://contoso.sharepoint.com/sites/d-intranet-employeehub/SitePages/Experience.aspx","ShouldSaveAsDraft":false,"Title":""I'm very happy" - Test'experience! ","OriginalSourceSiteId":"c038cc4c-de05-4bed-88e0-522888d8b2c8","OriginalSourceWebId":"5f74f9b7-1559-45d1-b6f2-7b86e329f768","OriginalSourceListId":"a9893c10-d354-4b97-a72f-52859f0a1d93","OriginalSourceItemId":"{AE695BA1-4EBA-4C06-B22D-D48DE39A5FA3}","__metadata":{"type":"SP.Publishing.RepostPage"}}
```
![Error](../images/powerautomate-json-error-how-to-handle-double-quotes/InternalError_Json_cause.png)

This results in an error because the JSON parser expects commas to separate elements, and double quotes within strings can disrupt this structure.

## Error Message

Here's the typical error message you might see:

>{
  "status": 400,
  "message": "Invalid JSON. A comma character ',' was expected in scope 'Object'. Every two elements in an array and properties of an object must be separated by commas.\r\nclientRequestId: ef86ef11-4817-43e9-ab55-92bee266232c\r\nserviceRequestId: a1793fa1-f0fb-9000-9682-448e50553858",
  "source": "https://contoso.sharepoint.com/sites/d-intranet/_api/sitepages/pages/reposts",
  "errors": [
    "-1",
    "Microsoft.SharePoint.Client.InvalidClientQueryException"
  ]
}

![Error](../images/powerautomate-json-error-how-to-handle-double-quotes/InternalError_Json.png)

## Solution

### Escape Double Quotes

To fix this issue, you need to escape double quotes in your JSON strings. Use a backslash (\) before each double quote.

**Replace function:**

```json
Replace(triggerBody()['text'], '"', '\"')
```

If a null value is passed to the Replace function, the error below will be thrown.

### Handle Null Values

To avoid errors with null values, use the coalesce function to provide a default value.

> Unable to process template language expressions in action 'Create_News_Link' inputs at line '0' and column '0': 'The template language function 'replace' expects its first parameter 'string' to be a string. The provided value is of type 'Null'. Please see https://aka.ms/logicexpressions#replace for usage details.'.

Using coalesce and replace together:

```json
Replace(coalesce(body('Parse_Page_Details_JSON')?['d']?['Title'],''),'"','\"')
Replace(coalesce(body('Parse_Page_Details_JSON')?['d']?['Description'], ''), '"', '\"')
```

## Example Fixed JSON Body
After applying the fix, your JSON body should look like this:

```Json
{"BannerImageUrl":"body('Parse_Page_Details_JSON')?['d']?['BannerImageUrl']?['Url']",
"Description":"replace(coalesce(body('Parse_Page_Details_JSON')?['d']?['Description'], ''), '"', '\"')","IsBannerImageUrlExternal":false,
"OriginalSourceUrl":"concat(triggerOutputs()?['body/SiteUrl'],'/SitePages/',triggerOutputs()?['body/PageUrl'])","ShouldSaveAsDraft":false,
"Title":"Replace(coalesce(body('Parse_Page_Details_JSON')?['d']?['Title'],''),'"','\"')",
"OriginalSourceSiteId":"triggerOutputs()?['body/OriginalSourceSiteId']",
"OriginalSourceWebId":"triggerOutputs()?['body/OriginalSourceWebId']",
"OriginalSourceListId":"triggerOutputs()?['body/OriginalSourceListId']",
"OriginalSourceItemId":"triggerOutputs()?['body/OriginalSourceItemId']",
"__metadata":{"type":"SP.Publishing.RepostPage"}}
```

By following these steps, you can ensure that your Power Automate flow correctly handles double quotes in JSON strings, preventing errors when creating and publishing news links from SharePoint List Items.