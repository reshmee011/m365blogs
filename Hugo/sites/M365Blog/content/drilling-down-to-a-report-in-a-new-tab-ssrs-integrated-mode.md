---
title: 'Drilling Down to a Report in a new tab SSRS Integrated Mode'
date: Tue, 23 Feb 2016 21:41:02 +0000
draft: false
tags: ['SSRS', 'SubReport Drilling']
---

There are two ways to navigate to another SSRS from a main Report

1.  Action “Go to Report”

*   Right-click on a TextBox, select Text Box Properties, select Action tab and select option “Go to report”.

![GoToReport](https://reshmeeauckloo.files.wordpress.com/2016/02/gotoreport.png)

*   Click on “Browse” and select the SSRS Report from the report library.
*   If the selected SSRS Report has parameters, click on “Add” to add parameter. From the “Name” dropdown select the parameter name and from the “Value” dropdown select the value to pass to the parameter.

2.  Action “Go to URL”: This option is ideal to open a SSRS report in a new tab

*   Right-click on a TextBox, select Text Box Properties, select Action tab and select option “Go to URL”.

![GoToURL](https://reshmeeauckloo.files.wordpress.com/2016/02/gotourl.png)

*   Click on “fx” button under “Select URL”
*   Type the URL of the report in the following format

\= “<PWASiteURL>/\_vti\_bin/reportserver? <PWASiteURL>/ <relativeSSRSURL>&<ParameterName>=<ParameterValue>” Example of a link to a report named “Project Overall Status” =LCase(Globals!ReportServerUrl) & “?” & Replace(LCase(Globals!ReportServerUrl),"/\_vti\_bin/reportserver","") & “/ProjectBICenter/SSRS Reports Library/Project Overall Status.rdl&ParmProjectName=" & LCase(Fields!ProjectUID\_STR.Value) The built in field Globals!ReportServerUrl returns the URL where the report is rendered.

*   If you want the report to open in a new tab, add the JavaScript function window.open

Type the URL of the report in the following format _\="javascript:void(window.open('<PWASiteURL>/\_vti\_bin/reportserver? <PWASiteURL>/ <relativeSSRSURL>&<ParameterName>=<ParameterValue> ','\_blank'));"_ Example of a link to a report “Project Overall Status” in DEV environment with the JavaScript code _\="javascript:void(window.open('" & LCase(Globals!ReportServerUrl) & "?" & Replace(LCase(Globals!ReportServerUrl),"/\_vti\_bin/reportserver","")  & "/ProjectBICenter/SSRS Reports Library/Project Overall Status.rdl&ParmProjectName=" & LCase(Fields!ProjectUID\_STR.Value) & "','\_blank'));"_