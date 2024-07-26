---
title: "Power Automate Approval Actions: Reminders option"
date: 2024-07-25T09:53:05+01:00
tags: ["Power Automate", "Approval", "Reminders"]
featured_image: '/posts/images/powerautomate-approval-action-reminders/approval.png'
omit_header_text: true
draft: true
---

Power Automate offers an approval action to fulfil business approval requirement. This post covers some of the action settings.

## The Pros of using approval action
 
**Interactive Emails**: Built-in, interactive emails with buttons and a text box for approvers to provide outcomes and comments.
**Parallel Approvals**: Supports parallel approval processes out of the box.
**Mobile Integration**: Integrated with the mobile app for on-the-go approvals.
**Centralized Management**: Manage approvals through flow.Microsoft.com.
**Push Notifications**: Receive app push notifications for approval requests.

## The Cons of approval action

**Limited Customisation**: Emails are not customizable, only the request details can be modified.
**Visibility Issues**: No visibility into existing approvals beyond your own.
**No Built-in Reminders**: Does not provide overdue reminders out of the box.
**No Serial Process**: No out-of-the-box support for serial approval processes; must be manually built.
**Internal Approvers Only**: Approvers must be within the organization.
**Timeout Limit**: Power Autpmate flow times out after 30 days due to a time out though the approval task stays active in the background.

## Approval Action Types

### Create an Approval

Behavior: Starts the approval process without waiting for a response.

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
