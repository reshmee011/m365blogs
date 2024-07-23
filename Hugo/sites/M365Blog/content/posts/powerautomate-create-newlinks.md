---
title: "Power Automate create news link"
date: 2024-07-23T09:53:05+01:00
tags: ["Power Automate","Copy Actions","Environment"]
featured_image: '/posts/images/powerautomate-create-newlinks/GetPageDetails.png'
omit_header_text: true
draft: true
---

1. Get Page Details
Site Address : site url 
Method : GET
Uri : /_api/web/lists/GetByTitle('Site%20Pages')/items(@{triggerOutputs()?['body/SourcePageId']})

![Get News Details](../images/powerautomate-create-newlinks/GetPageDetails.png)

2. Parse JSON
Content : body('Get_Page_Details')
Schema: 

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

3. Create News Link

Site Address : site url 
Method : POST 
Uri : _api/sitepages/pages/reposts
Headers : content-type: application/json;odata=verbose
        accept : application/json;odata=verbose

Body : {"BannerImageUrl":"@{body('Parse_Page_Details_JSON')?['d']?['BannerImageUrl']?['Url']}","Description":"@{body('Parse_Page_Details_JSON')?['d']?['Description']}","IsBannerImageUrlExternal":false,"OriginalSourceUrl":"@{concat(triggerOutputs()?['body/SiteUrl'],'/SitePages/',triggerOutputs()?['body/PageUrl'])}","ShouldSaveAsDraft":false,"Title":"@{body('Parse_Page_Details_JSON')?['d']?['Title']}","OriginalSourceSiteId":"@{triggerOutputs()?['body/OriginalSourceSiteId']}","OriginalSourceWebId":"@{triggerOutputs()?['body/OriginalSourceWebId']}","OriginalSourceListId":"@{triggerOutputs()?['body/OriginalSourceListId']}","OriginalSourceItemId":"@{triggerOutputs()?['body/OriginalSourceItemId']}","__metadata":{"type":"SP.Publishing.RepostPage"}}

![Create News Link](../images/powerautomate-create-newlinks/CreateNewsLink.png)