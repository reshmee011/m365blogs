---
title: "Power Automate: Retrieve Users from a SharePoint Group"
date: 2024-07-23T09:53:05+01:00
tags: ["Power Automate","SharePoint","SharePoint Group","REST API"]
featured_image: '/posts/images/powerautomate-create-newlinks/GetPageDetails.png'
omit_header_text: true
draft: false
---

This post covers how to leverage SharePoint REST API to get users from a SharePoint group from Power Automate using the `Send an Http request to SharePoint` action. In the example below a SharePoint Group has been defined for approvers of a particular process and needed to be retrieved to be assigned an approval task.

Within a Power Automate flow follow the steps below to retrieve users from a SharePoint group.

1. Use `Send an Http request to SharePoint` renamed to `Get Approvers from SharePointGroup`

![get group users](../images/powerautomate-get-users-from-SharePoint-Group/GetUsersFromSharePointGroup.png)

* Site Address: site url
* Method : Get
* Body : /_api/web/sitegroups/getbyname('test group')/users

2. Parse JSON action

Use the `Parse JSON` action and refer to the output from previous step

![parse json](../images/powerautomate-get-users-from-SharePoint-Group/ParseGroupJSON.png)

* Content : body('Get_Approvers_from_SharePoint_Group')
* Schema:
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

3. `Apply To each` action 

Add a `Apply To Each` action referring to the output from previous steps.

* Select output from previous steps : body('Parse_Approvers_JSON')?['d']?['results']

**Add step `Append to string variable`**
Define a string variable using the `Initialiaze Variable` to store the concatenated email addresses
 
* Name : string Variable
* Value  : items('Apply_to_each')?['Email'];

![Apply to each](../images/powerautomate-get-users-from-SharePoint-Group/ForEach_Results_To_Concatenate.png)

**Use the Concatenated User Variable**
Refer to the concatenated user variable in any subsequent action, such as the "Assigned to" field within the "Start and wait for an approval" action.

