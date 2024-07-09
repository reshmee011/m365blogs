---
title: "Power Automate updating multi line field with more than 255 characters"
tags: ["Power Automate","MultiLine Text field","SharePoint"]
date: 2024-07-07T17:12:39+01:00
featured_image: '/posts/images/powerautomate_field_more_255_characters/FieldMoreThan255Characters.png'
omit_header_text: true
draft: false
---

# Power Automate flow succeeded despite failure to update multi line text field within a SharePoint Library 

Updating multi-line text fields in a SharePoint Library from Power Automate flows, especially when the content exceeds 255 characters might fail if the field is not updated to allow unlimited length. This can cause workflows to fail silently.

In a recent scenario, an action was added to a Power Automate flow to update a SharePoint Library's multi-line text field named 'ErrorLog' with error details captured during the flow's execution. Despite the flow's apparent success, the 'ErrorLog' field remained unchanged.

![Update Error Log](../images/powerautomate_field_more_255_characters/updatestatustofailed.png)

The SharePoint update action's run log revealed the underlying issue:

> "This field can have no more than 255 characters."

![Flow succeeded despite failure to update multi line text field](../images/powerautomate_field_more_255_characters/FieldMoreThan255Characters.png)

## The Solution

The key to resolving this issue lies in adjusting the field settings in SharePoint to allow for unlimited text length in multi-line text fields.

### Steps to Allow Unlimited Length:

1. Navigate to the SharePoint list or library settings.
2. Click on the multi-line text field you wish to modify (e.g., 'ErrorLog').
3. In the field settings, find the option labeled **"Allow unlimited length in document libraries"** and enable it.

![Allow Unlimited length](../images/powerautomate_field_more_255_characters/ModernEdit_AllowUnlimitedLength.png)

This change effectively removes the 255 character limit, allowing Power Automate to update the field with more than 255 characters.

## Conclusion

The default 255-character limit in SharePoint's multi-line text fields can cause issues from Power Automate flows. However, by adjusting the field settings to allow for unlimited text length can resolve the issue