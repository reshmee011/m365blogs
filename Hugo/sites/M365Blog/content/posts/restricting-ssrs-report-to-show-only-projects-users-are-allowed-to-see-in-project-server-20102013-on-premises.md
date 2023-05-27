---
title: 'Restricting SSRS Report to show only projects users are allowed to see in Project Server 2010/2013 On Premises'
date: Sun, 17 Jan 2016 19:02:05 +0000
draft: false
tags: ['Project Server 2013', 'SSRS', 'SSRS Project Server', 'Uncategorized']
---

To restrict the projects users can see from SSRS, there are several options

### 1\. Use the published database.

However Microsoft does not recommend querying directly published database. This approach can be used at your own risk. `CREATE FUNCTION [dbo].[PWA_FN_UTILITY_GetProjectsUserCanViewProjectDetails] ( @WRES_NT_ACCOUNT nvarchar(255) ) RETURNS @OUTPUT TABLE ( ProjectUID uniqueidentifier ) AS BEGIN DECLARE @RES_UID uniqueidentifier -- get the user account from the published database, also available in reporting database -- but in reporting database no indication if user is just a resource or an actual user. -- REVISIT if necessary. SELECT @RES_UID = res.RES_UID FROM PUB.[dbo].MSP_RESOURCES res WHERE res.WRES_ACCOUNT = @WRES_NT_ACCOUNT AND res.RES_IS_WINDOWS_USER = 1 IF (@RES_UID IS NOT NULL) -- if user exists and is recognised, filter projects accordingly BEGIN INSERT INTO @OUTPUT SELECT p.ProjectUID FROM [dbo].MSP_EpmPROJECT p INNER JOIN MO_Project_Server_Published.[dbo].MSP_WEB_FN_SEC_GetAllProjectsResCanViewByViewID(@RES_UID, 'E98573C8-7EA6-41AB-8EA4-FF8DA9730D0A', NULL, 3) AS p1 ON p1.PROJ_UID = p.ProjectUID END -- else return empty table RETURN END`

### 2\. Using the PSI

#### 2.1 Create a XML data connection pointing to the PSI API

The PSI can be accessed from <PWA URL>/\_vti\_bin/psi/project.asmx Create a Shared Data Source of type XML. Enter the URL of the PSI into "Connection string"

#### ![PSIDataSource_XML](https://reshmeeauckloo.files.wordpress.com/2016/01/psidatasource_xml1.png)2.2 Create DataSet to get standard Project

The method  "ReadProjectStatus" from PSI can be used to query projects the users have access to view. However only one particular project type can be passed as parameter. The different project types are explained from https://msdn.microsoft.com/en-us/library/microsoft.office.project.server.library.project.projecttype\_di\_pj14mref.aspx

*   Create a Blank Report pointing to the Shared Data Source created in previous step.
*   Create an embedded dataset using the shared data source with the following query to get standard projects. Standard Projects have \[Project Type\] 0.

`<Query> <Method Namespace="http://schemas.microsoft.com/office/project/server/webservices/Project/" Name="ReadProjectStatus"> <Parameters> <Parameter Name="projType"><DefaultValue>0</DefaultValue></Parameter> </Parameters> </Method> <SoapAction>http://schemas.microsoft.com/office/project/server/webservices/Project/ReadProjectStatus</SoapAction> <ElementPath IgnoreNamespaces="true">ReadProjectStatusResponse/ReadProjectStatusResult/diffgram/ProjectDataSet/Project{PROJ_NAME,PROJ_UID,PROJ_TYPE}</ElementPath> </Query>`

*   Update Name to ProjectListDataSet\_0

