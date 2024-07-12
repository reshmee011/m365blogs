---
title: "Importing Dates and Numbers from Excel file into SharePoint list with Power Automate"
date: 2024-07-11T07:17:21+01:00
tags: ["PowerAutomate", "Excel Business Online","SharePoint", "Data Import"]
featured_image: '/posts/images/PowerAutomate-createitem-datefromExcel/ImportDataFromExcelIntoSPList.png'
omit_header_text: true
draft: false
---

# Importing Dates and Numbers from Excel file into SharePoint list with Power Automate

Transferring data from Excel to SharePoint lists can encounter format issues, especially with datetime and number fields. This post covers solutions to some challenges particularly related to dates and numbers.

![Excel Online for Business](../images/PowerAutomate-createitem-datefromExcel/ImportDataFromExcelIntoSPList.png)

The flow is quite simple with an action to read contents from excel file and an action to create items into a sharepoint list.

## Handling DateTime Fields

When importing datetime data with the "List rows present in a table" action from the "Excel Online for Business" connector, you might encounter format related errors.

![Excel Online for Business](../images/PowerAutomate-createitem-datefromExcel/ExcelOnline_ListRowsPresentInaTable.png)

The error typically appears when the "Create Item" action in SharePoint fails due to the date format not being recognized:

{{< warning >}}
The 'inputs.parameters' of workflow operation 'Create_item' of type 'OpenApiConnection' is not valid. Error details: Input parameter 'item/StartDate' is required to be of type 'String/date'. The runtime value '"44378"' to be converted doesn't have the expected format 'String/date'.**
{{< /warning >}}

![Issue Date 44378](../images/PowerAutomate-createitem-datefromExcel/Date_44378.png)

### Solution: Adjusting Date Format

The resolution is to adjust the setting on the **Excel Online for Business** action to interpret dates in the ISO8601 format.

![Date Format ISO8601.png](../images/PowerAutomate-createitem-datefromExcel/DateFormatISO8601.png)

## Handling Number

Importing empty number/double columns can lead to failures with errors of type **OpenApiOperationParameterTypeConversionFailed**.

{{< warning >}}
The 'inputs.parameters' of workflow operation 'Create_item' of type 'OpenApiConnection' is not valid. Error details: Input parameter 'item/ActualTotalCost' is required to be of type 'Number/double'. The runtime value '""' to be converted doesn't have the expected format 'Number/double'.
{{< /warning >}}

### Solution: Handling Null Values

The fix is to add a condition for the Number/double field to cater for null values before importing the data.

![Cost Double](../images/PowerAutomate-createitem-datefromExcel/PA_Cost_Double.png)

```yaml
if(empty(items('Foreach')?['Actual Total Cost']), null, items('Foreach')?['Actual Total Cost'])
```