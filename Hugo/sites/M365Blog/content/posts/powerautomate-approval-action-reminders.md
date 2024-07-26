---
title: "Power Automate Approval Actions: Reminders option"
date: 2024-07-25T09:53:05+01:00
tags: ["Power Automate", "Approval", "Reminders"]
featured_image: '/posts/images/powerautomate-approval-action-reminders/approval.png'
omit_header_text: true
draft: true
---

Power Automate offers an approval action to fulfil business approval requirement. This post covers some workarounds around two limitations

**No Built-in Reminders**: Does not provide overdue reminders out of the box.

**Timeout Limit**: Power Autpmate flow times out after 30 days due to a time out though the approval task stays active in the background.

## Steps to add reminders every 6 days from the approval being created 

### Initialise Variables

1. Create a variable to check whether approval is complete
![ApprovalComplete](../images/powerautomate-approval-action-reminders/var_approvalcomplete.png)

2. Create a variable to keep track of the times the reminder has been sent
![Reminder Count](../images/powerautomate-approval-action-reminders/var_remindercount.png)

3. Create a variable to store the approvalId
![Approval Id](../images/powerautomate-approval-action-reminders/var_approvalid.png)

4. Create a variable to store the respondlink
![Respond link](../images/powerautomate-approval-action-reminders/var_respondlink.png)

### Create an Approval 

The 'Create an Approval` starts the approval process without waiting for a response.
![Approval task](../images/powerautomate-approval-action-reminders/CreateApprovalTask.png)

### Wait for Approval 


### Loop through 4 four times or until approval is complated



## Start and Wait for an Approval

Behavior: Commonly used. Keeps the workflow running until all approvers complete their tasks.

## Timeout

Approval Task lasts forever despite

## Wait for an Approval

Behavior: Waits for a specific approval process to complete.

## Custom Outcomes and Responses

When using the "Start and Wait for an Approval" action, you can specify different approval types and custom responses:

### Approval Types:

Custom Responses - Wait for all responses
Approve/Reject - First to respond
Approve/Reject - Everyone must approve
Custom Responses - Wait for one response

## Approval Settings

* Timeout: Uses ISO 8601 format (e.g., P5D for 5 days, PT1M30S for 1 minute and 30 seconds).

* Retry Policy: Default is 4 retries. Options include none, exponential interval, or fixed interval, also using ISO 8601 format.

## Alternatives to Approval Actions

### Send Email with Options:

**Pros** 
* Allows for customized email text and outcomes.
* External people can be set as approvers.

### SharePoint Content Approval

**Pro**
* Enabled within SharePoint lists and libraries.
* Use on document, image, or page libraries.
* Supports versioning settings with content approval.
* Workflow can handle approval status changes and actions based on whether an item is approved or rejected.