![DataSet_Properties_ProjectList](https://reshmeeauckloo.files.wordpress.com/2016/01/dataset_properties_projectlist1.png)

#### 2.3 Create DataSet to get Proposal Project

Follow the same steps as 2.2 to create an embedded dataset.  Name to ProjectListDataSet\_4. Proposal Projects have \[Project Type\] 4 Enter Query <Query> <Method Namespace="http://schemas.microsoft.com/office/project/server/webservices/Project/" Name="ReadProjectStatus"> <Parameters> <Parameter Name="projType"><DefaultValue>4</DefaultValue></Parameter> </Parameters> </Method> <SoapAction>http://schemas.microsoft.com/office/project/server/webservices/Project/ReadProjectStatus</SoapAction> <ElementPath IgnoreNamespaces="true">ReadProjectStatusResponse/ReadProjectStatusResult/diffgram/ProjectDataSet/Project{PROJ\_NAME,PROJ\_UID,PROJ\_TYPE}</ElementPath> </Query>  

#### 2.4 Create DataSet to get Sub Project

Follow the same steps as 2.2 to create an embedded dataset.  Name to ProjectListDataSet\_5. Sub Projects have \[Project Type\] 5 Enter Query <Query> <Method Namespace="http://schemas.microsoft.com/office/project/server/webservices/Project/" Name="ReadProjectStatus"> <Parameters> <Parameter Name="projType"><DefaultValue>5</DefaultValue></Parameter> </Parameters> </Method> <SoapAction>http://schemas.microsoft.com/office/project/server/webservices/Project/ReadProjectStatus</SoapAction> <ElementPath IgnoreNamespaces="true">ReadProjectStatusResponse/ReadProjectStatusResult/diffgram/ProjectDataSet/Project{PROJ\_NAME,PROJ\_UID,PROJ\_TYPE}</ElementPath> </Query>  

#### 2.5 Create DataSet to get Master Project

Follow the same steps as 2.2 to create an embedded dataset.  Name to ProjectListDataSet\_6. Sub Projects have \[Project Type\] 6 Enter Query <Query> <Method Namespace="http://schemas.microsoft.com/office/project/server/webservices/Project/" Name="ReadProjectStatus"> <Parameters> <Parameter Name="projType"><DefaultValue>6</DefaultValue></Parameter> </Parameters> </Method> <SoapAction>http://schemas.microsoft.com/office/project/server/webservices/Project/ReadProjectStatus</SoapAction> <ElementPath IgnoreNamespaces="true">ReadProjectStatusResponse/ReadProjectStatusResult/diffgram/ProjectDataSet/Project{PROJ\_NAME,PROJ\_UID,PROJ\_TYPE}</ElementPath> </Query>

#### 2.6 Create hidden Parameter for Standard Projects

*   Create a Parameter named HiddenProjectUID\_0.
*   Check "Allow multiple values"
*   Select "Hidden" under "Select parameter visibility"

 

*   Click tab "Available Values", select option "Get values from a query" and pick DataSet "ProjectListPSIDataSet\_1".
*   Select PROJ\_UID for Value field and PROJ\_Name for Label field.

![AvailableValues_HiddenProjectUID_0](https://reshmeeauckloo.files.wordpress.com/2016/01/availablevalues_hiddenprojectuid_01.png)

*   Click tab "Default Values"

Select option "Get values from a query" and pick same dataset selected for "Available Values". Select DataSet "ProjectListPSIDataSet\_1" and pick "PROJ\_UID" from value field. ![DefaultValues_HiddenProjectUID_0](https://reshmeeauckloo.files.wordpress.com/2016/01/defaultvalues_hiddenprojectuid_02.png) Click on OK to create the dataset.

#### 2.7 Create hidden Parameter for Proposal Projects

Repeat the same steps as 2.6 but pick DataSet ProjectListPSIDataSet\_4 instead of DataSet ProjectListPSIDataSet\_1. Rename the parameter to HiddenProjectUID\_4.

#### 2.7 Create hidden Parameter for SubProject Projects

Repeat the same steps as 2.6 but pick DataSet ProjectListPSIDataSet\_5 instead of DataSet ProjectListPSIDataSet\_1. Rename the parameter to HiddenProjectUID\_5.

#### 2.8 Create hidden Parameter for Master Projects

Repeat the same steps as 2.6 but pick DataSet ProjectListPSIDataSet\_6 instead of DataSet ProjectListPSIDataSet\_1. Rename the parameter to HiddenProjectUID\_6.

#### 2.9 Create hidden parameter that will join the 4 parameters created above.

*   Create an embedded dataset. Update Name to "TempProjetUID".
*   Check "Allow multiple values".
*   Select Option "Hidden".

![TempProjectUID](https://reshmeeauckloo.files.wordpress.com/2016/01/tempprojectuid1.png)

*   Click on tab "Available Values". Select option "Specify values".

Under Label column, click the fx button and enter `=Split(Join(Parameters!HiddenProjectUID_0.Value,",") + iif(Parameters!HiddenProjectUID_4 is nothing,"", "," + Join(Parameters!HiddenProjectUID_4.Value,",")) + iif(Parameters!HiddenProjectUID_5 is nothing,"", "," + Join(Parameters!HiddenProjectUID_5.Value,",")) + iif(Parameters!HiddenProjectUID_6 is nothing,"", "," + Join(Parameters!HiddenProjectUID_6.Value,",")) ,",")` Under Value column, click the fx button and enter `=Split(Join(Parameters!HiddenProjectUID_0.Value,",") + iif(Parameters!HiddenProjectUID_4 is nothing,"", "," + Join(Parameters!HiddenProjectUID_4.Value,",")) + iif(Parameters!HiddenProjectUID_5 is nothing,"", "," + Join(Parameters!HiddenProjectUID_5.Value,",")) + iif(Parameters!HiddenProjectUID_6 is nothing,"", "," + Join(Parameters!HiddenProjectUID_6.Value,",")) ,",")` ![AvailableValues_TempProjectUID](https://reshmeeauckloo.files.wordpress.com/2016/01/availablevalues_tempprojectuid1.png)   Click on tab "Default Values". Select option "Specify values" and enter after clicking on fx button \=Split(Join(Parameters!HiddenProjectUID\_0.Value,",") + iif(Parameters!HiddenProjectUID\_4 is nothing,"", "," + Join(Parameters!HiddenProjectUID\_4.Value,",")) + iif(Parameters!HiddenProjectUID\_5 is nothing,"", "," + Join(Parameters!HiddenProjectUID\_5.Value,",")) + iif(Parameters!HiddenProjectUID\_6 is nothing,"", "," + Join(Parameters!HiddenProjectUID\_6.Value,",")) ,",") ![DefaultValues_TempProjectUID](https://reshmeeauckloo.files.wordpress.com/2016/01/defaultvalues_tempprojectuid1.png)

#### 2.9 Create a Shared Data Source pointing the Reporting Database.

#### 2.10 Create dataset to return ProjectName and ProjectUID using parameter TmpProjectUID

Create a dataset using the data source pointing to the reporting database. Rename the Dataset to "ProjectListDataSet. Select Option "Text". Enter the SQL query `SELECT ProjectUID, ProjectName FROM dbo.MSP_EpmProject_Userview WHERE ProjectUID in (@ProjectUID) ORDER BY ProjectName ![ProjectListDataSet_Prop_1](https://reshmeeauckloo.files.wordpress.com/2016/01/projectlistdataset_prop_1.png)` Click on Parameters tab. Map the parameter @ProjectUID to @TempProjectUID. ![Parameters_ProjectListDataSet_Prop_1.png](https://reshmeeauckloo.files.wordpress.com/2016/01/parameters_projectlistdataset_prop_1.png)

#### 2.11 Create visible parameter  to display list of projects user has access to.

Create parameter . Rename it to "Projects". Click on tab "Available Values" . Pick option "Get values from a query". Pick the dataset "ProjectListDataSet" created in step 2.10 . Pick ProjectUID from Value field and ProjectName from Label field ![AvailableValu_ProjectUID_Param](https://reshmeeauckloo.files.wordpress.com/2016/01/availablevalu_projectuid_param2.png)

#### 2.12 Run the report

The parameter Project display only projects the user can view.

### 3\. Use REST or CSOM in managed code

Though I have not tried it, CSOM or REST (http://ServerName/pwaName/\_api/ProjectServer/Projects) can be used in C# Managed Code to return list of projects as an array object which can be bound to a parameter.