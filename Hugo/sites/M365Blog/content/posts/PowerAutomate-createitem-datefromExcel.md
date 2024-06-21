---
title: "Importing data from Excel file into SharePoint list in Power Automate"
date: 2024-06-21T07:17:21+01:00
tags: ["PowerAutomate", "Excel Business Online","SharePoint", "Data Import"]
featured_image: '/posts/images/PowerAutomate_SavingApprovalDetailsToSharePoint/AllFieldsUpdatedCorrectly.png'
draft: true
---

# Importing data from Excel file into SharePoint list in Power Automate

If ever there are columns of datetime format and number format, while importing the data using the action "List rows present in a table" as part of the "Excel Online for Business" connector.

![Excel Online for Business](../images/PowerAutomate-createitem-datefromExcel/ExcelOnline_ListRowsPresentInaTable.png).

The flow is quite simple with an action to read contents from excel file and an action to create items into a sharepoint list.

## Date Time data
![Import data from Excel into SP List](../images/PowerAutomate-createitem-datefromExcel/ImportDataFromExcelIntoSPList.png)

The action "Create Item" might fail with the following error due to the date format

**The 'inputs.parameters' of workflow operation 'Create_item' of type 'OpenApiConnection' is not valid. Error details: Input parameter 'item/StartDate' is required to be of type 'String/date'. The runtime value '"44378"' to be converted doesn't have the expected format 'String/date'.**

![Issue Date 44378](../images/PowerAutomate-createitem-datefromExcel/Date_44378.png)

The fix for the issue is to update a setting on the **Excel Online for Business** action.

![Date Format ISO8601.png](../images/PowerAutomate-createitem-datefromExcel/DateFormatISO8601.png)

## Double data

If there are Number/double column which is empty , the data import will fail with errors of type **OpenApiOperationParameterTypeConversionFailed**.

**The 'inputs.parameters' of workflow operation 'Create_item' of type 'OpenApiConnection' is not valid. Error details: Input parameter 'item/ActualTotalCost' is required to be of type 'Number/double'. The runtime value '""' to be converted doesn't have the expected format 'Number/double'.**

The fix is to add a condition for the Number/double field to cater for null values before importing the data.

![Cost Double](../images/PowerAutomate-createitem-datefromExcel/PA_Cost_Double.png)

```yaml
if(empty(items('Foreach')?['Actual Total Cost']), null, items('Foreach')?['Actual Total Cost'])
```



