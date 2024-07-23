---
title: "Power Automate get SharePoint group users"
date: 2024-07-23T09:53:05+01:00
tags: ["Power Automate","Copy Actions","Environment"]
featured_image: '/posts/images/powerautomate-create-newlinks/GetPageDetails.png'
omit_header_text: true
draft: true
---

1. Http request to SharePoin
![get group users](../images/powerautomate-get-users-from-SharePoint-Group/GetUsersFromSharePointGroup.png)
Site Address: site url
Method : Get
Body : /_api/web/sitegroups/getbyname('test group')/users

2. Parse JSON action
![parse json](../images/powerautomate-get-users-from-SharePoint-Group/ParseGroupJSON.png)
Content : body('Get_Approvers_from_SharePoint_Group')
Schema:
```Json
{
    "type": "object",
    "properties": {
        "d": {
            "type": "object",
            "properties": {
                "results": {
                    "type": "array",
                    "items": {
                        "type": "object",
                        "properties": {
                            "__metadata": {
                                "type": "object"
                            },
                            "LoginName": {
                                "type": "string"
                            },
                            "Title": {
                                "type": "string"
                            },
                            "PrincipalType": {
                                "type": "integer"
                            },
                            "Email": {
                                "type": "string"
                            },
                            "Expiration": {
                                "type": "string"
                            },
                            "IsEmailAuthenticationGuestUser": {
                                "type": "boolean"
                            },
                            "IsShareByEmailGuestUser": {
                                "type": "boolean"
                            },
                            "IsSiteAdmin": {
                                "type": "boolean"
                            },
                            "UserId": {
                                "NameId": {
                                    "type": "string"
                                },
                                "NameIdIssuer": {
                                    "type": "string"
                                }
                            }
                        },
                        "UserPrincipalName": {
                            "type": "string"
                        }
                    }
                }
            }
        }
    }
}
```

3. Apply To each 

Select output from previous steps : body('Parse_Approvers_JSON')?['d']?['results']

 Add step `Append to string variable`
 Name : string Variable
 Value  : items('Apply_to_each')?['Email'];

![Apply to each](../images/powerautomate-get-users-from-SharePoint-Group/ ForEach_Results_To_Concatenate.png)

Refer to the concatenated user variable in any action, e.g. Assigned to field within action start and wait for an approval 
