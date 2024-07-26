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

### Until Loop to trigger reminder if not already approved for 4 times

![Until Loop condition](../images/powerautomate-approval-action-reminders/UntilApprovalIsComplete.png)

Add the following actions within the loop

#### Wait for Approval 
Behavior: Waits for a specific approval process to complete.
Condition to run on time out of the action `Wait for Approval`

![Wait for Approval](../images/powerautomate-approval-action-reminders/WaitforAnApproval.png)

Settings for Approval to timeout after 6 days

![Wait for Approval settings](../images/powerautomate-approval-action-reminders/WaitforAnApproval_Settings.png)

#### Add Condition to run on timeout

Action to check criteria to send reminder
![Condition to send reminder](../images/powerautomate-approval-action-reminders/condition_to_send_reminder.png)

Configure the Run after to be last action timing out

![Trigger on timeout](../images/powerautomate-approval-action-reminders/ConditionToSendReminderOnTimeOutifApprovalPending.png

#### Send Reminder Email 

![Condition to send reminder](../images/powerautomate-approval-action-reminders/condition_to_send_reminder_steps.png)

