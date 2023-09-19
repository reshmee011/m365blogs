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

I had the requirement upon approval to update file properties with the approval results without creating a new version and had to rely on the "Send HTTP request to SharePoint" action

In my scenerio, I am using the option first one to approve to proceed with the flow. 


Capturing details from approval steps
 

### Approver

[{'Key': 'i:0#.f|membership|@{outputs('Get_user_profile_(V2)')?['body/mail']}'}]

 

Date

formatDateTime(outputs('Start_and_wait_for_an_approval')?['body/completionDate'],'dd/MM/yyyy HH:mm')

 

Comments

outputs('Start_and_wait_for_an_approval')?['body/responses']?[0]?['comments']

References
 (Working with the SharePoint Send HTTP Request flow action in Power Automate)[https://learn.microsoft.com/en-us/sharepoint/dev/business-apps/power-automate/guidance/working-with-send-sp-http-request]