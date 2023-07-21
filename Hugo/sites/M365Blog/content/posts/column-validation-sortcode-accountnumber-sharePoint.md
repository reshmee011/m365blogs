---
title: "Column Validation for sort code and Account Number"
date: 2023-07-04T14:49:19+01:00
tags: ["AccountNumbeSortCode", "Column Validation","SharePoint Page"]
draft: false
---

# Column Validation
Sort Code

=AND(LEN([Sort Code])=6,ISNUMBER([Sort Code]+0))

![Column Validation for Sort Code](../images/column-validation-sortcode-accountnumber-sharePoint/ColumnValidationSortCode.png)

Account Number
=AND(LEN([Account Number])=8,ISNUMBER([Account Number]+0))

