---
title: "Power Automate: Save Run Flow Url to list Item"
date: 2024-08-07T12:52:18+01:00
tags: ["Power Automate","Save Flow Url","View Formatting"]
featured_image: '/posts/images/powerautomate_save_run_Flow_url_back_toListitem_on_error/ErrorflowUrl.png'
omit_header_text: true
draft: true
---

In the Power Automate flow running, the flow url can be captured and saved to the SharePoint List Item making it easier to go back to review in case failures or auditing. Bear in mind any flows more than 30 days old won't be available , hence the link saved may not work.

## Save the flow Url

The current flow Url can be captured and saved to a variable

> https://flow.microsoft.com/manage/environments/workflow().tags.environmentName/flows/workflow().name/runs/workflow().run.name

![Workflow Url](../images/powerautomate_save_run_Flow_url_back_toListitem_on_error/ErrorflowUrl.png)

Save the workflow url to the SharePoint list item

![Workflow Url](../images/powerautomate_save_run_Flow_url_back_toListitem_on_error/Update_To_Error.png)

Result of the save action to the list item

![Workflow Url](../images/powerautomate_save_run_Flow_url_back_toListitem_on_error/link_column.png)

## Apply View Formatting to improve the view

Copy the snippet into the column formatting to improve the view

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/sp/v2/column-formatting.schema.json",
  "elmType": "a",
  "txtContent": "=if(@currentField!='','View processing details','')",
  "attributes": {
    "target": "_blank",
    "href": "@currentField"
  }
}
```

## References

[Hyperlink Column JSON](https://tomriha.com/how-to-update-sharepoint-hyperlink-column-in-power-automate/)

[Storing a link into Sharepoint List](https://powerusers.microsoft.com/t5/Building-Flows/Storing-a-link-into-Sharepoint-List-Hyperlink-Column/m-p/1636264#M181989) 