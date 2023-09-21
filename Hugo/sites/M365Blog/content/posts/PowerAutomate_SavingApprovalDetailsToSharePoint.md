---
title: "Saving approval details back to SharePoint library"
date: 2023-09-14T07:17:21+01:00
draft: true
---

# Saving approval details back to SharePoint library 
 
There are two options that can be used to update File properties in Power Automate 

1. Send HTTP request to SharePoint
2. Update file properties

The "Update file properties" might be enough in most circumtances, I have used the "Send HTTP request to SharePoint" under the following circumstances

- Not to trigger a new file version to be created upon update
- Update some system columns e.g. author or modified date
- Update image column

I had the requirement upon approval to update file properties with the approval outcome, approval comments, approval date and approver without creating a new version and had to rely on the "Send HTTP request to SharePoint" action.

After several iterations going through different options, the fields were updated

![Approval Step](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/AllFieldsUpdatedCorrectly.png)

In my scenerio, I am using the "First to respond" approval type.

![Approval Step](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/ApprovalStep.png)

Capturing details from approval steps after the approval steps are complete


 

### Approver

Initially I was using the syntax i:0#.f|membership|outputs('Get_user_profile_(V2)')?['body/mail']

The flow succeeded that the step did not throw any exception 

![Person Field ](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/WrongPersonFormat.png)

[{'Key': 'i:0#.f|membership|@{outputs('Get_user_profile_(V2)')?['body/mail']}'}]

 
![Person Field ](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/CorrectPersonFormat.png)

Date

formatDateTime(outputs('Start_and_wait_for_an_approval')?['body/completionDate'],'dd/MM/yyyy HH:mm')

Issues with Date: 
Wrong format , for instance if the type of date column is "date only" providing 'dd/MM/yyyy HH:mm' as format will result in the following error

![Approval Step](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/WrongDateFormat_ApprovalDate.png)
 
After updating the column to date including time, using the date format still produced the exception despite the flow succeeded because the regional settings for the site was set to En-US instead of En-UK. After updating the regional settings of the site to en-uk going to "site settings>regional settings" : /_layouts/15/regionalsetng.aspx .

![Regional Setting issue](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/WrongDateFormat_ApprovalDate_Locale.png)


Comments
![Comments Issue](../images/PowerAutomate_SavingApprovalDetailsToSharePoint/WrongDateFormat_ApprovalComments.png)

outputs('Start_and_wait_for_an_approval')?['body/responses'][0]['comments']


outputs('Start_and_wait_for_an_approval')?['body/responses']?[0]?['comments']

References
 (Working with the SharePoint Send HTTP Request flow action in Power Automate)[https://learn.microsoft.com/en-us/sharepoint/dev/business-apps/power-automate/guidance/working-with-send-sp-http-request]
(SP.ListItem.validateUpdateListItem Method)[https://docs.microsoft.com/en-us/previous-versions/office/sharepoint-visio/jj246412%28v%3doffice.15%29?msclkid=4f151da6cd3e11ec9aec9a4c8ab167e0]