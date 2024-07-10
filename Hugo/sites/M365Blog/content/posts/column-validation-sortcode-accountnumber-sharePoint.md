---
title: "Column Validation for Sort Code and Account Number in SharePoint"
date: 2024-07-04T14:49:19+01:00
tags: ["AccountNumbeSortCode", "Column Validation","SharePoint","Sort Code","Account Number"]
featured_image: '/posts/images/column-validation-sortcode-accountnumber-sharePoint/ColumnValidationSortCode.PNG'
omit_header_text: true
draft: false
---

# Column Validation for Sort Code and Account Number in SharePoint

Column validation provides a solution to validate data for data integrity. This post covers column validation for Sort Code and Account Number in SharePoint.

## Validation for Sort Code

> =AND(LEN([Sort Code])=6,ISNUMBER([Sort Code]+0))

The above formula checks
- The length of the sort code must be exactly 6 digits.
- The sort code must be a number.

![Column Validation for Sort Code](../images/column-validation-sortcode-accountnumber-sharePoint/ColumnValidationSortCode.png)

## Validation for Account Number

> =AND(LEN([Account Number])=8,ISNUMBER([Account Number]+0))

The above formula checks 
- The account number must be exactly 8 digits long.
- The account number must be numeric.

## Error Message

If incorrect sort code or account number are entered, the end user will be presented with an error message.
![Error Message](../images/column-validation-sortcode-accountnumber-sharePoint/ErrorMessage.png)
