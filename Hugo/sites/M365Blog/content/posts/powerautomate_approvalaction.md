---
title: "Powerautomate_approvalaction"
date: 2024-03-28T04:45:38Z
draft: true
---

APPROVALS ACTION PROS/CONS

· What I like:
. Built in, interactive emails
. Buttons and text box in email, for outcome
and comments from approvers
. Built in parallel approvals
. Integrated with the mobile app
. Integrated with flow.Microsoft.com
Approvals section
· Receive app push notifications

APPROVALS ACTION PROS/CONS

· What I do not like:
· Emails are not customizable. Only the details of the request are.
· No visibility into existing approvals besides your own
. No out of box overdue reminders
· No out-of-box serial process
(They can be built out manually, using multiple approval actions in a Flow)
· Approvers can only be inside the organization.
. 30 day limit - flow action times out


Start versus Start and Wait

. Create an approval - does not wait,
just starts the approval process

· Start and wait for an approval -
Most common. Workflow will stay
running until people complete their
approvals.
* Timeout is automatically 30 days max

· Wait for an approval - waits for a specific approval process

Approvals

Actions

< Search connectors and actions

Triggers

Create an approval
Approvals

Start and wait for an approval
Approvals

Wait for an approval
Approvals

...

CUSTOM OUTCOMES / RESPONSES

Start and wait for an approval

Parameters

Settings

Code View

Testing

About

Approval Type *
Custom Responses - Wait for all responses

Approve/Reject - First to respond

Approve/Reject - Everyone must approve

Custom Responses - Wait for one response

Custom Responses - Wait for all responses

Start and wait for an approval


Approval Settings
OTHER SETTINGS

. Timeout - Uses ISO 8601 format. Example: P5D or PT1M30S
. Retry Policy - Default is 4. Can be set to none, exponential interval, or
fixed interval. Also uses ISO 8601 format.
. Tracked Property - Manually set values the action returns. Strings,
numbers, date/time.

. Take a look at the resources for this chapter, for references links to the
ISO syntax.

...

Alternatives
1. SEND EMAIL WITH OPTIONS
Optionally use this instead of the
Start an Approval action.
. Outcomes can be changed
. Email text can be customized
. Approvers (To) can be external
people.

2. SharePoint Content Approval
. Turned on from within SharePoint lists & libraries
· Use on document, image, or page libraries
· Versioning settings > Content Approval
· Added or edited item will have status = Pending
. Flow Actions
· Set content approval status to
Submit
. Condition looks at if an item was
approved
· If yes, Action = Approve
. If no, Action = Reject
. Get Etag from the set content
approval status (submit) action

3. TEAMS POST CHOICE OF OPTIONS

· Alerts a user in Teams chat
. Provides buttons and a comment box
. Flow waits for a response
· Useful Data returned:
. Selected option
. Comments
· Response time
· User



Error Handling

2. How do you set up a flow to take a certain action only if a specific error code
happens during the flow run?

Use logic and an expression in the flow to check the error's exact status code:
In this example, I'm looking for an error code of 409:
equals(int(actionOutputs('Create_file').statusCode), 409)