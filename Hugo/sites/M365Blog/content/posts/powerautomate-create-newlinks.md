---
title: "Power Automate: Create and Publish a News Link"
date: 2024-07-23T09:53:05+01:00
tags: ["Power Automate","Publish Page","SharePoint","News Link"]
featured_image: '/posts/images/powerautomate-create-newlinks/GetPageDetails.png'
omit_header_text: true
draft: false
---

This post covers how to leverage SharePoint REST API to create and publish a news link from Power Automate using the `Send an Http request to SharePoint` action.

Within a Power Automate flow follow the steps below to create and publish a news link details. 

1. `Send an Http request to SharePoint` action renamed to `Get Page Details`.

* Site Address : site url 
* Method : GET
* Uri : /_api/web/lists/GetByTitle('Site%20Pages')/items(10)

![Get News Details](../images/powerautomate-create-newlinks/GetPageDetails.png)

2. `Parse JSON` renamed to `Parse Page Details JSON`

Add the action `Parse JSON` and refer to the content from previous steps

* Content: body('Get_Page_Details')
* Schema:

```JSON
{
    "type": "object",
    "properties": {
        "d": {
            "type": "object",
            "properties": {
                "ContentTypeId": {
                    "type": "string"
                },
                "Title": {},
                "CanvasContent1": {
                    "type": "string"
                },
                "BannerImageUrl": {
                    "type": "object",
                    "properties": {
                        "__metadata": {
                            "type": "object",
                            "properties": {
                                "type": {
                                    "type": "string"
                                }
                            }
                        },
                        "Url": {
                            "type": "string"
                        }
                    }
                },
                "Description": {},
                "PromotedState": {
                    "type": "integer"
                },
                "FirstPublishedDate": {}
            }
        }
    }
}
```

![Parse Page Details](../images/powerautomate-create-newlinks/ParsePageDetailsJSON.png)

3. Add a `Send an Http request to SharePoint` action renamed to `Create News Link`

Configure the action

* Site Address : site url 
* Method : POST 
* Uri : _api/sitepages/pages/reposts
* Headers : content-type: application/json;odata=verbose
        accept : application/json;odata=verbose

Body : {"BannerImageUrl":"@{body('Parse_Page_Details_JSON')?['d']?['BannerImageUrl']?['Url']}","Description":"@{body('Parse_Page_Details_JSON')?['d']?['Description']}","IsBannerImageUrlExternal":false,"OriginalSourceUrl":"@{concat(triggerOutputs()?['body/SiteUrl'],'/SitePages/',triggerOutputs()?['body/PageUrl'])}","ShouldSaveAsDraft":false,"Title":"@{body('Parse_Page_Details_JSON')?['d']?['Title']}","OriginalSourceSiteId":"@{triggerOutputs()?['body/OriginalSourceSiteId']}","OriginalSourceWebId":"@{triggerOutputs()?['body/OriginalSourceWebId']}","OriginalSourceListId":"@{triggerOutputs()?['body/OriginalSourceListId']}","OriginalSourceItemId":"@{triggerOutputs()?['body/OriginalSourceItemId']}","__metadata":{"type":"SP.Publishing.RepostPage"}}

example of body sent to the API: 

{"BannerImageUrl":"https://contoso.sharepoint.com/_layouts/15/images/visualtemplateimage11.jpg","Description":"How do you get started? Select 'Edit' to start working with this basic two-column template with an emphasis on text and examples of text formatting. With your page in edit mode, select this paragraph and replace it with your own text. Then, select t…","IsBannerImageUrlExternal":false,"OriginalSourceUrl":"https://contoso.sharepoint.com/sites/d-intranet/SitePages/Demo-News-Need-to-Know.aspx","ShouldSaveAsDraft":false,"Title":"Demo News Need to Know","OriginalSourceSiteId":"c038cc4c-de05-4bed-88e0-522888d8b2c8","OriginalSourceWebId":"5f74f9b7-1559-45d1-b6f2-7b86e329f768","OriginalSourceListId":"a9893c10-d354-4b97-a72f-52859f0a1d93","OriginalSourceItemId":"{054308BF-59FB-4878-8F4A-6791D816E155}","__metadata":{"type":"SP.Publishing.RepostPage"}}

![Create News Link](../images/powerautomate-create-newlinks/CreateNewsLink.png)

4. Add a `Send an Http request to SharePoint` action renamed to `Publish Page`

Configure the action as follows

* Method: POST
* Uri: _api/sitepages/pages(@{body('Create_News_Link')['body']['d']['Id']})/publish
* Headers : content-type: application/json;odata=verbose
        accept : application/json;odata=verbose

![Publish News ](../images/powerautomate-create-newlinks/PublishNews.png)