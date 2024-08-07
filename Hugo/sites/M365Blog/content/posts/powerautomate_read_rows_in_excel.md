---
title: "Read Rows in Excel using Power Automate using 'Create Table' Action"
date: 2024-08-07T16:54:24+01:00
tags: ["Power Automate","Import Data","Excel for Business", "OFFSET Excel function"]
featured_image: '/posts/images/powerautomate_read_rows_in_excel/CreateTable.PNG'
omit_header_text: true
draft: true
---

I had an excel file with few columns to be imported into SharePoint List.  These excel files are automatically generated using an export process from an external system and it is not option to manually update the file to transform the data into table. There are two options if the data is not already in table format.

1. OfficeScript : However OfficeScript are saved in user's OneDrive and needs to be written in TypeScript adding complexity to deployment and flow logic. 
2. Use action `Create Table`: This converts vanilla data within the spreadsheet into a table which can be used subsequently by the action `List rows present in a table`

![Raw Data](../images/powerautomate_read_rows_in_excel/Excel_RawData.png)

## Create Table

For an indefinitive number of rows within the file, use the offset function to consider all the rows within the file, 

`=OFFSET(<SheetName>!A1,0,0,SUBTOTAL(103,<SheetName>!$A:$A),<Numberofcolumns>)`

1. <SheetName>!A1 - this is the reference point where we want to start
2.,0,0 - this says from that reference point above, move down 0 rows and over 0 columns, effectively starting at A1.
3. SUBTOTAL(103,Sheet1!$A:$A) - this represents the number of rows we want in our range base on column A, the "103" tells SUBTOTAL we want a count of the active rows (rows with data) in column A, and informs OFFSET we want the number returned from SUBTOTAL(103,Sheet1!$A:$A) as the number of rows in our range.
4. <Numberofcolumns> - these are the number of columns in the data range 

|Property|Value|
|---|---|
|Location|_The SharePoint site URL_|
|Document Library|<Document Library Drive Id>|
|File|triggerBody()?['{Identifier}']|
|Table range|=OFFSET(Sheet1!A1,0,0,SUBTOTAL(103,Sheet1!$A:$A),3)|
|Table name|<tablename>|

![Create table](../images/powerautomate_read_rows_in_excel/CreateTable.png)

Result

![Table](../images/powerautomate_read_rows_in_excel/Excel_Table.png)

## List rows present in a table

![List rows present in a table](../images/powerautomate_read_rows_in_excel/ListRows_Table.png)

If the file being created is triggering the flow, the file can be referenced by `triggerBody()?['{Identifier}']`

Refer the same table name in the table parameter, for instance if table Sheet1 was created in previous step , enter Sheet1 in table name.

|Property|Value|
|---|---|
|Location|_The SharePoint site URL_|
|Document Library|<Document Library Drive Id>|
|File|triggerBody()?['{Identifier}']|
|Table|<tablename>|

The drive Id is populated
![Drive Id](../images/powerautomate_get-library-drive-id/libraryconnection.png)

## Add subsequent actions

I added a `Apply to each` to loop through excel row to create an item to a SharePoint list.

![Loop import data](../images/powerautomate_read_rows_in_excel/Loop_ImportData.png)

## Reference

![Power Automate - How to create Excel table dynamically (Excel Formula)?](https://www.youtube.com/watch?v=Q4Q_OWEa-Jw)