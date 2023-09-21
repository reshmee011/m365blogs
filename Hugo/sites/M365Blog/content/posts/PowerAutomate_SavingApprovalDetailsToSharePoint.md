---
title: "Updating Approval Details in SharePoint Library"
date: 2023-09-14T07:17:21+01:00
draft: true
---

# Updating Approval Details in SharePoint Library
 
In Power Automate, there are two methods for updating file properties:

Send HTTP Request to SharePoint
Update File Properties
While the latter suffices for most cases, I opted for the "Send HTTP Request to SharePoint" in specific scenarios:

To avoid triggering a new file version upon update.
When modifying system columns such as author or modified date.
When updating an image column.
Upon approval, I needed to update file properties with the approval outcome, comments, date, and approver without generating a new version. This necessitated the use of the "Send HTTP Request to SharePoint" action.

After several iterations, the fields were successfully updated:

![Approval Step](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/AllFieldsUpdatedCorrectly.png)

In my scenario, I employed the "First to Respond" approval type:

![Approval Step](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/ApprovalStep.png)

## Capturing Details from Completed Approval Steps
### Approver

Initially, I used the syntax i:0#.f|membership|outputs('Get_user_profile_(V2)')?['body/mail'].

The flow succeeded, but the step did not throw any exceptions:

![Person Field ](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/WrongPersonFormat.png)

[{'Key': 'i:0#.f|membership|@{outputs('Get_user_profile_(V2)')?['body/mail']}'}]

 
![Person Field ](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/CorrectPersonFormat.png)

### Date

To format the date, I used the expression:

```json
formatDateTime(outputs('Start_and_wait_for_an_approval')?['body/completionDate'],'dd/MM/yyyy HH:mm')
```

However, there were issues with the date format. For instance, if the date column type is "date only," providing 'dd/MM/yyyy HH:mm' as the format will result in the following error:

![Approval Step](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/WrongDateFormat_ApprovalDate.png)

 After updating the column to include time, using the date format still produced an exception. This was due to the regional settings for the site being set to En-US instead of En-UK. After rectifying this in the site settings under "Regional Settings" (/_layouts/15/regionalsetng.aspx), the issue was resolved:

![Regional Setting issue](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/WrongDateFormat_ApprovalDate_Locale.png)


### Comments
![Comments Issue](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/WrongDateFormat_ApprovalComments.png)

I accessed the comments using the expression:
outputs('Start_and_wait_for_an_approval')?['body/responses'][0]['comments']

Or alternatively:
outputs('Start_and_wait_for_an_approval')?['body/responses']?[0]?['comments']

### References
 (Working with the SharePoint Send HTTP Request flow action in Power Automate)[https://learn.microsoft.com/en-us/sharepoint/dev/business-apps/power-automate/guidance/working-with-send-sp-http-request]
(SP.ListItem.validateUpdateListItem Method)[https://docs.microsoft.com/en-us/previous-versions/office/sharepoint-visio/jj246412%28v%3doffice.15%29?msclkid=4f151da6cd3e11ec9aec9a4c8ab167e0]