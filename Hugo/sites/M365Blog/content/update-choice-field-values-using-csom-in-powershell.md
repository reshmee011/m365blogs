---
title: 'Update Choice Field values using CSOM in PowerShell'
date: Wed, 13 Jul 2016 16:55:32 +0000
draft: false
tags: ['CastTo MakeGenericMethod', 'CSOM', 'FieldChoice', 'PowerShell', 'SharePoint 2013', 'SharePoint add-in', 'SharePoint Online']
---

I wanted a CSOM script in PowerShell to add a new value to a choice field in SharePoint. Most of the examples I found were in CSOM Managed Code (C#) but could not find anything in PowerShell. I came up with the embedded code. The client assemblies are referenced

*    SharePoint Assemblies\\Microsoft.SharePoint.Client.dll
*   SharePoint Assemblies\\Microsoft.SharePoint.Client.Runtime.dll

I have created the function Add-ChoiceValueToField to contain the logic to add a value to a choice field. The Field object has to be cast into FieldChoice to expose the Choices property. In PowerShell the CastTo method can be accessed using the MakeGenericMethod. `[Microsoft.SharePoint.Client.ClientContext].GetMethod("CastTo").MakeGenericMethod([Microsoft.SharePoint.Client.FieldChoice]).Invoke($ctx,$field)` I have used the  Load-CSOMProperties function from script [Load-CSOMProperties.ps1](https://gist.github.com/glapointe/cc75574a1d4a225f401b)  to load the existing property "Choices" which an immutable collection.  [Load-CSOMProperties.ps1](https://gist.github.com/glapointe/cc75574a1d4a225f401b)  is available from blog post [Loading Specific Values Using Lambda Expressions and the SharePoint CSOM API with Windows PowerShell](https://www.itunity.com/article/loading-specific-values-lambda-expressions-sharepoint-csom-api-windows-powershell-1249) to help with querying object properties like Lambda expressions in C#. `Load-CSOMProperties -object $fieldChoice -propertyNames @("Choices") ;` I have saved the collection in an array object $FieldChoiceValues to which the existing choice values have been added. If the value to be added does not exist, then it is added to the array object  $FieldChoiceValues. Finally the Field Choices property is updated with the array object $FieldChoiceValues `$fieldChoice.Choices = $FieldChoiceValues; $fieldChoice.Update()` The function can be called as follows `Add-ChoiceValueToField -ctx $ctx -ListTitle "ListTestManually" -FieldName "TypeNum" -ChoiceValue "7"`  

https://gist.github.com/reshmee011/7dec75c5206e4c68ded457bf40a3b32e