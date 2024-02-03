---
title: 'Invoke-PnPListDesign to create instances of lists libraires'
date: 2024-02-03T01:52:50Z
draft: false
tags: ['SPO', 'List Design', 'Document library','PnP', 'PowerShell' ]
---

# Custom document library template using PnP PowerShell
 
[Creating custom list templates](https://docs.microsoft.com/en-us/sharepoint/lists-custom-template?wt.mc_id=MVP_308367) is now possible to create both custom document libraries and lists although official microsoft documentation has not specified anything about supporting custom document library templates.

This article explores the option how to use a combination of list design and PowerShell script to provision multiple instances of document libraries using a CSV file and how to create a document library from a custom list template from UI.

First we can create a list design for our library based on an existing configured document library with custom content types, fields and views.

Create a List design for the document library using **Get-PnPSiteScriptFromList** and **Add-PnPListDesign**

```PowerShell
$siteUrl = "https://contoso.sharepoint.com/sites/Company311"
$doc = "testtemplate"

Connect-pnponline $adminSiteUrl -Interactive
Get-PnPList -Identity "testtemplate" | Get-PnPSiteScriptFromList | Add-PnPSiteScript -Title "testtemplate doc lib" | Add-PnPListDesign -Title "testtemplate doc lib" -Description "Deploy testtemplate doc lib" -ListColor Pink -ListIcon BullseyeTarget

get-pnplistdesign
```

From the UI , the list design for the custom library is available when clicking on New >List and selecting the tab "From your organization" under "Templates"

[![](https://reshmeeauckloo.files.wordpress.com/2021/10/image.png?w=421)](https://reshmeeauckloo.files.wordpress.com/2021/10/image.png)

Click on the custom list design to select it , click on "Use Template" button and enter a name and optionally a description for the library. 

Once "Create" is clicked the library is created with the relevant views , content types and fields.

That's great that we can create a document library using the custom list/library template, however there are some settings like versionings , indexed columns, permissions, etc.. which are not included in the list design template. 

I have used a PowerShell script with the cmdlet Invoke-PnPListDesign to apply the site design recursively to create multiple instances of the document library updating the internal name and display name based on the csv file and update the settings like creating indexed columns and setting versioning.

Sample CSV file format saved as libraries.csv

```
InternalName,DisplayName
AR,Annual Reports
CR,Credit Risk
Audit,Audit
PO,Purchase Orders
```

Execute the **Invoke-PnPListDesign **cmdlet

```
[CmdletBinding()] 
    Param(
    [Parameter(Mandatory=$false,  Position=0)]
    [String]$adminSiteUrl = "https://contoso-admin.sharepoint.com",
    [Parameter(Mandatory=$false,  Position=1)]
    [String]$siteUrl =  "https://contoso.sharepoint.com/sites/investment",
    [Parameter(Mandatory=$false,  Position=2)]
    [String]$librariesCSV =  "C:\Scripts\DocumentLibraryTemplate\libraries_1.csv",

    [Parameter(Mandatory=$false,  Position=4)]
    [String]$listDesignId = "5b38e500-0fab-4da7-b011-ad7113228920" # use Get-SPOListDesign to find the Id of the list design containing the document library template
  )
#creating indexed columns might help with performance of large libraries, i.e. >5000 files
function Create-Index ($list, $targetFieldName)
{
  $targetField = Get-PnPField -List $list -Identity $targetFieldName
  $targetField.Indexed = 1;
  $targetField.Update();
  $list.Context.ExecuteQuery();
}

Connect-PnPOnline -Url $siteUrl -Interactive
Import-Csv $librariesCSV | ForEach-Object {
Invoke-PnPListDesign -Identity $listDesignId -WebUrl $siteUrl
#Get library just created and update Internal name and display name
$lib = Get-PnPList -Identity "test_ct" -Includes RootFolder
while(!$lib)
{
 $lib = Get-PnPList -Identity "test_ct" -Includes RootFolder
 sleep -second 5
}
if($lib)
{
    $lib.Rootfolder.MoveTo($($_.InternalName))  
    Invoke-PnPQuery  
    #this will change library title  
    Set-PnPList -Identity $lib.Id -Title $($_.DisplayName)
    #add list to quick launch
    Add-PnPNavigationNode -Title $_.DisplayName -Url $($_.InternalName + "/") -Location "QuickLaunch"
    #enable versioning on the library
    Set-PnPList -Identity $lib.Id -EnableVersioning $True -EnableMinorVersions $True -MajorVersions 500 -MinorVersions 10
    Write-host "`tSetting versioning to major/minor to :"$_.DisplayName
    Create-Index $lib "Created By"
    Create-Index $lib "Modified"
 }
} 
```

Screenshot of the output running the script. **OutPut**

```
PS C:\Windows\system32> C:\Scripts\DocumentLibraryTemplate\ApplySiteDesignToCreateLibaries.ps1

Title                                        OutcomeText                            Outcome
-----                                        -----------                            -------
Create site column WorkAddress through XML                                          Success
Create site column _Status through XML                                              Success
Create site column digits through XML                                               Success
Create site column remarks through XML                                              Success
Create site column workinghours through XML                                         Success
Create site column Progress through XML                                             Success
Create content type Legal                                                              NoOp
Add site column WorkAddress to content type                                            NoOp
Add site column _Status to content type                                                NoOp
Create content type test_210304                                                        NoOp
Add site column digits to content type                                                 NoOp
Add site column remarks to content type                                                NoOp
Add site column workinghours to content type                                           NoOp
Add site column Progress to content type                                               NoOp
Add site column _Status to content type                                                NoOp
Create content type test_StatusComm                                                    NoOp
Add site column _Status to content type                                                NoOp
Create content type test_11                                                            NoOp
Add site column _Status to content type                                                NoOp
Create or update library "test_ct"           List with name test_ct already exists.    NoOp
Add list column "ActualWork"                 List with name test_ct already exists.    NoOp
Add list column "Initials"                   List with name test_ct already exists.    NoOp
Add list column "_Status"                    List with name test_ct already exists.    NoOp
Add list column "digits"                     List with name test_ct already exists.    NoOp
Add list column "remarks"                    List with name test_ct already exists.    NoOp
Add list column "workinghours"               List with name test_ct already exists.    NoOp
Add list column "Progress"                   List with name test_ct already exists.    NoOp
Add list column "_Comments"                  List with name test_ct already exists.    NoOp
Add list column "TriggerFlowInfo"            List with name test_ct already exists.    NoOp
Add list column "SelectFilename"             List with name test_ct already exists.    NoOp
Add content type "Document"                  List with name test_ct already exists.    NoOp
Add content type "Folder"                    List with name test_ct already exists.    NoOp
Add content type "Legal"                     List with name test_ct already exists.    NoOp
Add content type "test_210304"               List with name test_ct already exists.    NoOp
Add content type "test_StatusComm"           List with name test_ct already exists.    NoOp
Add content type "test_11"                   List with name test_ct already exists.    NoOp
Add view "All Documents"                     List with name test_ct already exists.    NoOp
Add view "All Documents sorted"              List with name test_ct already exists.    NoOp
Annual Reports1                                                                            
	Setting versioning to major/minor to : Annual Reports1 
```

Screenshot of the libraries created from UI (TestCreatingFromUserInterface) and the others created via script (Annual Reports, Audit, Credit Risk and Purchase Orders)

[![](https://reshmeeauckloo.files.wordpress.com/2021/10/image-1.png?w=1024)](https://reshmeeauckloo.files.wordpress.com/2021/10/image-1.png)

The custom template for list/library can save a lot of time deploying standardised lists/document libraries without having to manually configure views, fields and content types.

## References

[For admins: Add-SPOListDesign (Microsoft.Online.SharePoint.PowerShell) | Microsoft Learn](https://learn.microsoft.com/powershell/module/sharepoint-online/add-spolistdesign?view=sharepoint-ps?wt.mc_id=MVP_308367)

[For end users: Custom list templates - SharePoint in Microsoft 365 | Microsoft Learn][https://learn.microsoft.com/sharepoint/lists-custom-template?wt.mc_id=MVP_308367]