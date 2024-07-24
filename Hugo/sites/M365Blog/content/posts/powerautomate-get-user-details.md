---
title: "Power Automate: Retrieve User Details"
date: 2024-07-23T09:53:05+01:00
tags: ["Power Automate","Get User Details","REST API","SharePoint"]
featured_image: '/posts/images/powerautomate-get-user-details/GetAuthorDetails.png'
omit_header_text: true
draft: false
---

This post covers how to leverage SharePoint REST API to get user details from Power Automate using the `Send an Http request to SharePoint` action.

Within a Power Automate flow follow the steps below to retrieve a user details.

1. `Send an Http request to SharePoint` action renamed to `Get Page Author Details`

* Site Address: https://test.sharepoint.com
* Method: GET
* URI: _api/Web/GetUserById(14)

Here's the output from the API:

![Get User Details](../images/powerautomate-get-user-details/GetAuthorDetails.png)

Here's the output from the API:

```json
{
  "d": {
    "__metadata": {
      "id": "https://contoso.sharepoint.com/sites/d-intranet-hub/_api/Web/GetUserById(14)",
      "uri": "https://contoso.sharepoint.com/sites/d-intranet-hub/_api/Web/GetUserById(14)",
      "type": "SP.User"
    },
    "Alerts": {
      "__deferred": {
        "uri": "https://contoso.sharepoint.com/sites/d-intranet-hub/_api/Web/GetUserById(14)/Alerts"
      }
    },
    "Groups": {
      "__deferred": {
        "uri": "https://contoso.sharepoint.com/sites/d-intranet-hub/_api/Web/GetUserById(14)/Groups"
      }
    },
    "Id": 14,
    "IsHiddenInUI": false,
    "LoginName": "i:0#.f|membership|srv_test@ppf.co.uk",
    "Title": "SRV_test",
    "PrincipalType": 1,
    "Email": "SRV_test@ppf.co.uk",
    "Expiration": "",
    "IsEmailAuthenticationGuestUser": false,
    "IsShareByEmailGuestUser": false,
    "IsSiteAdmin": true,
    "UserId": {
      "__metadata": {
        "type": "SP.UserIdInfo"
      },
      "NameId": "100320022e6dd3f5",
      "NameIdIssuer": "urn:federation:microsoftonline"
    },
    "UserPrincipalName": "srv_test@ppf.co.uk"
  }
}
```

2. `Parse JSON` action renamed to `Parse JSON Author Details`

* Content: body('Get_Page_Author_details')
* Schema:

```json
{
    "type": "object",
    "properties": {
        "d": {
            "type": "object",
            "properties": {
                "__metadata": {
                    "type": "object",
                    "properties": {
                        "id": {
                            "type": "string"
                        },
                        "uri": {
                            "type": "string"
                        },
                        "type": {
                            "type": "string"
                        }
                    }
                },
                "Alerts": {
                    "type": "object",
                    "properties": {
                        "__deferred": {
                            "type": "object",
                            "properties": {
                                "uri": {
                                    "type": "string"
                                }
                            }
                        }
                    }
                },
                "Groups": {
                    "type": "object",
                    "properties": {
                        "__deferred": {
                            "type": "object",
                            "properties": {
                                "uri": {
                                    "type": "string"
                                }
                            }
                        }
                    }
                },
                "Id": {
                    "type": "integer"
                },
                "IsHiddenInUI": {
                    "type": "boolean"
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
```

3. Use Parsed User Details in any subsequent actions

To utilize the user details, refer to the parsed JSON:

```md
body('Parse_JSON_Author_Details')?['d']?['LoginName']
```

![Update Author](../images/powerautomate-get-user-details/UpdateAuthorDetails.png)