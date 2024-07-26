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

### Set Variables

![Set Variables](../images/powerautomate-approval-action-reminders/setVariables.png)

1. Set variable to store the approvalId
![Approval Id](../images/powerautomate-approval-action-reminders/var_approvalid.png)

2. Set variable to store the respondlink
![Respond link](../images/powerautomate-approval-action-reminders/var_respondlink.png)

### Until Loop to trigger reminder if not already approved for 4 times

Add `Do Unitl` loop action

![Until Loop condition](../images/powerautomate-approval-action-reminders/UntilApprovalIsComplete.png)

Add the following actions within the loop

#### Wait for Approval 

Behavior: Waits for a specific approval process to complete.
Condition to run on time out of the action `Wait for Approval`

![Wait for Approval](../images/powerautomate-approval-action-reminders/WaitforAnApproval.png)

Settings for Approval to timeout after 6 days

![Wait for Approval settings](../images/powerautomate-approval-action-reminders/WaitforAnApproval_Settings.png)

#### Condition to run on timeout

Action to check criteria to send reminder
![Condition to send reminder](../images/powerautomate-approval-action-reminders/condition_to_send_reminder.png)

Configure the Run after to be last action timing out

![Trigger on timeout](../images/powerautomate-approval-action-reminders/ConditionToSendReminderOnTimeOutifApprovalPending.png

#### Send Reminder Email 

![Condition to send reminder](../images/powerautomate-approval-action-reminders/condition_to_send_reminder_steps.png)

![Send Email](../images/powerautomate-approval-action-reminders/SendEmail.png)

#### Increment variable ReminderCount

ReminderCount keeps track of the number of reminders sent and is incremented by 1 each time a email reminder is sent

#### Add 'Terminate' action

Ad `Terminate` action and set status to `Succeeded` to complete the flow.

#### Handle Approval

![Handle Approvals](../images/powerautomate-approval-action-reminders/HandleApprovals.png)

Add a parallel `Set Variable` action after the `Wait for Approval` and specify the run after property to be successful to set the variable `varApprovalComplete` to true.

![Run after success](../images/powerautomate-approval-action-reminders/SetApprovalComplete_RunAfterSuccess.png)

Add other actions how to handle Approved or Reject status.

### Handle Timeout to cancel approval task 

![Handle Approvals](../images/powerautomate-approval-action-reminders/HandleTimeOut.png)

* Add `Delay` action for 28 days as a parallel action to the action 'Until Loop to trigger reminder if not already approved for 4 times'

* Cancel Approval Task using `Update a row` Dataverse connector

![Cancel Approvals](../images/powerautomate-approval-action-reminders/CancelApproval.png)

* Add `Terminate` Action to cancel flow setting status to 'Cancelled`

* Set variable varApprovalComplete to true

* Add any other corresponding actions like updating list item status