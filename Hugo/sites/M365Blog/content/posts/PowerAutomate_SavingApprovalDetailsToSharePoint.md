---
title: "Updating Approval Details in SharePoint Library"
date: 2023-09-21T07:17:21+01:00
tags: ["PowerAutomate", "Approval","SharePoint", "Send HTTP Request to SharePoint"]
featured_image: '/posts/images/PowerAutomate_SavingApprovalDetailsToSharePoint/AllFieldsUpdatedCorrectly.png'
draft: false
---

# Updating Approval Details in SharePoint Library
 
In Power Automate, there are two methods for updating file properties:

1. Send HTTP Request to SharePoint
2. Update File Properties

While the latter suffices for most cases, I opted for the "Send HTTP Request to SharePoint" in specific scenarios:

- To avoid triggering a new file version using system update.
- When modifying system columns such as author or modified date.
- When updating an image column.

Upon approval of a document, I needed to update file properties with the approval outcome, comments, date, and approver without generating a new version. This necessitated the use of the "Send HTTP Request to SharePoint" action.

After several iterations overcoming some challenges, the fields were successfully updated:

![Approval Step](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/AllFieldsUpdatedCorrectly.png)

In my scenario, I employed the "First to Respond" approval type:

![Approval Step](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/ApprovalStep.png)

After the approval is completed, the approval details are saved back to the SharePoint library. 

## Capturing Details from Completed Approval Steps

### Approver

Initially, I used the syntax

```json
i:0#.f|membership|outputs('Get_user_profile_(V2)')?['body/mail'].
```

The flow succeeded and the step did not throw any exceptions despite the approver field was not updated.

![Person Field ](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/WrongPersonFormat.png)

I found the accurate syntax for updating a person field by examining the network trace in the developer tools of the browser by simulating the update of a person field in SharePoint and capture the corresponding API call. The claims of the user need to be passed with a key property to update the person field.

```json
[{'Key': 'i:0#.f|membership|@{outputs('Get_user_profile_(V2)')?['body/mail']}'}]
```
 
![Person Field ](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/CorrectPersonFormat.png)

### Date

THe date needs to be tweaked for the correct datetime string, otherwise you might encounter the following error message.

"You must specify a valid date within the range of 01/01/1900 and 31/12/8900."

I used the following datetime format 'dd/MM/yyyy HH:mm'.

```json
formatDateTime(outputs('Start_and_wait_for_an_approval')?['body/completionDate'],'dd/MM/yyyy HH:mm')
```

However, there were issues with the date format. For instance, if the date column type is "date only," providing 'dd/MM/yyyy HH:mm' as the format will result in the following error:

![Approval Step](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/WrongDateFormat_ApprovalDate.png)

 After updating the column to include time, using the date format still produced an exception. This was due to the regional settings for the site being set to En-US instead of En-UK. After rectifying this in the site settings under "Regional Settings" (/_layouts/15/regionalsetng.aspx), the issue was resolved:

![Regional Setting issue](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/WrongDateFormat_ApprovalDate_Locale.png)


### Comments

I initially accessed the comments using the expression:

```json
outputs('Start_and_wait_for_an_approval')?['body/responses'][0]['comments']
```

However if no comments are provided during approval, the flow fails. 

![Comments issue](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/ApprovalComments_error.png)

To prevent the error I used the '?'. This is particularly useful for navigating through potentially nested structures where some properties may or may not exist. For instance, if outputs('Start_and_wait_for_an_approval') returns an object, and it has a property body which is also an object, and within that, there's a property responses which is an array, the ?[0] ensures that you're trying to access the first element of that array only if it exists. 

```json
outputs('Start_and_wait_for_an_approval')?['body/responses']?[0]?['comments']
```

### References

 [Working with the SharePoint Send HTTP Request flow action in Power Automate](https://learn.microsoft.com/en-us/sharepoint/dev/business-apps/power-automate/guidance/working-with-send-sp-http-request)

[SP.ListItem.validateUpdateListItem Method](https://docs.microsoft.com/en-us/previous-versions/office/sharepoint-visio/jj246412%28v%3doffice.15%29?msclkid=4f151da6cd3e11ec9aec9a4c8ab167e0)

[Parsing JSON in Power Apps](https://www.youtube.com/watch?v=VRXM7UT3iwU) by the amazing [Cat Schneider](https://twitter.com/YerAWizardCat)