---
title: 'Export Lists Schema to XML using CSOM and PnP'
date: Sat, 22 Oct 2016 09:36:01 +0000
draft: false
tags: ['CSOM', 'PnP', 'PnP Exporting Lists', 'PowerShell', 'SharePoint 2013', 'SharePoint Online']
---

I had to automate deployment of lists and libraries into different environments, e.g. on premises and online. There are different elements to consider when deploying lists or libraries namely - List Properties - List Content Types - Views - Fields I could have written a configuration file in xml manually to specify the details above. Instead I have decided to configure lists and libraries from a SharePoint 2013 On Premises Dev environment and export the list schemaXML property to a XML file. The list's SchemaXML property is not accessible using SharePoint CSOM assemblies. Instead a [remote procedure call using HTTP Get request ](https://msdn.microsoft.com/en-us/library/dd587743%28v=office.11%29.aspx?f=255&MSPPError=-2147217396 "URL Protocol SharePoint RPC") has to be executed. http://_Server\_Name_/\[sites/\]\[_Site\_Name_/\]\_vti\_bin/owssvr.dll?Cmd=ExportList&List=_GUID_ I am interested only in custom lists, I have built a list of out of the box lists/libraries which do not need to be exported, i.e. \_catalogs,Lists/ContentTypeSyncLog,/IWConvertedForms,FormServerTemplates,Lists/PublishedFeed,/ProjectPolicyItemList,SiteAssets,SitePages,Style Library,Lists/TaxonomyHiddenList. I have used the PnP PowerShell commands

*    To connect to SharePoint

Connect-SPOnline -Url $SiteUrl -CurrentCredentials

*    To retrieve all lists

```
$SPOLists = Get\-SPOList
```Download SharePointPnPPowerShell2016.msi from https://github.com/OfficeDev/PnP-PowerShell/releases to use PnP PowerShell Commands. After all the lists are retrieved, if the list is not in the exclusions criteria, its schemaXML is downloaded using HTTP get request with URL and to XML file. $ListSchemaUrl = "$SiteUrl/\_vti\_bin/owssvr.dll?Cmd=ExportList&List={" + $list.Id.ToString() + "}"; The script can be downloaded from [technet gallery](https://gallery.technet.microsoft.com/Export-Lists-Schema-as-XML-c472d82c)

```
 Param( 
 \[Parameter(Mandatory=$true)\] 
 \[string\]$SiteUrl, 
 \[Parameter(Mandatory=$false)\] 
 \[string\]$XMLListsFileName = "SiteLists.xml" 
 ) 
##OOB SharePoint Lists to exclude from Export 
 $ListsNameToExclude  = "\_catalogs,Lists/ContentTypeSyncLog,/IWConvertedForms,FormServerTemplates,Lists/PublishedFeed,/ProjectPolicyItemList,SiteAssets,SitePages,Style Library,Lists/TaxonomyHiddenList"   
 $ArrayListtoExcl = $ListsNameToExclude.Split(",");  
Set-Location $PSScriptRoot 
 
function LoadAndConnectToSharePoint($url) 
{ 
  ##Using PnP library 
  Connect\-SPOnline \-Url $SiteUrl \-CurrentCredentials 
  $spContext =  Get\-SPOContext 
  return $spContext 
} 
 
$Context = LoadAndConnectToSharePoint $SiteUrl 
 
$SPOLists = Get\-SPOList 
$PathToExportXMLSiteLists = $PSScriptRoot 
 
$xmlFilePath = "$PathToExportXMLSiteLists\\$XMLListsFileName" 
 
 #Create Export Files 
 New-Item $xmlFilePath \-type file \-force 
 
 #Export Site Columns to XML file 
 Add-Content $xmlFilePath "<?xml version=\`"1.0\`" encoding=\`"utf\-8\`"?>" 
 Add-Content $xmlFilePath "\`n<Lists>" 
 
foreach($list in $SPOLists) 
{ 
  $exclude = $false; 
  foreach($lsEx in $ArrayListtoExcl) 
  { 
   if($list.DefaultViewUrl \-match $lsEx) 
   { 
    $exclude =$true; 
    break; 
   } 
  } 
  if($exclude  \-eq $false) 
  { 
    $ListSchemaUrl = "$SiteUrl/\_vti\_bin/owssvr.dll?Cmd=ExportList&List={" + $list.Id.ToString() + "}"; 
    $listSchemaXML = Invoke\-WebRequest \-Uri $ListSchemaUrl \-UseDefaultCredentials 
    Add-Content $xmlFilePath $listSchemaXML; 
  } 
} 
Add-Content $xmlFilePath "</Lists>" 
 
 

```

Provide the siteURL (url of the SharePoint site from which you want to export list definitions) parameter value and optionally provide the XMLListsFileName (name of te XML file to store the list schema) parameter value. The output of running the script is as below. To create lists/libraries using the XML config file, refer to post[ Create Lists Libraries from Schema as XML using CSOM](https://reshmeeauckloo.wordpress.com/2016/10/22/create-lists-libraries-from-schema-as-xml-using-csom/)

```
<?xml version\="1.0" encoding\="utf-8"?> 
 
<Lists\>   <List Name\="{999137BC-274B-4F74-AF93-BA1068806BEE}" Title\="Documents" Description\="" Direction\="0" BaseType\="1" FeatureId\="{00BFEA71-E717-4E80-AA17-D0C71B360101}" SendToLocation\="|" ServerTemplate\="101" Url\="Shared Documents" DisableAttachments\="TRUE" EnableContentTypes\="TRUE" NavigateForFormsPages\="TRUE" BrowserFileHandling\="permissive" Version\="7"\>     <MetaData\>       <Views\>         <View Name\="{B6022AAF-88E4-4DA2-BECC-416178FC8ABB}" DefaultView\="TRUE" MobileView\="TRUE" MobileDefaultView\="TRUE" Type\="HTML" DisplayName\="All Documents" Url\="Shared Documents/Forms/AllItems.aspx" Level\="1" BaseViewID\="1" ContentTypeID\="0x" ImageUrl\="/\_layouts/15/images/dlicon.png?rev=23"\>           <ViewFields\>             <FieldRef Name\="DocIcon"/> 
            <FieldRef Name\="LinkFilename"/> 
            <FieldRef Name\="Modified"/> 
            <FieldRef Name\="MigrationSourceURL"/> 
            <FieldRef Name\="Level0"/> 
            <FieldRef Name\="Level1"/> 
          </ViewFields> 
          <ViewData/> 
          <Query\>             <OrderBy\>               <FieldRef Name\="FileLeafRef"/> 
            </OrderBy> 
          </Query> 
          <Aggregations Value\="Off"/> 
          <RowLimit Paged\="TRUE"\>30</RowLimit> 
          <Mobile MobileItemLimit\="3" MobileSimpleViewField\="LinkFilename"/> 
          <XslLink\>main.xsl</XslLink> 
          <JSLink\>clienttemplates.js</JSLink> 
          <Toolbar Type\="Standard"/> 
          <ParameterBindings\>             <ParameterBinding Name\="NoAnnouncements" Location\="Resource(wss,noitemsinview\_doclibrary)"/> 
            <ParameterBinding Name\="NoAnnouncementsHowTo" Location\="Resource(wss,noitemsinview\_doclibrary\_howto2)"/> 
          </ParameterBindings> 
        </View> 
        <View Name\="{43D7A4A0-85DC-4BAF-A317-1B7114C80E2B}" Type\="HTML" Hidden\="TRUE" TabularView\="FALSE" DisplayName\="Merge Documents" Url\="Shared Documents/Forms/Combine.aspx" Level\="1" BaseViewID\="7" ContentTypeID\="0x" ToolbarTemplate\="MergeToolBar" ImageUrl\="/\_layouts/15/images/dlicon.png?rev=23"\>           <XslLink\>main.xsl</XslLink> 
          <Query\>             <OrderBy\>               <FieldRef Name\="FileLeafRef"/> 
            </OrderBy> 
          </Query> 
          <ViewFields\>             <FieldRef Name\="DocIcon"/> 
            <FieldRef Name\="LinkFilename"/> 
            <FieldRef Name\="Combine"/> 
            <FieldRef Name\="Modified"/> 
            <FieldRef Name\="Editor"/> 
          </ViewFields> 
          <RowLimit Paged\="TRUE"\>30</RowLimit> 
          <JSLink\>clienttemplates.js</JSLink> 
          <Toolbar Type\="Standard"/> 
          <ParameterBindings\>             <ParameterBinding Name\="NoAnnouncements" Location\="Resource(wss,noitemsinview\_doclibrary)"/> 
            <ParameterBinding Name\="NoAnnouncementsHowTo" Location\="Resource(wss,noitemsinview\_doclibrary\_howto2)"/> 
          </ParameterBindings> 
        </View> 
        <View Name\="{9E9F7CB8-CBC6-44DC-92E7-E43231556A9D}" Type\="HTML" Hidden\="TRUE" TabularView\="FALSE" DisplayName\="Relink Documents" Url\="Shared Documents/Forms/repair.aspx" Level\="1" BaseViewID\="9" ContentTypeID\="0x" ToolbarTemplate\="RelinkToolBar" ImageUrl\="/\_layouts/15/images/dlicon.png?rev=23"\>           <XslLink\>main.xsl</XslLink> 
          <Query\>             <OrderBy\>               <FieldRef Name\="FileLeafRef"/> 
            </OrderBy> 
            <Where\>               <Neq\>                 <FieldRef Name\="xd\_Signature"/> 
                <Value Type\="Boolean"\>1</Value> 
              </Neq> 
            </Where> 
          </Query> 
          <ViewFields\>             <FieldRef Name\="DocIcon"/> 
            <FieldRef Name\="LinkFilenameNoMenu"/> 
            <FieldRef Name\="RepairDocument"/> 
            <FieldRef Name\="Modified"/> 
            <FieldRef Name\="Editor"/> 
            <FieldRef Name\="ContentType"/> 
            <FieldRef Name\="TemplateUrl"/> 
          </ViewFields> 
          <RowLimit Paged\="TRUE"\>30</RowLimit> 
          <Toolbar Type\="Standard"/> 
          <ParameterBindings\>             <ParameterBinding Name\="NoAnnouncements" Location\="Resource(wss,noitemsinview\_doclibrary)"/> 
            <ParameterBinding Name\="NoAnnouncementsHowTo" Location\="Resource(wss,noitemsinview\_doclibrary\_howto2)"/> 
          </ParameterBindings> 
        </View> 
        <View Name\="{086F7F40-F379-43C8-97DD-618EF9E7B0E1}" Type\="HTML" Hidden\="TRUE" DisplayName\="assetLibTemp" Url\="Shared Documents/Forms/Thumbnails.aspx" Level\="1" BaseViewID\="40" ContentTypeID\="0x" ImageUrl\="/\_layouts/15/images/dlicon.png?rev=23"\>           <Query\>             <OrderBy\>               <FieldRef Name\="LinkFilename"/> 
            </OrderBy> 
          </Query> 
          <ViewFields\>             <FieldRef Name\="LinkFilename"/> 
          </ViewFields> 
          <RowLimit\>20</RowLimit> 
        </View> 
        <View Name\="{952ADDDF-B3F3-4FC9-A676-457BF6FAB561}" Type\="HTML" Hidden\="TRUE" DisplayName\="" Url\="SitePages/Home.aspx" Level\="1" BaseViewID\="50" ContentTypeID\="0x"\>           <XslLink Default\="TRUE"\>main.xsl</XslLink> 
          <JSLink\>clienttemplates.js</JSLink> 
          <RowLimit Paged\="TRUE"\>15</RowLimit> 
          <Toolbar Type\="Standard"/> 
          <ViewFields\>             <FieldRef Name\="DocIcon"/> 
            <FieldRef Name\="LinkFilename"/> 
          </ViewFields> 
          <ParameterBindings\>             <ParameterBinding Name\="NoAnnouncements" Location\="Resource(wss,noitemsinview\_doclibrary)"/> 
            <ParameterBinding Name\="NoAnnouncementsHowTo" Location\="Resource(wss,noitemsinview\_doclibrary\_howto2)"/> 
          </ParameterBindings> 
          <Query\>             <OrderBy\>               <FieldRef Name\="Modified" Ascending\="FALSE"/> 
            </OrderBy> 
          </Query> 
        </View> 
      </Views> 
      <Fields\>         <Field ID\="{1d22ea11-1e32-424e-89ab-9fedbadb6ce1}" ColName\="tp\_ID" RowOrdinal\="0" ReadOnly\="TRUE" Type\="Counter" Name\="ID" DisplayName\="ID" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="ID" FromBaseType\="TRUE"/> 
        <Field ID\="{03e45e84-1992-4d42-9116-26f756012634}" RowOrdinal\="0" Type\="ContentTypeId" Sealed\="TRUE" ReadOnly\="TRUE" Hidden\="TRUE" DisplayName\="Content Type ID" Name\="ContentTypeId" DisplaceOnUpgrade\="TRUE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="ContentTypeId" ColName\="tp\_ContentTypeId" FromBaseType\="TRUE"/> 
        <Field ID\="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Type\="Computed" DisplayName\="Content Type" Name\="ContentType" DisplaceOnUpgrade\="TRUE" RenderXMLUsingPattern\="TRUE" Sortable\="FALSE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="ContentType" Group\="\_Hidden" PITarget\="MicrosoftWindowsSharePointServices" PIAttribute\="ContentTypeID" FromBaseType\="TRUE"\>           <FieldRefs\>             <FieldRef Name\="ContentTypeId"/> 
          </FieldRefs> 
          <DisplayPattern\>             <MapToContentType\>               <Column Name\="ContentTypeId"/> 
            </MapToContentType> 
          </DisplayPattern> 
        </Field> 
        <Field ID\="{8c06beca-0777-48f7-91c7-6da68bc07b69}" ColName\="tp\_Created" RowOrdinal\="0" ReadOnly\="TRUE" Type\="DateTime" Name\="Created" DisplayName\="Created" StorageTZ\="TRUE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="Created" FromBaseType\="TRUE"/> 
        <Field ID\="{1df5e554-ec7e-46a6-901d-d85a3881cb18}" ColName\="tp\_Author" RowOrdinal\="0" ReadOnly\="TRUE" Type\="User" List\="UserInfo" Name\="Author" DisplayName\="Created By" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="Author" FromBaseType\="TRUE"/> 
        <Field ID\="{28cf69c5-fa48-462a-b5cd-27b6f9d2bd5f}" ColName\="tp\_Modified" RowOrdinal\="0" ReadOnly\="TRUE" Type\="DateTime" Name\="Modified" DisplayName\="Modified" StorageTZ\="TRUE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="Modified" FromBaseType\="TRUE"/> 
        <Field ID\="{d31655d1-1d5b-4511-95a1-7a09e9b75bf2}" ColName\="tp\_Editor" RowOrdinal\="0" ReadOnly\="TRUE" Type\="User" List\="UserInfo" Name\="Editor" DisplayName\="Modified By" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="Editor" FromBaseType\="TRUE"/> 
        <Field ID\="{26d0756c-986a-48a7-af35-bf18ab85ff4a}" ColName\="tp\_HasCopyDestinations" RowOrdinal\="0" Sealed\="TRUE" Hidden\="TRUE" ReadOnly\="TRUE" Type\="Boolean" Name\="\_HasCopyDestinations" DisplaceOnUpgrade\="TRUE" DisplayName\="Has Copy Destinations" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="\_HasCopyDestinations" FromBaseType\="TRUE"/> 
        <Field ID\="{6b4e226d-3d88-4a36-808d-a129bf52bccf}" ColName\="tp\_CopySource" RowOrdinal\="0" Sealed\="TRUE" Hidden\="FALSE" ReadOnly\="TRUE" Type\="Text" Name\="\_CopySource" DisplaceOnUpgrade\="TRUE" DisplayName\="Copy Source" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="\_CopySource" FromBaseType\="TRUE"/> 
        <Field ID\="{fdc3b2ed-5bf2-4835-a4bc-b885f3396a61}" ColName\="tp\_ModerationStatus" RowOrdinal\="0" ReadOnly\="TRUE" Type\="ModStat" Name\="\_ModerationStatus" DisplayName\="Approval Status" Hidden\="TRUE" CanToggleHidden\="TRUE" Required\="FALSE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="\_ModerationStatus" FromBaseType\="TRUE"\>           <CHOICES\>             <CHOICE\>0;#Approved</CHOICE> 
            <CHOICE\>1;#Rejected</CHOICE> 
            <CHOICE\>2;#Pending</CHOICE> 
            <CHOICE\>3;#Draft</CHOICE> 
            <CHOICE\>4;#Scheduled</CHOICE> 
          </CHOICES> 
          <Default\>0</Default> 
        </Field> 
        <Field ID\="{34ad21eb-75bd-4544-8c73-0e08330291fe}" ReadOnly\="TRUE" Type\="Note" Name\="\_ModerationComments" DisplayName\="Approver Comments" Hidden\="TRUE" CanToggleHidden\="TRUE" Filterable\="FALSE" Sortable\="FALSE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="\_ModerationComments" FromBaseType\="TRUE" ColName\="ntext1" RowOrdinal\="0"/> 
        <Field ID\="{94f89715-e097-4e8b-ba79-ea02aa8b7adb}" Name\="FileRef" ReadOnly\="TRUE" Hidden\="TRUE" Type\="Lookup" DisplayName\="URL Path" List\="Docs" FieldRef\="ID" ShowField\="FullUrl" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="FileRef" FromBaseType\="TRUE"/> 
        <Field ID\="{56605df6-8fa1-47e4-a04c-5b384d59609f}" Name\="FileDirRef" ReadOnly\="TRUE" Hidden\="TRUE" Type\="Lookup" DisplayName\="Path" List\="Docs" FieldRef\="ID" ShowField\="DirName" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="FileDirRef" FromBaseType\="TRUE"/> 
        <Field ID\="{173f76c8-aebd-446a-9bc9-769a2bd2c18f}" Name\="Last\_x0020\_Modified" ReadOnly\="TRUE" Hidden\="TRUE" DisplayName\="Modified" Type\="Lookup" List\="Docs" FieldRef\="ID" ShowField\="TimeLastModified" Format\="TRUE" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="Last\_x0020\_Modified" FromBaseType\="TRUE"/> 
        <Field ID\="{998b5cff-4a35-47a7-92f3-3914aa6aa4a2}" Name\="Created\_x0020\_Date" ReadOnly\="TRUE" Hidden\="TRUE" DisplayName\="Created" Type\="Lookup" List\="Docs" FieldRef\="ID" ShowField\="TimeCreated" Format\="TRUE" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="Created\_x0020\_Date" FromBaseType\="TRUE"/> 
        <Field ID\="{8fca95c0-9b7d-456f-8dae-b41ee2728b85}" Name\="File\_x0020\_Size" Hidden\="TRUE" ReadOnly\="TRUE" Type\="Lookup" DisplayName\="File Size" List\="Docs" FieldRef\="ID" ShowField\="SizeInKB" Format\="TRUE" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="File\_x0020\_Size" FromBaseType\="TRUE"/> 
        <Field ID\="{30bb605f-5bae-48fe-b4e3-1f81d9772af9}" Name\="FSObjType" ReadOnly\="TRUE" Hidden\="TRUE" ShowInFileDlg\="FALSE" Type\="Lookup" DisplayName\="Item Type" List\="Docs" FieldRef\="ID" ShowField\="FSType" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="FSObjType" FromBaseType\="TRUE"/> 
        <Field ID\="{423874f8-c300-4bfb-b7a1-42e2159e3b19}" Name\="SortBehavior" ReadOnly\="TRUE" Hidden\="TRUE" ShowInFileDlg\="FALSE" Type\="Lookup" DisplayName\="Sort Type" List\="Docs" FieldRef\="ID" ShowField\="SortBehavior" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="SortBehavior" FromBaseType\="TRUE"/> 
        <Field ID\="{ba3c27ee-4791-4867-8821-ff99000bac98}" Name\="PermMask" DisplaceOnUpgrade\="TRUE" ReadOnly\="TRUE" Hidden\="TRUE" RenderXMLUsingPattern\="TRUE" ShowInFileDlg\="FALSE" Type\="Computed" DisplayName\="Effective Permissions Mask" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="PermMask" FromBaseType\="TRUE"\>           <FieldRefs\>             <FieldRef Name\="ID"/> 
          </FieldRefs> 
          <DisplayPattern\>             <CurrentRights/> 
          </DisplayPattern> 
        </Field> 
        <Field ID\="{a7b731a3-1df1-4d74-a5c6-e2efba617ae2}" Name\="CheckedOutUserId" ReadOnly\="TRUE" Hidden\="TRUE" ShowInFileDlg\="FALSE" Type\="Lookup" DisplayName\="ID of the User who has the item Checked Out" List\="Docs" FieldRef\="ID" ShowField\="CheckoutUserId" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="CheckedOutUserId" FromBaseType\="TRUE"/> 
        <Field ID\="{cfaabd0f-bdbd-4bc2-b375-1e779e2cad08}" Name\="IsCheckedoutToLocal" DisplaceOnUpgrade\="TRUE" ReadOnly\="TRUE" Hidden\="TRUE" ShowInFileDlg\="FALSE" Type\="Lookup" DisplayName\="Is Checked out to local" List\="Docs" FieldRef\="ID" ShowField\="IsCheckoutToLocal" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="IsCheckedoutToLocal" FromBaseType\="TRUE"/> 
        <Field ID\="{3881510a-4e4a-4ee8-b102-8ee8e2d0dd4b}" ColName\="tp\_CheckoutUserId" RowOrdinal\="0" ReadOnly\="TRUE" Type\="User" List\="UserInfo" Name\="CheckoutUser" DisplaceOnUpgrade\="TRUE" DisplayName\="Checked Out To" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="CheckoutUser" FromBaseType\="TRUE"/> 
        <Field ID\="{8553196d-ec8d-4564-9861-3dbe931050c8}" ShowInFileDlg\="FALSE" ShowInVersionHistory\="FALSE" Type\="File" Name\="FileLeafRef" DisplayName\="Name" AuthoringInfo\="(for use in forms)" List\="Docs" FieldRef\="ID" ShowField\="LeafName" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" Required\="TRUE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="FileLeafRef" FromBaseType\="TRUE"/> 
        <Field ID\="{4b7403de-8d94-43e8-9f0f-137a3e298126}" Name\="UniqueId" DisplaceOnUpgrade\="TRUE" ReadOnly\="TRUE" Hidden\="TRUE" ShowInFileDlg\="FALSE" Type\="Lookup" DisplayName\="Unique Id" List\="Docs" FieldRef\="ID" ShowField\="UniqueId" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="UniqueId" FromBaseType\="TRUE"/> 
        <Field ID\="{6d2c4fde-3605-428e-a236-ce5f3dc2b4d4}" Name\="SyncClientId" DisplaceOnUpgrade\="TRUE" Hidden\="TRUE" ReadOnly\="TRUE" DisplayName\="Client Id" ShowInFileDlg\="FALSE" Type\="Lookup" List\="Docs" FieldRef\="ID" ShowField\="SyncClientId" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="SyncClientId" FromBaseType\="TRUE"/> 
        <Field ID\="{c5c4b81c-f1d9-4b43-a6a2-090df32ebb68}" Name\="ProgId" DisplaceOnUpgrade\="TRUE" ReadOnly\="TRUE" Hidden\="TRUE" ShowInFileDlg\="FALSE" Type\="Lookup" DisplayName\="ProgId" List\="Docs" FieldRef\="ID" ShowField\="ProgId" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="ProgId" FromBaseType\="TRUE"/> 
        <Field ID\="{dddd2420-b270-4735-93b5-92b713d0944d}" Name\="ScopeId" DisplaceOnUpgrade\="TRUE" ReadOnly\="TRUE" Hidden\="TRUE" ShowInFileDlg\="FALSE" Type\="Lookup" DisplayName\="ScopeId" List\="Docs" FieldRef\="ID" ShowField\="ScopeId" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="ScopeId" FromBaseType\="TRUE"/> 
        <Field ID\="{4a389cb9-54dd-4287-a71a-90ff362028bc}" Name\="VirusStatus" Hidden\="TRUE" ReadOnly\="TRUE" Type\="Lookup" DisplayName\="Virus Status" List\="Docs" FieldRef\="ID" ShowField\="Size" Format\="TRUE" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="VirusStatus" FromBaseType\="TRUE"/> 
        <Field ID\="{9d4adc35-7cc8-498c-8424-ee5fd541e43a}" Name\="CheckedOutTitle" Hidden\="TRUE" ReadOnly\="TRUE" Type\="Lookup" DisplayName\="Checked Out To" List\="Docs" FieldRef\="ID" ShowField\="CheckedOutTitle" Format\="TRUE" JoinColName\="DoclibRowId" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="CheckedOutTitle" FromBaseType\="TRUE"/> 
        <Field ID\="{58014f77-5463-437b-ab67-eec79532da67}" Name\="\_CheckinComment" DisplaceOnUpgrade\="TRUE" ReadOnly\="TRUE" Type\="Lookup" DisplayName\="Check In Comment" List\="Docs" FieldRef\="ID" ShowField\="CheckinComment" Filterable\="FALSE" Format\="TRUE" JoinColName\="DoclibRowId" JoinType\="INNER" Sortable\="FALSE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="\_CheckinComment" FromBaseType\="TRUE"/> 
        <Field ID\="{e2a15dfd-6ab8-4aec-91ab-02f6b64045b0}" ReadOnly\="TRUE" Hidden\="TRUE" Type\="Computed" Name\="LinkCheckedOutTitle" DisplayName\="Checked Out To" Filterable\="TRUE" AuthoringInfo\="(link to username to user details page)" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="LinkCheckedOutTitle" FromBaseType\="TRUE"\>           <FieldRefs\>             <FieldRef Name\="CheckedOutTitle"/> 
            <FieldRef Name\="CheckedOutUserId"/> 
          </FieldRefs> 
          <DisplayPattern\>             <IfEqual\>               <Expr1\>                 <Field Name\="CheckedOutUserId"/> 
              </Expr1> 
              <Expr2/> 
              <Then/> 
              <Else\>                 <HTML\><!\[CDATA\[<a href="\]\]></HTML> 
                <HttpVDir CurrentWeb="TRUE"/> 
                <HTML><!\[CDATA\[/\_layouts/15/userdisp.aspx?id=\]\]></HTML> 
                <Field Name="CheckedOutUserId"/> 
                <HTML><!\[CDATA\[">\]\]></HTML> 
                <Field HTMLEncode="TRUE" Name="CheckedOutTitle"/> 
                <HTML><!\[CDATA\[</a>\]\]></HTML> 
              </Else> 
            </IfEqual> 
          </DisplayPattern> 
        </Field> 
        <Field ID="{822c78e3-1ea9-4943-b449-57863ad33ca9}" ReadOnly="TRUE" Hidden="TRUE" Type="Text" Name="Modified\_x0020\_By" DisplayName="Document Modified By" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Modified\_x0020\_By" FromBaseType="TRUE" ColName="nvarchar10" RowOrdinal="0"/> 
        <Field ID="{4dd7e525-8d6b-4cb4-9d3e-44ee25f973eb}" ReadOnly="TRUE" Hidden="TRUE" Type="Text" Name="Created\_x0020\_By" DisplayName="Document Created By" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Created\_x0020\_By" FromBaseType="TRUE" ColName="nvarchar11" RowOrdinal="0"/> 
        <Field ID="{39360f11-34cf-4356-9945-25c44e68dade}" ReadOnly="TRUE" Hidden="TRUE" Type="Text" Name="File\_x0020\_Type" DisplayName="File Type" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="File\_x0020\_Type" FromBaseType="TRUE" ColName="nvarchar12" RowOrdinal="0"/> 
        <Field ID="{0c5e0085-eb30-494b-9cdd-ece1d3c649a2}" ReadOnly="TRUE" Hidden="TRUE" Type="Text" Name="HTML\_x0020\_File\_x0020\_Type" DisplayName="HTML File Type" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="HTML\_x0020\_File\_x0020\_Type" FromBaseType="TRUE" ColName="nvarchar13" RowOrdinal="0"/> 
        <Field ID="{c63a459d-54ba-4ab7-933a-dcf1c6fadec2}" Name="\_SourceUrl" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Text" DisplayName="Source URL" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_SourceUrl" FromBaseType="TRUE" ColName="nvarchar14" RowOrdinal="0"/> 
        <Field ID="{034998e9-bf1c-4288-bbbd-00eacfc64410}" Name="\_SharedFileIndex" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Text" DisplayName="Shared File Index" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_SharedFileIndex" FromBaseType="TRUE" ColName="nvarchar15" RowOrdinal="0"/> 
        <Field ID="{3c6303be-e21f-4366-80d7-d6d0a3b22c7a}" Hidden="TRUE" ReadOnly="TRUE" Type="Computed" Name="\_EditMenuTableStart" DisplaceOnUpgrade="TRUE" DisplayName="Edit Menu Table Start" ClassInfo="Menu" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_EditMenuTableStart" FromBaseType="TRUE"> 
          <FieldRefs> 
            <FieldRef Name="FileLeafRef"/> 
            <FieldRef Name="FileDirRef"/> 
            <FieldRef Name="FSObjType"/> 
            <FieldRef Name="ID"/> 
            <FieldRef Name="ServerUrl"/> 
            <FieldRef Name="HTML\_x0020\_File\_x0020\_Type"/> 
            <FieldRef Name="File\_x0020\_Type"/> 
            <FieldRef Name="PermMask"/> 
            <FieldRef Name="IsCheckedoutToLocal"/> 
            <FieldRef Name="CheckoutUser"/> 
            <FieldRef Name="\_SourceUrl"/> 
            <FieldRef Name="\_HasCopyDestinations"/> 
            <FieldRef Name="\_CopySource"/> 
            <FieldRef Name="ContentType"/> 
            <FieldRef Name="ContentTypeId"/> 
            <FieldRef Name="\_ModerationStatus"/> 
            <FieldRef Name="\_UIVersion"/> 
          </FieldRefs> 
          <DisplayPattern> 
            <HTML><!\[CDATA\[
```

                            " id="                            " Url="                            " DRef="                            " Perm="                            " type="                            " Ext="                            " Icon="                                              |                                            " OType="                            " COUId="                            " SRed="                                              |                                            " DefaultIO="                            " COut="                            " HCD="                            " CSrc="                            " MS="                                                                                    " UIS="                                          " SUrl="                            \]\]>                                                                                                              " id="                                                                                                              \]\]></HTML>              <HTML><!\[CDATA\[

\]\]>

```
\]\]> 
            \]\]> 
             \]\]> 
            \]\]></HTML> 
          </DisplayPattern> 
        </Field> 
        <Field ID="{9d30f126-ba48-446b-b8f9-83745f322ebe}" ReadOnly="TRUE" Type="Computed" Name="LinkFilenameNoMenu" DisplayName="Name" DisplayNameSrcField="FileLeafRef" Filterable="FALSE" AuthoringInfo="(linked to document)" ListItemMenuAllowed="Prohibited" LinkToItemAllowed="Prohibited" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="LinkFilenameNoMenu" FromBaseType="TRUE"> 
          <FieldRefs> 
            <FieldRef Name="FileLeafRef"/> 
            <FieldRef Name="FSObjType"/> 
            <FieldRef Name="Created\_x0020\_Date"/> 
            <FieldRef Name="FileRef"/> 
            <FieldRef Name="File\_x0020\_Type"/> 
            <FieldRef Name="HTML\_x0020\_File\_x0020\_Type"/> 
            <FieldRef Name="ContentTypeId"/> 
            <FieldRef Name="PermMask"/> 
            <FieldRef Name="CheckoutUser"/> 
            <FieldRef Name="IsCheckedoutToLocal"/> 
            <FieldRef Name="ServerUrl"/> 
          </FieldRefs> 
          <DisplayPattern> 
            <IfEqual> 
              <Expr1> 
                <LookupColumn Name="FSObjType"/> 
              </Expr1> 
              <Expr2>1</Expr2> 
              <Then> 
                <FieldSwitch> 
                  <Expr> 
                    <GetVar Name="RecursiveView"/> 
                  </Expr> 
                  <Case Value="1"> 
                    <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/> 
                  </Case> 
                  <Default> 
                    <SetVar Name="UnencodedFilterLink"> 
                      <SetVar Name="RootFolder"> 
                        <HTML>/</HTML> 
                        <LookupColumn Name="FileRef"/> 
                      </SetVar> 
                      <SetVar Name="SkipHost">1</SetVar> 
                      <SetVar Name="FolderCTID"> 
                        <FieldSwitch> 
                          <Expr> 
                            <ListProperty Select="EnableContentTypes"/> 
                          </Expr> 
                          <Case Value="1"> 
                            <Column Name="ContentTypeId"/> 
                          </Case> 
                        </FieldSwitch> 
                      </SetVar> 
                      <FilterLink Default="" Paged="FALSE"/> 
                    </SetVar> 
                    <HTML><!\[CDATA\[<a onfocus="OnLink(this)" href="\]\]></HTML> 
                    <GetVar Name="UnencodedFilterLink" HTMLEncode="TRUE"/> 
                    <HTML><!\[CDATA\[" onmousedown="javascript:VerifyFolderHref(this,event, '\]\]></HTML> 
                     
                       
                     
                     
                     
                       
                         
                       
                     
                     
                     
                       
                     
                     
                     
                       
                         
                        | 
                         
                       
                     
                     
                     
                       
                     
                     
                     
                       
                         
                        | 
                         
                       
                     
                     
                     
                     
                       
                     
                     
                     
                       
                     
                     
                     
                       
                         
                       
                     
                     
                     
                       
                         
                       
                     
                     
                     
                       
                         
                       
                     
                     
                     
                       
                     
                     
                     
                       
                         
                        | 
                         
                       
                     
                     
                     
                       
                     
                     
                     
                       
                         
                        | 
                         
                       
                     
                     
                     
                       
                     
                     
                     
                       
                     
                     
                     
                       
                     
                     
                     
                       
                     
                     
                     
                       
                     
                    \]\]> 
                     
                     
                       
                         
                       
                      1 
                       
                        \]\]> 
                       
                     
                    \]\]> 
                   
                 
               
               
                 
                 
                 
                 
                   
                 
                 
                 
                   
                     
                   
                 
                 
                 
                   
                     
                   
                 
                 
                 
                   
                     
                   
                 
                 
                 
                   
                 
                 
                 
                   
                     
                    | 
                     
                   
                 
                 
                 
                   
                 
                 
                 
                   
                     
                    | 
                     
                   
                 
                 
                 
                   
                 
                 
                 
                   
                 
                 
                 
                   
                 
                 
                 
                   
                 
                 
                 
                   
                 
                \]\]> 
                 
                   
                 
                 
                   
                     
                   
                  1 
                   
                    \]\]> 
                   
                 
                \]\]> 
                 
                   
                  New 
                  \]\]> 
                 
               
             
           
         
         
           
             
             
             
             
           
           
             
               
                 
               
               
                 
               
               
                 
                 
                \]\]> 
                 
                \]\]> 
                \]\]> 
                 \]\]> 
                \]\]> 
                 \]\]> 
                \]\]> 
               
             
           
         
         
           
             
             
             
             
           
           
             
               
                 
               
               
                 
               
               
                 
                 
                 
                 
                 
               
             
           
         
         
           
             
             
             
             
             
             
             
             
             
             
           
           
             
               
                 
                   
                     
                   
                  1 
                   
                     
                      0x0120D5 
                       
                         
                       
                       
                        Document Collection:  
                         
                       
                       
                        Folder:  
                         
                       
                     
                   
                   
                     
                   
                 
               
               
                 
                   
                     
                   
                   
                   
                     
                       
                         
                       
                      1 
                       
                         
                           
                             
                            | 
                             
                           
                           
                            | 
                           
                           
                            folder.gif 
                           
                           
                             
                               
                                 
                                | 
                                 
                               
                             
                             
                               
                                 
                               
                               
                                 
                               
                               
                                folder.gif 
                               
                               
                                 
                               
                             
                           
                         
                       
                       
                         
                           
                          | 
                           
                         
                       
                     
                   
                   
                     
                       
                     
                   
                 
               
               
               
               
               
               
               
              \]\]> 
             
             
               
                 
                   
                 
                 
                 
                   
                     
                       
                     
                     
                     
                       
                         
                         
                         
                       
                       
                       
                       
                       
                      \]\]> 
                     
                   
                 
                 
                   
                   
                     
                   
                  \]\]> 
                 
               
             
             
               
                 
               
              1 
               
                 
                   
                     
                   
                   
                     
                     
                   
                   
                     
                       
                        / 
                         
                       
                      1 
                       
                         
                           
                             
                           
                           
                             
                           
                         
                       
                       
                     
                     
                       
                         
                       
                       
                         
                         
                       
                       
                         
                         
                         
                         
                           
                         
                         
                         
                           
                             
                           
                         
                         
                         
                           
                         
                         
                         
                           
                             
                            | 
                             
                           
                         
                         
                         
                           
                         
                         
                         
                           
                             
                            | 
                             
                           
                         
                         
                         
                         
                           
                         
                         
                         
                           
                         
                         
                         
                           
                             
                           
                         
                         
                         
                           
                             
                           
                         
                         
                         
                           
                             
                           
                         
                         
                         
                           
                         
                         
                         
                           
                             
                            | 
                             
                           
                         
                         
                         
                           
                         
                         
                         
                           
                             
                            | 
                             
                           
                         
                         
                         
                           
                         
                         
                         
                           
                         
                         
                         
                           
                         
                         
                         
                           
                         
                         
                         
                           
                         
                        \]\]> 
                         
                         
                        \]\]> 
                       
                     
                   
                 
               
               
                 
                 
                 
                 
                   
                 
                 
                 
                   
                     
                   
                 
                 
                 
                   
                     
                   
                 
                 
                 
                   
                     
                   
                 
                 
                 
                   
                 
                 
                 
                   
                     
                    | 
                     
                   
                 
                 
                 
                   
                 
                 
                 
                   
                     
                    | 
                     
                   
                 
                 
                 
                   
                 
                 
                 
                   
                 
                 
                 
                   
                 
                 
                 
                   
                 
                 
                 
                   
                 
                \]\]> 
                 
                 
                \]\]> 
               
             
           
         
         
           
             
           
           
            / 
             
           
         
         
           
             
           
           
             
            / 
             
           
         
         
           
             
             
           
           
             
               
                 
               
              1 
               
                 
               
               
                 
                   
                 
               
             
           
         
         
           
             
             
           
           
             
               
                 
               
               
                 
                 KB 
               
             
           
         
         
         
         
         
         
         
         
         
           
             
           
           
             
               
                 
               
               
                 
               
               
                 
                Selected 
                \]\]> 
               
               
                 
                 
                 
                 
                   
                 
                 
                 
                 
                 
                   
                 
                \]\]> 
                 
                Normal 
                \]\]> 
                \]\]> 
               
             
           
         
         
           
             
           
           
             
               
                 
               
               
                 
               
               
                 
                Selected 
                \]\]> 
               
               
                 
                 
                 
                 
                   
                 
                 
                 
                 
                 
                   
                 
                \]\]> 
                 
                Normal 
                \]\]> 
                \]\]> 
               
             
           
         
         
           
             
             
             
           
           
             
               
                 
               
               
                 
                 
                 
                 
                   
                     
                   
                  1 
                   
                     
                       
                         
                       
                       
                       
                         
                           
                             
                           
                          1 
                           
                             
                           
                           
                             
                           
                         
                       
                       
                         
                       
                     
                   
                   
                     
                   
                 
                 
                 
                   
                 
                 
                 
                   
                 
                 
                 
                   
                 
                 
                 
                   
                 
                 
                 
                   
                 
                \]\]> 
                 
                Edit Document Properties 
                \]\]> 
                \]\]> 
               
               
                 
               
             
           
         
         
         
         
         
         
         
         
         
         
         
         
         
         
         
         
         
           
             
             
             
           
           
             
               
                 
               
              0 
               
                 
                 
                \]\]> 
                 
                 
                \]\]> 
                 
                 
                  | 
                   
                     
                   
                 
                \]\]> 
               
             
           
         
         
           
             
             
           
           
             
               
                 
               
              0 
               
                 
                 
                \]\]> 
               
             
           
         
         
         
         
       
       
         
           
           
             
             
             
             
             
             
             
             
             
             
             
           
           
            PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+RG9jdW1lbnRMaWJyYXJ5Rm9ybTwvRGlzcGxheT48RWRpdD5Eb2N1bWVudExpYnJhcnlGb3JtPC9FZGl0PjxOZXc+RG9jdW1lbnRMaWJyYXJ5Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4= 
           
         
         
           
             
             
             
             
             
           
           
            PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+TGlzdEZvcm08L0Rpc3BsYXk+PEVkaXQ+TGlzdEZvcm08L0VkaXQ+PE5ldz5MaXN0Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4= 
           
         
         
           
           
             
             
             
             
             
             
             
             
           
           
           
            PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+RG9jdW1lbnRMaWJyYXJ5Rm9ybTwvRGlzcGxheT48RWRpdD5Eb2N1bWVudExpYnJhcnlGb3JtPC9FZGl0PjxOZXc+RG9jdW1lbnRMaWJyYXJ5Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4= 
           
         
       
       
         
         
         
       
       
        1 
        1 
        0 
       
      Shared Documents/Forms/template.dotx 
       
         
          Content Following Item Updated Event Receiver 101 
          2 
          10002 
          10000 
          Microsoft.Office.Server.UserProfiles, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c 
          Microsoft.Office.Server.UserProfiles.ContentFollowingItemEventReceiver 
           
         
       
     
   
   
     
       
         
          main.xsl 
          clienttemplates.js 
          30 
           
           
             
             
             
             
           
           
             
             
           
           
             
               
             
           
         
         
          main.xsl 
          clienttemplates.js 
          30 
           
           
             
             
             
             
             
           
           
             
             
           
           
             
               
             
           
         
         
          main.xsl 
          30 
           
           
             
             
             
             
             
             
             
           
           
             
             
           
           
             
               
             
             
               
                 
                1 
               
             
           
         
         
           
             
           
          20 
           
             
               
             
           
         
         
           
             
             
             
             
           
           
           
           
          30 
           
          main.xsl 
          clienttemplates.js 
           
           
             
             
           
         
       
       
         
         
         
         
         
         
         
         
         
         
         
         
         
         
         
         
         
         
         
          FALSE 
         
         
          FALSE 
         
         
         
           
             
             
             
             
             
             
             
           
           
             
               
                 
               
               
               
                 
                   
                     
                   
                  1 
                   
                     
                     
                      / 
                       
                     
                    /\_w/ 
                     
                       
                     
                    \_ 
                     
                       
                     
                    .jpg 
                   
                   
                     
                       
                         
                       
                      0 
                       
                         
                         
                           
                          | 
                           
                         
                       
                       
                         
                           
                             
                           
                           
                           
                             
                             
                               
                              | 
                               
                             
                           
                           
                             
                             
                              / 
                               
                             
                            /\_w/ 
                             
                               
                             
                            \_ 
                             
                               
                             
                            .jpg 
                           
                         
                       
                     
                   
                 
               
               
                 
               
             
           
         
         
           
             
             
             
             
           
           
             
             
             
             
               
                 
               
               
               
                 
               
               
                 
               
             
             
             
            \]\]> 
           
         
         
           
             
           
           
             
           
         
         
           
             
             
             
           
           
             
               
                 
               
              0 
               
                 
                   
                     
                   
                   
                   
                   
                     
                       
                         
                       
                      0 
                       
                       
                        \]\]> 
                         
                         
                           
                         
                         
                        \]\]> 
                       
                     
                   
                 
               
             
           
         
         
           
             
             
             
             
             
             
             
           
           
             
               
                 
               
               
               
                 
                   
                     
                   
                  1 
                   
                     
                     
                      / 
                       
                     
                    /\_t/ 
                     
                       
                     
                    \_ 
                     
                       
                     
                    .jpg 
                   
                   
                     
                       
                         
                       
                      0 
                       
                         
                         
                           
                          | 
                           
                         
                       
                       
                         
                           
                             
                           
                           
                           
                             
                             
                               
                              | 
                               
                             
                           
                           
                             
                             
                              / 
                               
                             
                            /\_t/ 
                             
                               
                             
                            \_ 
                             
                               
                             
                            .jpg 
                           
                         
                       
                     
                   
                 
               
               
                 
               
             
           
         
         
           
             
           
           
             
               
                 
               
              0 
               
                 
                  fSelectFieldAppeared = true; firstIdWithCheckbox ='\]\]> 
                 
                 
                 
                   
                    <input type=checkbox disabled style="visibility:hidden" title='selection checkbox' name="selectionCheckBox" id="cbx\_\]\]> 
                </HTML> 
                <Column Name="ID"/> 
                <HTML><!\[CDATA\[" onfocus="if (!IsImgLibJssLoaded()) return; HiLiteRow(\]\]></HTML> 
                <Column Name="ID"/> 
                <HTML><!\[CDATA\[)" onclick="if (!IsImgLibJssLoaded()) return; ToggleSelection(\]\]></HTML> 
                <Column Name="ID"/> 
                <HTML><!\[CDATA\[)">\]\]></HTML> 
              </Then> 
            </IfEqual> 
          </DisplayPattern> 
        </Field> 
        <Field ID="{76d1cc87-56de-432c-8a2a-16e5ba5331b3}" ReadOnly="TRUE" Type="Computed" Group="\_Hidden" Name="NameOrTitle" ClassInfo="Menu" DisplayName="Name" DisplayNameSrcField="FileLeafRef" Filterable="FALSE" AuthoringInfo="(linked to display items)" SourceID="http://schemas.microsoft.com/sharepoint/v3/fields" StaticName="NameOrTitle" Customization=""> 
          <FieldRefs> 
            <FieldRef ID="{687C7F94-686A-42d3-9B67-2782EAC4B4F8}" Name="FileLeafRef"/> 
            <FieldRef ID="{56605DF6-8FA1-47e4-A04C-5B384D59609F}" Name="FileDirRef"/> 
            <FieldRef ID="{FA564E0F-0C70-4ab9-B863-0177E6DDD247}" Name="Title"/> 
            <FieldRef ID="{081C6E4C-5C14-4f20-B23E-1A71CEB6A67C}" Name="DocIcon"/> 
            <FieldRef ID="{94F89715-E097-4e8b-BA79-EA02AA8B7ADB}" Name="FileRef"/> 
            <FieldRef ID="{9DA97A8A-1DA5-4a77-98D3-4BC10456E700}" Name="Description"/> 
            <FieldRef ID="{30BB605F-5BAE-48fe-B4E3-1F81D9772AF9}" Name="FSObjType"/> 
            <FieldRef ID="{3881510A-4E4A-4ee8-B102-8EE8E2D0DD4B}" Name="CheckedOutUserId"/> 
            <FieldRef ID="{105F76CE-724A-4bba-AECE-F81F2FCE58F5}" Name="ServerUrl"/> 
            <FieldRef ID="{998b5cff-4a35-47a7-92f3-3914aa6aa4a2}" Name="Created\_x0020\_Date"/> 
          </FieldRefs> 
          <DisplayPattern> 
            <HTML> 
              <!\[CDATA\[ 
            <table cellspacing="0" class="ms-unselectedtitle" NameOrTitle="true" 
            Url="\]\]> 
            </HTML> 
            <Field Name="ServerUrl" URLEncodeAsURL="TRUE"/> 
            <HTML> 
              <!\[CDATA\["  
            DRef="\]\]> 
            </HTML> 
            <Field Name="FileDirRef"/> 
            <HTML> 
              <!\[CDATA\[" 
            COUId="\]\]> 
            </HTML> 
            <Field Name="CheckedOutUserId"/> 
            <HTML> 
              <!\[CDATA\[" 
            Type="\]\]> 
            </HTML> 
            <Column Name="HTML\_x0020\_File\_x0020\_Type"/> 
            <HTML> 
              &quot; 
              Icon=&quot; 
            </HTML> 
            <MapToAll> 
              <Column Name="HTML\_x0020\_File\_x0020\_Type"/> 
              <HTML>|</HTML> 
              <Column Name="File\_x0020\_Type"/> 
            </MapToAll> 
            <HTML>&quot; OType=&quot;</HTML> 
            <LookupColumn Name="FSObjType"/> 
            <HTML> 
              <!\[CDATA\[" 
                        Ext="\]\]> 
            </HTML> 
            <Column Name="File\_x0020\_Type"/> 
            <HTML><!\[CDATA\[" cellspacing="0" cellpadding="0"><tr id="title\]\]></HTML> 
            <Column Name="ID"/> 
            <HTML><!\[CDATA\[" onclick="if (!IsImgLibJssLoaded()) return; ClickRow(\]\]></HTML> 
            <Column Name="ID"/> 
            <HTML><!\[CDATA\[)" oncontextmenu="if (!IsImgLibJssLoaded()) return true; return ContextMenuOnRow(\]\]></HTML> 
            <Column Name="ID"/> 
            <HTML><!\[CDATA\[);" onmouseover="if (!IsImgLibJssLoaded()) return; MouseOverRow(\]\]></HTML> 
            <Column Name="ID"/> 
            <HTML><!\[CDATA\[)" onmouseout="if (!IsImgLibJssLoaded()) return; MouseOutRow(\]\]></HTML> 
            <Column Name="ID"/> 
            <HTML> 
              <!\[CDATA\[)"> 
            <td width="100%" class="ms-vb">\]\]> 
            </HTML> 
            <IfEqual> 
              <Expr1> 
                <LookupColumn Name="FSObjType"/> 
              </Expr1> 
              <Expr2>0</Expr2> 
              <Then> 
                <IfEqual> 
                  <Expr1> 
                    <LookupColumn Name="ImageWidth"/> 
                  </Expr1> 
                  <Expr2/> 
                  <Then> 
                    <HTML><!\[CDATA\[<a href="\]\]></HTML> 
                    <Field Name="EncodedAbsUrl"/> 
                    <HTML><!\[CDATA\[" onclick="if (!IsImgLibJssLoaded()) return; javascript:DisplayItemOnFileRef(\]\]></HTML> 
                    <Column Name="ID"/> 
                    <HTML><!\[CDATA\[);return false;" onmouseover="if (!IsImgLibJssLoaded()) return; javascript:MouseOverRow(\]\]></HTML> 
                    <Column Name="ID"/> 
                    <HTML><!\[CDATA\[)" target="\_self">\]\]></HTML> 
                    <UrlBaseName HTMLEncode="TRUE"> 
                      <LookupColumn Name="FileLeafRef"/> 
                    </UrlBaseName> 
                    <HTML><!\[CDATA\[</a>\]\]></HTML> 
                  </Then> 
                  <Else> 
                    <HTML><!\[CDATA\[<a href="\]\]></HTML> 
                    <URL Cmd="Display"/> 
                    <HTML><!\[CDATA\[" \]\]></HTML> 
                    <HTML><!\[CDATA\[onclick="if (!IsImgLibJssLoaded()) return; javascript:DisplayItemOnFileRef(\]\]></HTML> 
                    <Column Name="ID"/> 
                    <HTML><!\[CDATA\[);return false;" onmouseover="if (!IsImgLibJssLoaded()) return; javascript:MouseOverRow(\]\]></HTML> 
                    <Column Name="ID"/> 
                    <HTML><!\[CDATA\[)" target="\_self">\]\]></HTML> 
                    <UrlBaseName HTMLEncode="TRUE"> 
                      <LookupColumn Name="FileLeafRef"/> 
                    </UrlBaseName> 
                    <HTML><!\[CDATA\[</a>\]\]></HTML> 
                  </Else> 
                </IfEqual> 
                <IfNew Name="Created\_x0020\_Date"> 
                  <HTML><!\[CDATA\[<img src="/\_layouts/15/1033/images/new.gif" alt="\]\]></HTML> 
                  <HTML>New</HTML> 
                  <HTML><!\[CDATA\[" class="ms-newgif" />\]\]></HTML> 
                </IfNew> 
              </Then> 
              <Else> 
                <Switch> 
                  <Expr> 
                    <GetVar Name="RecursiveView"/> 
                  </Expr> 
                  <Case Value="1"> 
                    <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/> 
                  </Case> 
                  <Default> 
                    <HTML><!\[CDATA\[<a href="\]\]></HTML> 
                    <SetVar Name="RootFolder"> 
                      <HTML>/</HTML> 
                      <LookupColumn Name="FileRef"/> 
                    </SetVar> 
                    <FilterLink Default="" Paged="FALSE"/> 
                    <HTML><!\[CDATA\[" target="\_self">\]\]></HTML> 
                    <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/> 
                    <HTML><!\[CDATA\[</a>\]\]></HTML> 
                  </Default> 
                </Switch> 
              </Else> 
            </IfEqual> 
            <HTML> 
              <!\[CDATA\[</td> 
                <td class='ms-menuimagecell' style='visibility:hidden' width="10" id="menuTd\]\]> 
            </HTML> 
            <Column Name="ID"/> 
            <HTML><!\[CDATA\["><a id="menuHref\]\]></HTML> 
            <Column Name="ID"/> 
            <HTML><!\[CDATA\[" href="javascript:ClickRow(\]\]></HTML> 
            <Column Name="ID"/> 
            <HTML><!\[CDATA\[)" onclick="JavaScript:ClickRow(\]\]></HTML> 
            <Column Name="ID"/> 
            <HTML><!\[CDATA\[); JavaScript:return false;"><img border="0" width="13" src="/\_layouts/15/images/menudark.gif?rev=23" alt="\]\]></HTML> 
            <HTML>Edit Menu</HTML> 
            <HTML> 
              <!\[CDATA\["></a></td> 
            </tr></table> 
            \]\]> 
            </HTML> 
          </DisplayPattern> 
        </Field> 
        <Field ID="{de1baa4b-2117-473b-aa0c-4d824034142d}" ReadOnly="TRUE" Type="Computed" Group="\_Hidden" Name="RequiredField" DisplayName="Required Field" AuthoringInfo="(must be selected for picture library details view)" Sortable="FALSE" Filterable="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3/fields" StaticName="RequiredField" Customization=""> 
          <FieldRefs> 
            <FieldRef ID="{94F89715-E097-4e8b-BA79-EA02AA8B7ADB}" Name="FileRef"/> 
            <FieldRef ID="{687C7F94-686A-42d3-9B67-2782EAC4B4F8}" Name="FileLeafRef"/> 
            <FieldRef ID="{FA564E0F-0C70-4ab9-B863-0177E6DDD247}" Name="Title"/> 
            <FieldRef ID="{9DA97A8A-1DA5-4a77-98D3-4BC10456E700}" Name="Description"/> 
            <FieldRef ID="{922551B8-C7E0-46a6-B7E3-3CF02917F68A}" Name="ImageSize"/> 
            <FieldRef ID="{081C6E4C-5C14-4f20-B23E-1A71CEB6A67C}" Name="DocIcon"/> 
          </FieldRefs> 
          <DisplayPattern/> 
        </Field> 
        <Field ID="{ac7bb138-02dc-40eb-b07a-84c15575b6e9}" ReadOnly="TRUE" Type="Computed" Group="\_Hidden" Name="Thumbnail" ShowInNewForm="FALSE" ShowInFileDlg="FALSE" ShowInEditForm="FALSE" DisplayName="Thumbnail" Sealed="TRUE" Sortable="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3/fields" StaticName="Thumbnail" Customization=""> 
          <FieldRefs> 
            <FieldRef ID="{7E68A0F9-AF76-404c-9613-6F82BC6DC28C}" Name="ImageWidth"/> 
            <FieldRef ID="{1944C034-D61B-42af-AA84-647F2E74CA70}" Name="ImageHeight"/> 
            <FieldRef ID="{30BB605F-5BAE-48fe-B4E3-1F81D9772AF9}" Name="FSObjType"/> 
            <FieldRef ID="{B9E6F3AE-5632-4b13-B636-9D1A2BD67120}" Name="EncodedAbsThumbnailUrl"/> 
            <FieldRef ID="{1F43CD21-53C5-44c5-8675-B8BB86083244}" Name="ThumbnailExists"/> 
            <FieldRef ID="{F39D44AF-D3F3-4ae6-B43F-AC7330B5E9BD}" Name="AlternateThumbnailUrl"/> 
          </FieldRefs> 
          <DisplayPattern> 
            <IfEqual> 
              <Expr1> 
                <LookupColumn Name="FSObjType"/> 
              </Expr1> 
              <Expr2>0</Expr2> 
              <Then> 
                <IfEqual> 
                  <Expr1> 
                    <LookupColumn Name="ImageWidth"/> 
                  </Expr1> 
                  <Expr2/> 
                  <Then/> 
                  <Else> 
                    <IfEqual> 
                      <Expr1> 
                        <LookupColumn Name="ImageWidth"/> 
                      </Expr1> 
                      <Expr2>0</Expr2> 
                      <Then/> 
                      <Else> 
                        <HTML><!\[CDATA\[<a href="\]\]></HTML> 
                        <URL Cmd="Display"/> 
                        <HTML><!\[CDATA\["><img border="0" ALT="Thumbnail" src="\]\]></HTML> 
                        <Field Name="EncodedAbsThumbnailUrl"/> 
                        <HTML> 
                          <!\[CDATA\["> 
                                    </a> 
                                    \]\]> 
                        </HTML> 
                      </Else> 
                    </IfEqual> 
                  </Else> 
                </IfEqual> 
              </Then> 
            </IfEqual> 
          </DisplayPattern> 
        </Field> 
        <Field ID="{bd716b26-546d-43f2-b229-62699581fa9f}" ReadOnly="TRUE" Type="Computed" Group="\_Hidden" Name="Preview" ShowInNewForm="FALSE" ShowInFileDlg="FALSE" ShowInEditForm="FALSE" DisplayName="Web Preview" Sealed="TRUE" Sortable="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3/fields" StaticName="Preview" Customization=""> 
          <FieldRefs> 
            <FieldRef ID="{7E68A0F9-AF76-404c-9613-6F82BC6DC28C}" Name="ImageWidth"/> 
            <FieldRef ID="{1944C034-D61B-42af-AA84-647F2E74CA70}" Name="ImageHeight"/> 
            <FieldRef ID="{30BB605F-5BAE-48fe-B4E3-1F81D9772AF9}" Name="FSObjType"/> 
            <FieldRef ID="{A1CA0063-779F-49f9-999C-A4A2E3645B07}" Name="EncodedAbsWebImgUrl"/> 
            <FieldRef ID="{1F43CD21-53C5-44c5-8675-B8BB86083244}" Name="PreviewExists"/> 
            <FieldRef ID="{F39D44AF-D3F3-4ae6-B43F-AC7330B5E9BD}" Name="AlternateThumbnailUrl"/> 
          </FieldRefs> 
          <DisplayPattern> 
            <IfEqual> 
              <Expr1> 
                <LookupColumn Name="FSObjType"/> 
              </Expr1> 
              <Expr2>0</Expr2> 
              <Then> 
                <IfEqual> 
                  <Expr1> 
                    <LookupColumn Name="ImageWidth"/> 
                  </Expr1> 
                  <Expr2/> 
                  <Then/> 
                  <Else> 
                    <IfEqual> 
                      <Expr1> 
                        <LookupColumn Name="ImageWidth"/> 
                      </Expr1> 
                      <Expr2>0</Expr2> 
                      <Then/> 
                      <Else> 
                        <HTML><!\[CDATA\[<a href="\]\]></HTML> 
                        <URL Cmd="Display"/> 
                        <HTML><!\[CDATA\["><img  ALT="Web Preview" src="\]\]></HTML> 
                        <Field Name="EncodedAbsWebImgUrl"/> 
                        <HTML> 
                          <!\[CDATA\["> 
                                    </a> 
                                    \]\]> 
                        </HTML> 
                      </Else> 
                    </IfEqual> 
                  </Else> 
                </IfEqual> 
              </Then> 
            </IfEqual> 
          </DisplayPattern> 
        </Field> 
        <Field ID="{086f2b30-460c-4251-b75a-da88a5b205c1}" Type="Text" Group="\_Hidden" Name="ShowCombineView" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ShowCombineView" DisplayName="Show Combine View" Filterable="TRUE" Sortable="TRUE" Hidden="TRUE" FromBaseType="TRUE" Customization="" ColName="nvarchar12" RowOrdinal="0"/> 
        <Field ID="{11851948-b05e-41be-9d9f-bc3bf55d1de3}" Name="ShowRepairView" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ShowRepairView" Type="Text" Group="\_Hidden" DisplayName="Show Repair View" Filterable="TRUE" Sortable="TRUE" Hidden="TRUE" FromBaseType="TRUE" Customization="" ColName="nvarchar13" RowOrdinal="0"/> 
        <Field ID="{1d22ea11-1e32-424e-89ab-9fedbadb6ce1}" ColName="tp\_ID" RowOrdinal="0" ReadOnly="TRUE" Type="Counter" Name="ID" DisplayName="ID" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ID" FromBaseType="TRUE"/> 
        <Field ID="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Type="Computed" DisplayName="Content Type" Name="ContentType" DisplaceOnUpgrade="TRUE" RenderXMLUsingPattern="TRUE" Sortable="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ContentType" Group="\_Hidden" PITarget="MicrosoftWindowsSharePointServices" PIAttribute="ContentTypeID" FromBaseType="TRUE"> 
          <FieldRefs> 
            <FieldRef Name="ContentTypeId"/> 
          </FieldRefs> 
          <DisplayPattern> 
            <MapToContentType> 
              <Column Name="ContentTypeId"/> 
            </MapToContentType> 
          </DisplayPattern> 
        </Field> 
        <Field ID="{8c06beca-0777-48f7-91c7-6da68bc07b69}" ColName="tp\_Created" RowOrdinal="0" ReadOnly="TRUE" Type="DateTime" Name="Created" DisplayName="Created" StorageTZ="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Created" FromBaseType="TRUE"/> 
        <Field ID="{1df5e554-ec7e-46a6-901d-d85a3881cb18}" ColName="tp\_Author" RowOrdinal="0" ReadOnly="TRUE" Type="User" List="UserInfo" Name="Author" DisplayName="Created By" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Author" FromBaseType="TRUE"/> 
        <Field ID="{28cf69c5-fa48-462a-b5cd-27b6f9d2bd5f}" ColName="tp\_Modified" RowOrdinal="0" ReadOnly="TRUE" Type="DateTime" Name="Modified" DisplayName="Modified" StorageTZ="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Modified" FromBaseType="TRUE"/> 
        <Field ID="{d31655d1-1d5b-4511-95a1-7a09e9b75bf2}" ColName="tp\_Editor" RowOrdinal="0" ReadOnly="TRUE" Type="User" List="UserInfo" Name="Editor" DisplayName="Modified By" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Editor" FromBaseType="TRUE"/> 
        <Field ID="{26d0756c-986a-48a7-af35-bf18ab85ff4a}" ColName="tp\_HasCopyDestinations" RowOrdinal="0" Sealed="TRUE" Hidden="TRUE" ReadOnly="TRUE" Type="Boolean" Name="\_HasCopyDestinations" DisplaceOnUpgrade="TRUE" DisplayName="Has Copy Destinations" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_HasCopyDestinations" FromBaseType="TRUE"/> 
        <Field ID="{6b4e226d-3d88-4a36-808d-a129bf52bccf}" ColName="tp\_CopySource" RowOrdinal="0" Sealed="TRUE" Hidden="FALSE" ReadOnly="TRUE" Type="Text" Name="\_CopySource" DisplaceOnUpgrade="TRUE" DisplayName="Copy Source" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_CopySource" FromBaseType="TRUE"/> 
        <Field ID="{fdc3b2ed-5bf2-4835-a4bc-b885f3396a61}" ColName="tp\_ModerationStatus" RowOrdinal="0" ReadOnly="TRUE" Type="ModStat" Name="\_ModerationStatus" DisplayName="Approval Status" Hidden="TRUE" CanToggleHidden="TRUE" Required="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_ModerationStatus" FromBaseType="TRUE"> 
          <CHOICES> 
            <CHOICE>0;#Approved</CHOICE> 
            <CHOICE>1;#Rejected</CHOICE> 
            <CHOICE>2;#Pending</CHOICE> 
            <CHOICE>3;#Draft</CHOICE> 
            <CHOICE>4;#Scheduled</CHOICE> 
          </CHOICES> 
          <Default>0</Default> 
        </Field> 
        <Field ID="{94f89715-e097-4e8b-ba79-ea02aa8b7adb}" Name="FileRef" ReadOnly="TRUE" Hidden="TRUE" Type="Lookup" DisplayName="URL Path" List="Docs" FieldRef="ID" ShowField="FullUrl" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="FileRef" FromBaseType="TRUE"/> 
        <Field ID="{56605df6-8fa1-47e4-a04c-5b384d59609f}" Name="FileDirRef" ReadOnly="TRUE" Hidden="TRUE" Type="Lookup" DisplayName="Path" List="Docs" FieldRef="ID" ShowField="DirName" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="FileDirRef" FromBaseType="TRUE"/> 
        <Field ID="{173f76c8-aebd-446a-9bc9-769a2bd2c18f}" Name="Last\_x0020\_Modified" ReadOnly="TRUE" Hidden="TRUE" DisplayName="Modified" Type="Lookup" List="Docs" FieldRef="ID" ShowField="TimeLastModified" Format="TRUE" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Last\_x0020\_Modified" FromBaseType="TRUE"/> 
        <Field ID="{998b5cff-4a35-47a7-92f3-3914aa6aa4a2}" Name="Created\_x0020\_Date" ReadOnly="TRUE" Hidden="TRUE" DisplayName="Created" Type="Lookup" List="Docs" FieldRef="ID" ShowField="TimeCreated" Format="TRUE" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Created\_x0020\_Date" FromBaseType="TRUE"/> 
        <Field ID="{8fca95c0-9b7d-456f-8dae-b41ee2728b85}" Name="File\_x0020\_Size" Hidden="TRUE" ReadOnly="TRUE" Type="Lookup" DisplayName="File Size" List="Docs" FieldRef="ID" ShowField="SizeInKB" Format="TRUE" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="File\_x0020\_Size" FromBaseType="TRUE"/> 
        <Field ID="{30bb605f-5bae-48fe-b4e3-1f81d9772af9}" Name="FSObjType" ReadOnly="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="Item Type" List="Docs" FieldRef="ID" ShowField="FSType" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="FSObjType" FromBaseType="TRUE"/> 
        <Field ID="{423874f8-c300-4bfb-b7a1-42e2159e3b19}" Name="SortBehavior" ReadOnly="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="Sort Type" List="Docs" FieldRef="ID" ShowField="SortBehavior" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="SortBehavior" FromBaseType="TRUE"/> 
        <Field ID="{ba3c27ee-4791-4867-8821-ff99000bac98}" Name="PermMask" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" RenderXMLUsingPattern="TRUE" ShowInFileDlg="FALSE" Type="Computed" DisplayName="Effective Permissions Mask" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="PermMask" FromBaseType="TRUE"> 
          <FieldRefs> 
            <FieldRef Name="ID"/> 
          </FieldRefs> 
          <DisplayPattern> 
            <CurrentRights/> 
          </DisplayPattern> 
        </Field> 
        <Field ID="{a7b731a3-1df1-4d74-a5c6-e2efba617ae2}" Name="CheckedOutUserId" ReadOnly="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="ID of the User who has the item Checked Out" List="Docs" FieldRef="ID" ShowField="CheckoutUserId" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="CheckedOutUserId" FromBaseType="TRUE"/> 
        <Field ID="{cfaabd0f-bdbd-4bc2-b375-1e779e2cad08}" Name="IsCheckedoutToLocal" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="Is Checked out to local" List="Docs" FieldRef="ID" ShowField="IsCheckoutToLocal" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="IsCheckedoutToLocal" FromBaseType="TRUE"/> 
        <Field ID="{3881510a-4e4a-4ee8-b102-8ee8e2d0dd4b}" ColName="tp\_CheckoutUserId" RowOrdinal="0" ReadOnly="TRUE" Type="User" List="UserInfo" Name="CheckoutUser" DisplaceOnUpgrade="TRUE" DisplayName="Checked Out To" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="CheckoutUser" FromBaseType="TRUE"/> 
        <Field ID="{4b7403de-8d94-43e8-9f0f-137a3e298126}" Name="UniqueId" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="Unique Id" List="Docs" FieldRef="ID" ShowField="UniqueId" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="UniqueId" FromBaseType="TRUE"/> 
        <Field ID="{6d2c4fde-3605-428e-a236-ce5f3dc2b4d4}" Name="SyncClientId" DisplaceOnUpgrade="TRUE" Hidden="TRUE" ReadOnly="TRUE" DisplayName="Client Id" ShowInFileDlg="FALSE" Type="Lookup" List="Docs" FieldRef="ID" ShowField="SyncClientId" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="SyncClientId" FromBaseType="TRUE"/> 
        <Field ID="{c5c4b81c-f1d9-4b43-a6a2-090df32ebb68}" Name="ProgId" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="ProgId" List="Docs" FieldRef="ID" ShowField="ProgId" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ProgId" FromBaseType="TRUE"/> 
        <Field ID="{dddd2420-b270-4735-93b5-92b713d0944d}" Name="ScopeId" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="ScopeId" List="Docs" FieldRef="ID" ShowField="ScopeId" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ScopeId" FromBaseType="TRUE"/> 
        <Field ID="{4a389cb9-54dd-4287-a71a-90ff362028bc}" Name="VirusStatus" Hidden="TRUE" ReadOnly="TRUE" Type="Lookup" DisplayName="Virus Status" List="Docs" FieldRef="ID" ShowField="Size" Format="TRUE" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="VirusStatus" FromBaseType="TRUE"/> 
        <Field ID="{9d4adc35-7cc8-498c-8424-ee5fd541e43a}" Name="CheckedOutTitle" Hidden="TRUE" ReadOnly="TRUE" Type="Lookup" DisplayName="Checked Out To" List="Docs" FieldRef="ID" ShowField="CheckedOutTitle" Format="TRUE" JoinColName="DoclibRowId" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="CheckedOutTitle" FromBaseType="TRUE"/> 
        <Field ID="{58014f77-5463-437b-ab67-eec79532da67}" Name="\_CheckinComment" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Type="Lookup" DisplayName="Check In Comment" List="Docs" FieldRef="ID" ShowField="CheckinComment" Filterable="FALSE" Format="TRUE" JoinColName="DoclibRowId" JoinType="INNER" Sortable="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_CheckinComment" FromBaseType="TRUE"/> 
        <Field ID="{e2a15dfd-6ab8-4aec-91ab-02f6b64045b0}" ReadOnly="TRUE" Hidden="TRUE" Type="Computed" Name="LinkCheckedOutTitle" DisplayName="Checked Out To" Filterable="TRUE" AuthoringInfo="(link to username to user details page)" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="LinkCheckedOutTitle" FromBaseType="TRUE"> 
          <FieldRefs> 
            <FieldRef Name="CheckedOutTitle"/> 
            <FieldRef Name="CheckedOutUserId"/> 
          </FieldRefs> 
          <DisplayPattern> 
            <IfEqual> 
              <Expr1> 
                <Field Name="CheckedOutUserId"/> 
              </Expr1> 
              <Expr2/> 
              <Then/> 
              <Else> 
                <HTML><!\[CDATA\[<a href="\]\]></HTML> 
                <HttpVDir CurrentWeb="TRUE"/> 
                <HTML><!\[CDATA\[/\_layouts/15/userdisp.aspx?id=\]\]></HTML> 
                <Field Name="CheckedOutUserId"/> 
                <HTML><!\[CDATA\[">\]\]></HTML> 
                <Field HTMLEncode="TRUE" Name="CheckedOutTitle"/> 
                <HTML><!\[CDATA\[</a>\]\]></HTML> 
              </Else> 
            </IfEqual> 
          </DisplayPattern> 
        </Field> 
        <Field ID="{3c6303be-e21f-4366-80d7-d6d0a3b22c7a}" Hidden="TRUE" ReadOnly="TRUE" Type="Computed" Name="\_EditMenuTableStart" DisplaceOnUpgrade="TRUE" DisplayName="Edit Menu Table Start" ClassInfo="Menu" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_EditMenuTableStart" FromBaseType="TRUE"> 
          <FieldRefs> 
            <FieldRef Name="FileLeafRef"/> 
            <FieldRef Name="FileDirRef"/> 
            <FieldRef Name="FSObjType"/> 
            <FieldRef Name="ID"/> 
            <FieldRef Name="ServerUrl"/> 
            <FieldRef Name="HTML\_x0020\_File\_x0020\_Type"/> 
            <FieldRef Name="File\_x0020\_Type"/> 
            <FieldRef Name="PermMask"/> 
            <FieldRef Name="IsCheckedoutToLocal"/> 
            <FieldRef Name="CheckoutUser"/> 
            <FieldRef Name="\_SourceUrl"/> 
            <FieldRef Name="\_HasCopyDestinations"/> 
            <FieldRef Name="\_CopySource"/> 
            <FieldRef Name="ContentType"/> 
            <FieldRef Name="ContentTypeId"/> 
            <FieldRef Name="\_ModerationStatus"/> 
            <FieldRef Name="\_UIVersion"/> 
          </FieldRefs> 
          <DisplayPattern> 
            <HTML><!\[CDATA\[
```

                            " id="                            " Url="                            " DRef="                            " Perm="                            " type="                            " Ext="                            " Icon="                                              |                                            " OType="                            " COUId="                            " SRed="                                              |                                            " DefaultIO="                            " COut="                            " HCD="                            " CSrc="                            " MS="                                                                                    " UIS="                                          " SUrl="                            \]\]>                                                                                                              " id="                                                                                                              \]\]></HTML>              <HTML><!\[CDATA\[

\]\]>               \]\]>              \]\]>               \]\]>              \]\]></HTML>            </DisplayPattern>          </Field>          <Field ID="{9d30f126-ba48-446b-b8f9-83745f322ebe}" ReadOnly="TRUE" Type="Computed" Name="LinkFilenameNoMenu" DisplayName="Name" DisplayNameSrcField="FileLeafRef" Filterable="FALSE" AuthoringInfo="(linked to document)" ListItemMenuAllowed="Prohibited" LinkToItemAllowed="Prohibited" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="LinkFilenameNoMenu" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="FileLeafRef"/>              <FieldRef Name="FSObjType"/>              <FieldRef Name="Created\_x0020\_Date"/>              <FieldRef Name="FileRef"/>              <FieldRef Name="File\_x0020\_Type"/>              <FieldRef Name="HTML\_x0020\_File\_x0020\_Type"/>              <FieldRef Name="ContentTypeId"/>              <FieldRef Name="PermMask"/>              <FieldRef Name="CheckoutUser"/>              <FieldRef Name="IsCheckedoutToLocal"/>              <FieldRef Name="ServerUrl"/>            </FieldRefs>            <DisplayPattern>              <IfEqual>                <Expr1>                  <LookupColumn Name="FSObjType"/>                </Expr1>                <Expr2>1</Expr2>                <Then>                  <FieldSwitch>                    <Expr>                      <GetVar Name="RecursiveView"/>                    </Expr>                    <Case Value="1">                      <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/>                    </Case>                    <Default>                      <SetVar Name="UnencodedFilterLink">                        <SetVar Name="RootFolder">                          <HTML>/</HTML>                          <LookupColumn Name="FileRef"/>                        </SetVar>                        <SetVar Name="SkipHost">1</SetVar>                        <SetVar Name="FolderCTID">                          <FieldSwitch>                            <Expr>                              <ListProperty Select="EnableContentTypes"/>                            </Expr>                            <Case Value="1">                              <Column Name="ContentTypeId"/>                            </Case>                          </FieldSwitch>                        </SetVar>                        <FilterLink Default="" Paged="FALSE"/>                      </SetVar>                      <HTML><!\[CDATA\[<a onfocus="OnLink(this)" href="\]\]></HTML>                      <GetVar Name="UnencodedFilterLink" HTMLEncode="TRUE"/>                      <HTML><!\[CDATA\[" onmousedown="javascript:VerifyFolderHref(this,event, '\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <GetVar Name="UnencodedFilterLink"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ServerProperty Select="HtmlTrProgId">                          <Column Name="File\_x0020\_Type"/>                        </ServerProperty>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ListProperty Select="DefaultItemOpen"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <MapToControl>                          <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                          <HTML>|</HTML>                          <Column Name="File\_x0020\_Type"/>                        </MapToControl>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ServerProperty Select="GetServerFileRedirect">                          <Field Name="ServerUrl"/>                          <HTML>|</HTML>                          <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                        </ServerProperty>                      </ScriptQuote>                      <HTML><!\[CDATA\[')"\]\]></HTML>                      <HTML><!\[CDATA\[" onclick="return HandleFolder(this,event, '\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <GetVar Name="UnencodedFilterLink"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ServerProperty Select="HtmlTransform"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ServerProperty Select="HtmlTrAcceptType">                          <Column Name="File\_x0020\_Type"/>                        </ServerProperty>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ServerProperty Select="HtmlTrHandleUrl">                          <Column Name="File\_x0020\_Type"/>                        </ServerProperty>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ServerProperty Select="HtmlTrProgId">                          <Column Name="File\_x0020\_Type"/>                        </ServerProperty>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ListProperty Select="DefaultItemOpen"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <MapToControl>                          <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                          <HTML>|</HTML>                          <Column Name="File\_x0020\_Type"/>                        </MapToControl>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ServerProperty Select="GetServerFileRedirect">                          <Field Name="ServerUrl"/>                          <HTML>|</HTML>                          <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                        </ServerProperty>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <Column Name="CheckoutUser"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <UserID AllowAnonymous="TRUE"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ListProperty Select="ForceCheckout"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <Field Name="IsCheckedoutToLocal"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <Field Name="PermMask"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[');">\]\]></HTML>                      <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/>                      <IfEqual>                        <Expr1>                          <GetVar Name="ShowAccessibleIcon"/>                        </Expr1>                        <Expr2>1</Expr2>                        <Then>                          <HTML><!\[CDATA\[<img src="/\_layouts/15/images/blank.gif?rev=23" class="ms-hidden" border="0" width="1" height="1" alt="Use SHIFT+ENTER to open the menu (new window)."/>\]\]></HTML>                        </Then>                      </IfEqual>                      <HTML><!\[CDATA\[</a>\]\]></HTML>                    </Default>                  </FieldSwitch>                </Then>                <Else>                  <HTML><!\[CDATA\[<a onfocus="OnLink(this)" href="\]\]></HTML>                  <Field Name="ServerUrl" URLEncodeAsURL="TRUE"/>                  <HTML><!\[CDATA\[" onclick="return DispEx(this,event,'\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ServerProperty Select="HtmlTransform"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ServerProperty Select="HtmlTrAcceptType">                      <Column Name="File\_x0020\_Type"/>                    </ServerProperty>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ServerProperty Select="HtmlTrHandleUrl">                      <Column Name="File\_x0020\_Type"/>                    </ServerProperty>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ServerProperty Select="HtmlTrProgId">                      <Column Name="File\_x0020\_Type"/>                    </ServerProperty>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ListProperty Select="DefaultItemOpen"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <MapToControl>                      <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                      <HTML>|</HTML>                      <Column Name="File\_x0020\_Type"/>                    </MapToControl>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ServerProperty Select="GetServerFileRedirect">                      <Field Name="ServerUrl"/>                      <HTML>|</HTML>                      <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                    </ServerProperty>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Column Name="CheckoutUser"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <UserID AllowAnonymous="TRUE"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ListProperty Select="ForceCheckout"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Field Name="IsCheckedoutToLocal"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Field Name="PermMask"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[')">\]\]></HTML>                  <UrlBaseName HTMLEncode="TRUE">                    <LookupColumn Name="FileLeafRef"/>                  </UrlBaseName>                  <IfEqual>                    <Expr1>                      <GetVar Name="ShowAccessibleIcon"/>                    </Expr1>                    <Expr2>1</Expr2>                    <Then>                      <HTML><!\[CDATA\[<img src="/\_layouts/15/images/blank.gif?rev=23" class="ms-hidden" border="0" width="1" height="1" alt="Use SHIFT+ENTER to open the menu (new window)."/>\]\]></HTML>                    </Then>                  </IfEqual>                  <HTML><!\[CDATA\[</a>\]\]></HTML>                  <IfNew Name="Created\_x0020\_Date">                    <HTML><!\[CDATA\[<img src="/\_layouts/15/1033/images/new.gif" alt="\]\]></HTML>                    <HTML>New</HTML>                    <HTML><!\[CDATA\[" class="ms-newgif" />\]\]></HTML>                  </IfNew>                </Else>              </IfEqual>            </DisplayPattern>          </Field>          <Field ID="{5cc6dc79-3710-4374-b433-61cb4a686c12}" ReadOnly="TRUE" Type="Computed" Name="LinkFilename" DisplayName="Name" DisplayNameSrcField="FileLeafRef" Filterable="FALSE" ClassInfo="Menu" AuthoringInfo="(linked to document with edit menu)" ListItemMenuAllowed="Required" LinkToItemAllowed="Prohibited" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="LinkFilename" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="FileLeafRef"/>              <FieldRef Name="LinkFilenameNoMenu"/>              <FieldRef Name="\_EditMenuTableStart2"/>              <FieldRef Name="\_EditMenuTableEnd"/>            </FieldRefs>            <DisplayPattern>              <FieldSwitch>                <Expr>                  <GetVar Name="FreeForm"/>                </Expr>                <Case Value="TRUE">                  <Field Name="LinkFilenameNoMenu"/>                </Case>                <Default>                  <HTML><!\[CDATA\[

                                    \]\]>                                    \]\]></HTML>                  <HTML><!\[CDATA\[

\]\]>                   \]\]>                  \]\]>                   \]\]>                  \]\]></HTML>                </Default>              </FieldSwitch>            </DisplayPattern>          </Field>          <Field ID="{224ba411-da77-4050-b0eb-62d422f13d3e}" Hidden="TRUE" ReadOnly="TRUE" Type="Computed" Name="LinkFilename2" DisplayName="Name" DisplayNameSrcField="FileLeafRef" Filterable="FALSE" ClassInfo="Menu" AuthoringInfo="(linked to document with edit menu) (old)" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="LinkFilename2" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="FileLeafRef"/>              <FieldRef Name="LinkFilenameNoMenu"/>              <FieldRef Name="\_EditMenuTableStart"/>              <FieldRef Name="\_EditMenuTableEnd"/>            </FieldRefs>            <DisplayPattern>              <FieldSwitch>                <Expr>                  <GetVar Name="FreeForm"/>                </Expr>                <Case Value="TRUE">                  <Field Name="LinkFilenameNoMenu"/>                </Case>                <Default>                  <Field Name="\_EditMenuTableStart"/>                  <SetVar Name="ShowAccessibleIcon" Value="1"/>                  <Field Name="LinkFilenameNoMenu"/>                  <SetVar Name="ShowAccessibleIcon" Value="0"/>                  <Field Name="\_EditMenuTableEnd"/>                </Default>              </FieldSwitch>            </DisplayPattern>          </Field>          <Field ID="{081c6e4c-5c14-4f20-b23e-1a71ceb6a67c}" Type="Computed" ReadOnly="TRUE" Name="DocIcon" DisplayName="Type" TextOnly="TRUE" ClassInfo="Icon" AuthoringInfo="(icon linked to document)" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="DocIcon" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="File\_x0020\_Type"/>              <FieldRef Name="FSObjType"/>              <FieldRef Name="FileRef"/>              <FieldRef Name="FileLeafRef"/>              <FieldRef Name="HTML\_x0020\_File\_x0020\_Type"/>              <FieldRef Name="PermMask"/>              <FieldRef Name="CheckoutUser" ShowField="Title"/>              <FieldRef Name="IsCheckedoutToLocal"/>              <FieldRef Name="ServerUrl"/>              <FieldRef Name="IconOverlay"/>            </FieldRefs>            <DisplayPattern>              <SetVar Name="DocIconImg">                <SetVar Name="DocIconAltText">                  <IfEqual>                    <Expr1>                      <LookupColumn Name="FSObjType"/>                    </Expr1>                    <Expr2>1</Expr2>                    <Then>                      <IfSubString>                        <Expr1>0x0120D5</Expr1>                        <Expr2>                          <Column Name="ContentTypeId"/>                        </Expr2>                        <Then>                          <HTML>Document Collection: </HTML>                          <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/>                        </Then>                        <Else>                          <HTML>Folder: </HTML>                          <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/>                        </Else>                      </IfSubString>                    </Then>                    <Else>                      <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/>                    </Else>                  </IfEqual>                </SetVar>                <SetVar Name="DocIconFileName">                  <IfEqual>                    <Expr1>                      <Column Name="IconOverlay"/>                    </Expr1>                    <Expr2/>                    <Then>                      <IfEqual>                        <Expr1>                          <LookupColumn Name="FSObjType"/>                        </Expr1>                        <Expr2>1</Expr2>                        <Then>                          <IfEqual>                            <Expr1>                              <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                              <HTML>|</HTML>                              <Column Name="File\_x0020\_Type"/>                            </Expr1>                            <Expr2>                              <HTML>|</HTML>                            </Expr2>                            <Then>                              <HTML>folder.gif</HTML>                            </Then>                            <Else>                              <SetVar Name="FolderIconFromMap">                                <MapToIcon>                                  <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                                  <HTML>|</HTML>                                  <Column Name="File\_x0020\_Type"/>                                </MapToIcon>                              </SetVar>                              <IfEqual>                                <Expr1>                                  <GetVar Name="FolderIconFromMap"/>                                </Expr1>                                <Expr2>                                  <MapToIcon/>                                </Expr2>                                <Then>                                  <HTML>folder.gif</HTML>                                </Then>                                <Else>                                  <GetVar Name="FolderIconFromMap"/>                                </Else>                              </IfEqual>                            </Else>                          </IfEqual>                        </Then>                        <Else>                          <MapToIcon>                            <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                            <HTML>|</HTML>                            <Column Name="File\_x0020\_Type"/>                          </MapToIcon>                        </Else>                      </IfEqual>                    </Then>                    <Else>                      <MapToIcon>                        <Column Name="IconOverlay"/>                      </MapToIcon>                    </Else>                  </IfEqual>                </SetVar>                <HTML><!\[CDATA\[<img border="0" alt="\]\]></HTML>                <GetVar Name="DocIconAltText"/>                <HTML><!\[CDATA\[" title="\]\]></HTML>                <GetVar Name="DocIconAltText"/>                <HTML><!\[CDATA\[" src="/\_layouts/15/images/\]\]></HTML>                <GetVar Name="DocIconFileName"/>                <HTML><!\[CDATA\[" />\]\]></HTML>              </SetVar>              <SetVar Name="DocIconOverlayImg">                <IfEqual>                  <Expr1>                    <Column Name="IconOverlay"/>                  </Expr1>                  <Expr2/>                  <Then>                    <IfEqual>                      <Expr1>                        <Column Name="CheckoutUser"/>                      </Expr1>                      <Expr2/>                      <Else>                        <SetVar Name="DocIconOverlayAltText">                          <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/>                          <HTML><!\[CDATA\[10;Checked Out To: \]\]></HTML>                          <LookupColumn Name="CheckoutUser" ShowField="Title" HTMLEncode="TRUE"/>                        </SetVar>                        <HTML><!\[CDATA\[<img class="ms-vb-icon-overlay" alt="\]\]></HTML>                        <GetVar Name="DocIconOverlayAltText"/>                        <HTML><!\[CDATA\[" title="\]\]></HTML>                        <GetVar Name="DocIconOverlayAltText"/>                        <HTML><!\[CDATA\[" src="/\_layouts/15/images/checkoutoverlay.gif" />\]\]></HTML>                      </Else>                    </IfEqual>                  </Then>                  <Else>                    <HTML><!\[CDATA\[<img class="ms-vb-icon-overlay" alt="\*" src="/\_layouts/15/images/\]\]></HTML>                    <MapToOverlay>                      <Column Name="IconOverlay"/>                    </MapToOverlay>                    <HTML><!\[CDATA\[" />\]\]></HTML>                  </Else>                </IfEqual>              </SetVar>              <IfEqual>                <Expr1>                  <LookupColumn Name="FSObjType"/>                </Expr1>                <Expr2>1</Expr2>                <Then>                  <FieldSwitch>                    <Expr>                      <GetVar Name="RecursiveView"/>                    </Expr>                    <Case Value="1">                      <GetVar Name="DocIconImg"/>                      <GetVar Name="DocIconOverlayImg"/>                    </Case>                    <Default>                      <SetVar Name="UnencodedFilterLink">                        <SetVar Name="RootFolder">                          <HTML>/</HTML>                          <LookupColumn Name="FileRef"/>                        </SetVar>                        <SetVar Name="SkipHost">1</SetVar>                        <SetVar Name="FolderCTID">                          <FieldSwitch>                            <Expr>                              <ListProperty Select="EnableContentTypes"/>                            </Expr>                            <Case Value="1">                              <Column Name="ContentTypeId"/>                            </Case>                          </FieldSwitch>                        </SetVar>                        <FilterLink Default="" Paged="FALSE"/>                      </SetVar>                      <FieldSwitch>                        <Expr>                          <GetVar Name="FileDialog"/>                        </Expr>                        <Case Value="1">                          <GetVar Name="DocIconImg"/>                          <GetVar Name="DocIconOverlayImg"/>                        </Case>                        <Default>                          <HTML><!\[CDATA\[<a href="\]\]></HTML>                          <GetVar Name="UnencodedFilterLink" HTMLEncode="TRUE"/>                          <HTML><!\[CDATA\[" onmousedown="javascript:VerifyFolderHref(this,event, '\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <GetVar Name="UnencodedFilterLink"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <ServerProperty Select="HtmlTrProgId">                              <Column Name="File\_x0020\_Type"/>                            </ServerProperty>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <ListProperty Select="DefaultItemOpen"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <MapToControl>                              <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                              <HTML>|</HTML>                              <Column Name="File\_x0020\_Type"/>                            </MapToControl>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <ServerProperty Select="GetServerFileRedirect">                              <Field Name="ServerUrl"/>                              <HTML>|</HTML>                              <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                            </ServerProperty>                          </ScriptQuote>                          <HTML><!\[CDATA\[')"\]\]></HTML>                          <HTML><!\[CDATA\[" onclick="return HandleFolder(this,event, '\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <GetVar Name="UnencodedFilterLink"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <ServerProperty Select="HtmlTransform"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <ServerProperty Select="HtmlTrAcceptType">                              <Column Name="File\_x0020\_Type"/>                            </ServerProperty>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <ServerProperty Select="HtmlTrHandleUrl">                              <Column Name="File\_x0020\_Type"/>                            </ServerProperty>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <ServerProperty Select="HtmlTrProgId">                              <Column Name="File\_x0020\_Type"/>                            </ServerProperty>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <ListProperty Select="DefaultItemOpen"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <MapToControl>                              <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                              <HTML>|</HTML>                              <Column Name="File\_x0020\_Type"/>                            </MapToControl>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <ServerProperty Select="GetServerFileRedirect">                              <Field Name="ServerUrl"/>                              <HTML>|</HTML>                              <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                            </ServerProperty>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <Column Name="CheckoutUser"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <UserID AllowAnonymous="TRUE"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <ListProperty Select="ForceCheckout"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <Field Name="IsCheckedoutToLocal"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[','\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <Field Name="PermMask"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[');">\]\]></HTML>                          <GetVar Name="DocIconImg"/>                          <GetVar Name="DocIconOverlayImg"/>                          <HTML><!\[CDATA\[</a>\]\]></HTML>                        </Default>                      </FieldSwitch>                    </Default>                  </FieldSwitch>                </Then>                <Else>                  <HTML><!\[CDATA\[<a href="\]\]></HTML>                  <Field Name="ServerUrl" URLEncodeAsURL="TRUE"/>                  <HTML><!\[CDATA\[" onclick="return DispEx(this,event,'\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ServerProperty Select="HtmlTransform"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ServerProperty Select="HtmlTrAcceptType">                      <Column Name="File\_x0020\_Type"/>                    </ServerProperty>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ServerProperty Select="HtmlTrHandleUrl">                      <Column Name="File\_x0020\_Type"/>                    </ServerProperty>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ServerProperty Select="HtmlTrProgId">                      <Column Name="File\_x0020\_Type"/>                    </ServerProperty>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ListProperty Select="DefaultItemOpen"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <MapToControl>                      <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                      <HTML>|</HTML>                      <Column Name="File\_x0020\_Type"/>                    </MapToControl>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ServerProperty Select="GetServerFileRedirect">                      <Field Name="ServerUrl"/>                      <HTML>|</HTML>                      <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                    </ServerProperty>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Column Name="CheckoutUser"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <UserID AllowAnonymous="TRUE"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ListProperty Select="ForceCheckout"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Field Name="IsCheckedoutToLocal"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Field Name="PermMask"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[')">\]\]></HTML>                  <GetVar Name="DocIconImg"/>                  <GetVar Name="DocIconOverlayImg"/>                  <HTML><!\[CDATA\[</a>\]\]></HTML>                </Else>              </IfEqual>            </DisplayPattern>          </Field>          <Field ID="{105f76ce-724a-4bba-aece-f81f2fce58f5}" ReadOnly="TRUE" Hidden="TRUE" Type="Computed" Name="ServerUrl" DisplayName="Server Relative URL" Filterable="FALSE" RenderXMLUsingPattern="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ServerUrl" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="FileRef"/>            </FieldRefs>            <DisplayPattern>              <HTML>/</HTML>              <LookupColumn Name="FileRef"/>            </DisplayPattern>          </Field>          <Field ID="{7177cfc7-f399-4d4d-905d-37dd51bc90bf}" ReadOnly="TRUE" Hidden="TRUE" Type="Computed" Name="EncodedAbsUrl" DisplayName="Encoded Absolute URL" Filterable="FALSE" RenderXMLUsingPattern="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="EncodedAbsUrl" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="FileRef"/>            </FieldRefs>            <DisplayPattern>              <HttpHost URLEncodeAsURL="TRUE"/>              <HTML>/</HTML>              <LookupColumn Name="FileRef" IncludeVersions="TRUE" URLEncodeAsURL="TRUE"/>            </DisplayPattern>          </Field>          <Field ID="{7615464b-559e-4302-b8e2-8f440b913101}" ReadOnly="TRUE" Hidden="TRUE" Type="Computed" Name="BaseName" DisplayName="Name" Filterable="FALSE" RenderXMLUsingPattern="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="BaseName" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="FileLeafRef"/>              <FieldRef Name="FSObjType"/>            </FieldRefs>            <DisplayPattern>              <IfEqual>                <Expr1>                  <LookupColumn Name="FSObjType"/>                </Expr1>                <Expr2>1</Expr2>                <Then>                  <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/>                </Then>                <Else>                  <UrlBaseName HTMLEncode="TRUE">                    <LookupColumn Name="FileLeafRef"/>                  </UrlBaseName>                </Else>              </IfEqual>            </DisplayPattern>          </Field>          <Field ID="{78a07ba4-bda8-4357-9e0f-580d64487583}" Type="Computed" ReadOnly="TRUE" Name="FileSizeDisplay" DisplayName="File Size" AuthoringInfo="" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="FileSizeDisplay" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="File\_x0020\_Size"/>              <FieldRef Name="FSObjType"/>            </FieldRefs>            <DisplayPattern>              <Switch>                <Expr>                  <LookupColumn Name="FSObjType"/>                </Expr>                <Case Value="0">                  <LookupColumn Name="File\_x0020\_Size"/>                  <HTML> KB</HTML>                </Case>              </Switch>            </DisplayPattern>          </Field>          <Field ID="{687c7f94-686a-42d3-9b67-2782eac4b4f8}" Name="MetaInfo" DisplaceOnUpgrade="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="Property Bag" List="Docs" FieldRef="ID" ShowField="MetaInfo" JoinColName="DoclibRowId" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="MetaInfo" FromBaseType="TRUE"/>          <Field ID="{43bdd51b-3c5b-4e78-90a8-fb2087f71e70}" ColName="tp\_Level" RowOrdinal="0" ReadOnly="TRUE" Type="Integer" Name="\_Level" DisplaceOnUpgrade="TRUE" DisplayName="Level" Hidden="TRUE" Required="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_Level" FromBaseType="TRUE"/>          <Field ID="{c101c3e7-122d-4d4d-bc34-58e94a38c816}" ColName="tp\_IsCurrentVersion" DisplaceOnUpgrade="TRUE" RowOrdinal="0" ReadOnly="TRUE" Type="Boolean" Name="\_IsCurrentVersion" DisplayName="Is Current Version" Hidden="TRUE" Required="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_IsCurrentVersion" FromBaseType="TRUE"/>          <Field ID="{b824e17e-a1b3-426e-aecf-f0184d900485}" Name="ItemChildCount" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="Item Child Count" List="Docs" FieldRef="ID" ShowField="ItemChildCount" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ItemChildCount" FromBaseType="TRUE"/>          <Field ID="{960ff01f-2b6d-4f1b-9c3f-e19ad8927341}" Name="FolderChildCount" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="Folder Child Count" List="Docs" FieldRef="ID" ShowField="FolderChildCount" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="FolderChildCount" FromBaseType="TRUE"/>          <Field ID="{6bfaba20-36bf-44b5-a1b2-eb6346d49716}" ColName="tp\_AppAuthor" RowOrdinal="0" ReadOnly="TRUE" Hidden="FALSE" Type="Lookup" List="AppPrincipals" Name="AppAuthor" DisplayName="App Created By" ShowField="Title" JoinColName="Id" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="AppAuthor" FromBaseType="TRUE"/>          <Field ID="{e08400f3-c779-4ed2-a18c-ab7f34caa318}" ColName="tp\_AppEditor" RowOrdinal="0" ReadOnly="TRUE" Hidden="FALSE" Type="Lookup" List="AppPrincipals" Name="AppEditor" DisplayName="App Modified By" ShowField="Title" JoinColName="Id" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="AppEditor" FromBaseType="TRUE"/>          <Field ID="{b1f7969b-ea65-42e1-8b54-b588292635f2}" ReadOnly="TRUE" Type="Computed" Name="SelectTitle" Filterable="FALSE" Sortable="FALSE" Hidden="TRUE" CanToggleHidden="TRUE" DisplayName="Select" Dir="" AuthoringInfo="(web part connection)" HeaderImage="blank.gif" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="SelectTitle" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="ID"/>            </FieldRefs>            <DisplayPattern>              <IfEqual>                <Expr1>                  <GetVar Name="SelectedID"/>                </Expr1>                <Expr2>                  <Column Name="ID"/>                </Expr2>                <Then>                  <HTML><!\[CDATA\[<img border="0" align="absmiddle" style="cursor: pointer" src="/\_layouts/15/images/rbsel.gif?rev=23" alt="\]\]></HTML>                  <HTML>Selected</HTML>                  <HTML><!\[CDATA\["/>\]\]></HTML>                </Then>                <Else>                  <HTML><!\[CDATA\[<a href="javascript:SelectField('\]\]></HTML>                  <GetVar Name="View"/>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Column Name="ID"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[');return false;" onclick="javascript:SelectField('\]\]></HTML>                  <GetVar Name="View"/>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Column Name="ID"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[');return false;" target="\_self">\]\]></HTML>                  <HTML><!\[CDATA\[<img border="0" align="absmiddle" style="cursor: pointer" src="/\_layouts/15/images/rbunsel.gif?rev=23"  alt="\]\]></HTML>                  <HTML>Normal</HTML>                  <HTML><!\[CDATA\["/>\]\]></HTML>                  <HTML><!\[CDATA\[</a>\]\]></HTML>                </Else>              </IfEqual>            </DisplayPattern>          </Field>          <Field ID="{5f47e085-2150-41dc-b661-442f3027f552}" ReadOnly="TRUE" Type="Computed" Name="SelectFilename" DisplayName="Select" Hidden="TRUE" CanToggleHidden="TRUE" Sortable="FALSE" Filterable="FALSE" AuthoringInfo="(web part connection)" HeaderImage="blank.gif" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="SelectFilename" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="ID"/>            </FieldRefs>            <DisplayPattern>              <IfEqual>                <Expr1>                  <GetVar Name="SelectedID"/>                </Expr1>                <Expr2>                  <Column Name="ID"/>                </Expr2>                <Then>                  <HTML><!\[CDATA\[<img align="absmiddle" style="cursor: pointer" src="/\_layouts/15/images/rbsel.gif?rev=23" alt="\]\]></HTML>                  <HTML>Selected</HTML>                  <HTML><!\[CDATA\["/>\]\]></HTML>                </Then>                <Else>                  <HTML><!\[CDATA\[<a href="javascript:SelectField('\]\]></HTML>                  <GetVar Name="View"/>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Column Name="ID"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[');return false;" onclick="javascript:SelectField('\]\]></HTML>                  <GetVar Name="View"/>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Column Name="ID"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[');return false;" target="\_self">\]\]></HTML>                  <HTML><!\[CDATA\[<img border="0" align="absmiddle" style="cursor: pointer" src="/\_layouts/15/images/rbunsel.gif?rev=23"  alt="\]\]></HTML>                  <HTML>Normal</HTML>                  <HTML><!\[CDATA\["/>\]\]></HTML>                  <HTML><!\[CDATA\[</a>\]\]></HTML>                </Else>              </IfEqual>            </DisplayPattern>          </Field>          <Field ID="{503f1caa-358e-4918-9094-4a2cdc4bc034}" ReadOnly="TRUE" Type="Computed" Name="Edit" Sortable="FALSE" Filterable="FALSE" DisplayName="Edit" ClassInfo="Icon" AuthoringInfo="(link to edit item)" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Edit" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="IsCheckedoutToLocal"/>              <FieldRef Name="ServerUrl"/>              <FieldRef Name="CheckedOutUserId"/>            </FieldRefs>            <DisplayPattern>              <IfHasRights>                <RightsChoices>                  <RightsGroup PermEditListItems="required"/>                </RightsChoices>                <Then>                  <HTML><!\[CDATA\[<a href="\]\]></HTML>                  <URL Cmd="Edit"/>                  <HTML><!\[CDATA\[" onclick="EditItemWithCheckoutAlert(event, this.href, '\]\]></HTML>                  <IfEqual>                    <Expr1>                      <ListProperty Select="ForceCheckout"/>                    </Expr1>                    <Expr2>1</Expr2>                    <Then>                      <IfEqual>                        <Expr1>                          <Field Name="CheckedOutUserId"/>                        </Expr1>                        <Expr2/>                        <Then>                          <IfEqual>                            <Expr1>                              <LookupColumn Name="FSObjType"/>                            </Expr1>                            <Expr2>1</Expr2>                            <Then>                              <HTML><!\[CDATA\[0\]\]></HTML>                            </Then>                            <Else>                              <HTML><!\[CDATA\[1\]\]></HTML>                            </Else>                          </IfEqual>                        </Then>                        <Else>                          <HTML><!\[CDATA\[0\]\]></HTML>                        </Else>                      </IfEqual>                    </Then>                    <Else>                      <HTML><!\[CDATA\[0\]\]></HTML>                    </Else>                  </IfEqual>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Field Name="IsCheckedoutToLocal"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Field Name="ServerUrl"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <HttpVDir/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Field Name="CheckedOutUserId"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <UserID AllowAnonymous="TRUE"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[') ;return false;" target="\_self">\]\]></HTML>                  <HTML><!\[CDATA\[<img border="0" alt="\]\]></HTML>                  <HTML>Edit Document Properties</HTML>                  <HTML><!\[CDATA\[" src="/\_layouts/images/edititem.gif"/>\]\]></HTML>                  <HTML><!\[CDATA\[</a>\]\]></HTML>                </Then>                <Else>                  <HTML><!\[CDATA\[160;\]\]></HTML>                </Else>              </IfHasRights>            </DisplayPattern>          </Field>          <Field ID="{d4e44a66-ee3a-4d02-88c9-4ec5ff3f4cd5}" ColName="tp\_Version" RowOrdinal="0" Hidden="TRUE" ReadOnly="TRUE" Type="Integer" SetAs="owshiddenversion" Name="owshiddenversion" DisplayName="owshiddenversion" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="owshiddenversion" FromBaseType="TRUE"/>          <Field ID="{7841bf41-43d0-4434-9f50-a673baef7631}" ColName="tp\_UIVersion" RowOrdinal="0" ReadOnly="TRUE" Type="Integer" Name="\_UIVersion" DisplaceOnUpgrade="TRUE" DisplayName="UI Version" Hidden="TRUE" CanToggleHidden="TRUE" Required="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_UIVersion" FromBaseType="TRUE"/>          <Field ID="{dce8262a-3ae9-45aa-aab4-83bd75fb738a}" ColName="tp\_UIVersionString" RowOrdinal="0" ReadOnly="TRUE" Type="Text" Name="\_UIVersionString" DisplaceOnUpgrade="TRUE" DisplayName="Version" CanToggleHidden="TRUE" Required="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_UIVersionString" FromBaseType="TRUE"/>          <Field ID="{50a54da4-1528-4e67-954a-e2d24f1e9efb}" Name="InstanceID" DisplayName="Instance ID" ColName="tp\_InstanceID" RowOrdinal="0" ReadOnly="TRUE" Hidden="TRUE" Type="Integer" Min="0" Max="99991231" Filterable="TRUE" Sortable="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="InstanceID" FromBaseType="TRUE"/>          <Field ID="{ca4addac-796f-4b23-b093-d2a3f65c0774}" ColName="tp\_ItemOrder" RowOrdinal="0" Name="Order" DisplayName="Order" Type="Number" Hidden="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Order" FromBaseType="TRUE"/>          <Field ID="{ae069f25-3ac2-4256-b9c3-15dbc15da0e0}" ColName="tp\_GUID" RowOrdinal="0" ReadOnly="TRUE" Hidden="TRUE" Type="Guid" Name="GUID" DisplaceOnUpgrade="TRUE" DisplayName="GUID" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="GUID" FromBaseType="TRUE"/>          <Field ID="{f1e020bc-ba26-443f-bf2f-b68715017bbc}" ColName="tp\_WorkflowVersion" RowOrdinal="0" Hidden="TRUE" ReadOnly="TRUE" Type="Integer" Name="WorkflowVersion" DisplaceOnUpgrade="TRUE" DisplayName="Workflow Version" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="WorkflowVersion" FromBaseType="TRUE"/>          <Field ID="{de8beacf-5505-47cd-80a6-aa44e7ffe2f4}" ColName="tp\_WorkflowInstanceID" RowOrdinal="0" ReadOnly="TRUE" Hidden="TRUE" Type="Guid" Name="WorkflowInstanceID" DisplaceOnUpgrade="TRUE" DisplayName="Workflow Instance ID" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="WorkflowInstanceID" FromBaseType="TRUE"/>          <Field ID="{bc1a8efb-0f4c-49f8-a38f-7fe22af3d3e0}" Name="ParentVersionString" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" DisplayName="Source Version (Converted Document)" ShowInFileDlg="FALSE" Type="Lookup" List="Docs" FieldRef="ID" ShowField="ParentVersionString" JoinColName="DoclibRowId" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ParentVersionString" FromBaseType="TRUE"/>          <Field ID="{774eab3a-855f-4a34-99da-69dc21043bec}" Name="ParentLeafName" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" DisplayName="Source Name (Converted Document)" ShowInFileDlg="FALSE" Type="Lookup" List="Docs" FieldRef="ID" ShowField="ParentLeafName" JoinColName="DoclibRowId" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ParentLeafName" FromBaseType="TRUE"/>          <Field ID="{8e69e8e8-df8a-45dc-858a-1b806dde24c0}" Name="DocConcurrencyNumber" DisplaceOnUpgrade="TRUE" Hidden="TRUE" ReadOnly="TRUE" DisplayName="Document Concurrency Number" ShowInFileDlg="FALSE" Type="Lookup" List="Docs" FieldRef="ID" ShowField="DocConcurrencyNumber" JoinColName="DoclibRowId" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="DocConcurrencyNumber" FromBaseType="TRUE"/>          <Field ID="{e52012a0-51eb-4c0c-8dfb-9b8a0ebedcb6}" ReadOnly="TRUE" Type="Computed" Name="Combine" DisplaceOnUpgrade="TRUE" DisplayName="Merge" Filterable="FALSE" Sortable="FALSE" Hidden="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Combine">            <FieldRefs>              <FieldRef Name="FSObjType" Key="Primary"/>              <FieldRef Name="EncodedAbsUrl"/>              <FieldRef Name="TemplateUrl"/>            </FieldRefs>            <DisplayPattern>              <IfEqual>                <Expr1>                  <Field Name="FSObjType"/>                </Expr1>                <Expr2>0</Expr2>                <Then>                  <HTML><!\[CDATA\[<input id="chkCombine" type="CHECKBOX" title="Merge\]\]" href="\]\]></HTML>                  <Field Name="EncodedAbsUrl"/>                  <HTML><!\[CDATA\[">\]\]></HTML>                  <HTML><!\[CDATA\[<input id="chkUrl" type="HIDDEN" href="\]\]></HTML>                  <Column Name="TemplateUrl" HTMLEncode="TRUE"/>                  <HTML><!\[CDATA\[">\]\]></HTML>                  <HTML><!\[CDATA\[<input id="chkProgID" type="HIDDEN" href="\]\]></HTML>                  <MapToControl>                    <HTML>|</HTML>                    <GetFileExtension>                      <Column Name="TemplateUrl" HTMLEncode="TRUE"/>                    </GetFileExtension>                  </MapToControl>                  <HTML><!\[CDATA\[">\]\]></HTML>                </Then>              </IfEqual>            </DisplayPattern>          </Field>          <Field ID="{5d36727b-bcb2-47d2-a231-1f0bc63b7439}" ReadOnly="TRUE" Type="Computed" Name="RepairDocument" DisplaceOnUpgrade="TRUE" DisplayName="Relink" Filterable="FALSE" Sortable="FALSE" Hidden="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="RepairDocument">            <FieldRefs>              <FieldRef Name="FSObjType" Key="Primary"/>              <FieldRef Name="ID"/>            </FieldRefs>            <DisplayPattern>              <IfEqual>                <Expr1>                  <Field Name="FSObjType"/>                </Expr1>                <Expr2>0</Expr2>                <Then>                  <HTML><!\[CDATA\[<input id="chkRepair" type="CHECKBOX" title="Relink" docid="\]\]></HTML>                  <Field Name="ID"/>                  <HTML><!\[CDATA\[">\]\]></HTML>                </Then>              </IfEqual>            </DisplayPattern>          </Field>        </Fields>        <ContentTypes>          <ContentType ID="0x0101005841B6968E02404B96D5E429EDA1FA59" Name="Document" Group="Document Content Types" Description="Create a new document." Version="0" FeatureId="{695b6570-a48b-4a8e-8ea5-26ea7fc1d162}">            <Folder TargetName="Forms/Document"/>            <FieldRefs>              <FieldRef ID="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Name="ContentType"/>              <FieldRef ID="{5f47e085-2150-41dc-b661-442f3027f552}" Name="SelectFilename"/>              <FieldRef ID="{8553196d-ec8d-4564-9861-3dbe931050c8}" Name="FileLeafRef" Required="TRUE"/>              <FieldRef ID="{8c06beca-0777-48f7-91c7-6da68bc07b69}" Name="Created" Hidden="TRUE"/>              <FieldRef ID="{fa564e0f-0c70-4ab9-b863-0177e6ddd247}" Name="Title" Required="FALSE" ShowInNewForm="FALSE" ShowInEditForm="TRUE"/>              <FieldRef ID="{28cf69c5-fa48-462a-b5cd-27b6f9d2bd5f}" Name="Modified" Hidden="TRUE"/>              <FieldRef ID="{822c78e3-1ea9-4943-b449-57863ad33ca9}" Name="Modified\_x0020\_By" Hidden="FALSE"/>              <FieldRef ID="{4dd7e525-8d6b-4cb4-9d3e-44ee25f973eb}" Name="Created\_x0020\_By" Hidden="FALSE"/>            </FieldRefs>            <XmlDocuments>              <XmlDocument NamespaceURI="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms">PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+RG9jdW1lbnRMaWJyYXJ5Rm9ybTwvRGlzcGxheT48RWRpdD5Eb2N1bWVudExpYnJhcnlGb3JtPC9FZGl0PjxOZXc+RG9jdW1lbnRMaWJyYXJ5Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4=</XmlDocument>            </XmlDocuments>          </ContentType>          <ContentType ID="0x012000F883BB94D64DA645A4434A87E0A7F82E" Name="Folder" Group="Folder Content Types" Description="Create a new folder." Sealed="TRUE" Version="0" FeatureId="{695b6570-a48b-4a8e-8ea5-26ea7fc1d162}">            <FieldRefs>              <FieldRef ID="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Name="ContentType"/>              <FieldRef ID="{fa564e0f-0c70-4ab9-b863-0177e6ddd247}" Name="Title" Required="FALSE" Hidden="TRUE"/>              <FieldRef ID="{8553196d-ec8d-4564-9861-3dbe931050c8}" Name="FileLeafRef" Required="TRUE" Hidden="FALSE"/>              <FieldRef ID="{b824e17e-a1b3-426e-aecf-f0184d900485}" Name="ItemChildCount"/>              <FieldRef ID="{960ff01f-2b6d-4f1b-9c3f-e19ad8927341}" Name="FolderChildCount"/>            </FieldRefs>            <XmlDocuments>              <XmlDocument NamespaceURI="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms">PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+TGlzdEZvcm08L0Rpc3BsYXk+PEVkaXQ+TGlzdEZvcm08L0VkaXQ+PE5ldz5MaXN0Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4=</XmlDocument>            </XmlDocuments>          </ContentType>          <ContentType ID="0x01010900E9216EC6C932CA40BD6E7943876C456A" Name="Basic Page" Group="Document Content Types" Description="Create a new basic page." Version="0" FeatureId="{695b6570-a48b-4a8e-8ea5-26ea7fc1d162}">            <Folder TargetName="Forms/Basic Page"/>            <FieldRefs>              <FieldRef ID="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Name="ContentType"/>              <FieldRef ID="{5f47e085-2150-41dc-b661-442f3027f552}" Name="SelectFilename"/>              <FieldRef ID="{8553196d-ec8d-4564-9861-3dbe931050c8}" Name="FileLeafRef" Required="TRUE"/>              <FieldRef ID="{8c06beca-0777-48f7-91c7-6da68bc07b69}" Name="Created" Hidden="TRUE"/>              <FieldRef ID="{fa564e0f-0c70-4ab9-b863-0177e6ddd247}" Name="Title" Required="FALSE" ShowInNewForm="FALSE" ShowInEditForm="TRUE"/>              <FieldRef ID="{28cf69c5-fa48-462a-b5cd-27b6f9d2bd5f}" Name="Modified" Hidden="TRUE"/>              <FieldRef ID="{822c78e3-1ea9-4943-b449-57863ad33ca9}" Name="Modified\_x0020\_By" Hidden="FALSE"/>              <FieldRef ID="{4dd7e525-8d6b-4cb4-9d3e-44ee25f973eb}" Name="Created\_x0020\_By" Hidden="FALSE"/>            </FieldRefs>            <DocumentTemplate TargetName="/\_layouts/15/bpcf.aspx"/>            <XmlDocuments>              <XmlDocument NamespaceURI="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms">PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+RG9jdW1lbnRMaWJyYXJ5Rm9ybTwvRGlzcGxheT48RWRpdD5Eb2N1bWVudExpYnJhcnlGb3JtPC9FZGl0PjxOZXc+RG9jdW1lbnRMaWJyYXJ5Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4=</XmlDocument>            </XmlDocuments>          </ContentType>          <ContentType ID="0x01010200D5963C4B37B9CF43A3BA30F1653E9C90" Name="Picture" Group="Document Content Types" Description="Upload an image or a photograph." Version="0" FeatureId="{695b6570-a48b-4a8e-8ea5-26ea7fc1d162}">            <Folder TargetName="Forms/Picture"/>            <FieldRefs>              <FieldRef ID="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Name="ContentType"/>              <FieldRef ID="{5f47e085-2150-41dc-b661-442f3027f552}" Name="SelectFilename"/>              <FieldRef ID="{8553196d-ec8d-4564-9861-3dbe931050c8}" Name="FileLeafRef" Required="TRUE"/>              <FieldRef ID="{8c06beca-0777-48f7-91c7-6da68bc07b69}" Name="Created" Hidden="TRUE"/>              <FieldRef ID="{28cf69c5-fa48-462a-b5cd-27b6f9d2bd5f}" Name="Modified" Hidden="TRUE"/>              <FieldRef ID="{822c78e3-1ea9-4943-b449-57863ad33ca9}" Name="Modified\_x0020\_By" Hidden="FALSE"/>              <FieldRef ID="{4dd7e525-8d6b-4cb4-9d3e-44ee25f973eb}" Name="Created\_x0020\_By" Hidden="FALSE"/>              <FieldRef ID="{8c0d0aac-9b76-4951-927a-2490abe13c0b}" Name="PreviewOnForm"/>              <FieldRef ID="{fa564e0f-0c70-4ab9-b863-0177e6ddd247}" Name="Title" ShowInNewForm="FALSE" ShowInFileDlg="FALSE"/>              <FieldRef ID="{c53a03f3-f930-4ef2-b166-e0f2210c13c0}" Name="FileType"/>              <FieldRef ID="{922551b8-c7e0-46a6-b7e3-3cf02917f68a}" Name="ImageSize"/>              <FieldRef ID="{7e68a0f9-af76-404c-9613-6f82bc6dc28c}" Name="ImageWidth"/>              <FieldRef ID="{1944c034-d61b-42af-aa84-647f2e74ca70}" Name="ImageHeight"/>              <FieldRef ID="{a5d2f824-bc53-422e-87fd-765939d863a5}" Name="ImageCreateDate"/>              <FieldRef ID="{9da97a8a-1da5-4a77-98d3-4bc10456e700}" Name="Comments" ShowInNewForm="FALSE" ShowInFileDlg="FALSE"/>              <FieldRef ID="{b9e6f3ae-5632-4b13-b636-9d1a2bd67120}" Name="EncodedAbsThumbnailUrl"/>              <FieldRef ID="{a1ca0063-779f-49f9-999c-a4a2e3645b07}" Name="EncodedAbsWebImgUrl"/>              <FieldRef ID="{7ebf72ca-a307-4c18-9e5b-9d89e1dae74f}" Name="SelectedFlag"/>              <FieldRef ID="{76d1cc87-56de-432c-8a2a-16e5ba5331b3}" Name="NameOrTitle"/>              <FieldRef ID="{de1baa4b-2117-473b-aa0c-4d824034142d}" Name="RequiredField"/>              <FieldRef ID="{b66e9b50-a28e-469b-b1a0-af0e45486874}" Name="Keywords" ShowInNewForm="FALSE" ShowInFileDlg="FALSE"/>              <FieldRef ID="{ac7bb138-02dc-40eb-b07a-84c15575b6e9}" Name="Thumbnail"/>              <FieldRef ID="{bd716b26-546d-43f2-b229-62699581fa9f}" Name="Preview"/>              <FieldRef ID="{1f43cd21-53c5-44c5-8675-b8bb86083244}" Name="ThumbnailExists"/>              <FieldRef ID="{3ca8efcd-96e8-414f-ba90-4c8c4a8bfef8}" Name="PreviewExists"/>              <FieldRef ID="{f39d44af-d3f3-4ae6-b43f-ac7330b5e9bd}" Name="AlternateThumbnailUrl"/>            </FieldRefs>            <DocumentTemplate TargetName="/\_layouts/15/upload.aspx"/>            <XmlDocuments>              <XmlDocument NamespaceURI="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms">PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+RG9jdW1lbnRMaWJyYXJ5Rm9ybTwvRGlzcGxheT48RWRpdD5Eb2N1bWVudExpYnJhcnlGb3JtPC9FZGl0PjxOZXc+RG9jdW1lbnRMaWJyYXJ5Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4=</XmlDocument>            </XmlDocuments>          </ContentType>          <ContentType ID="0x01010100D50E45BD774B0D45BA56CB41A920A7D7" Name="Form" Group="Document Content Types" Description="Fill out this form." Version="1" FeatureId="{695b6570-a48b-4a8e-8ea5-26ea7fc1d162}">            <Folder TargetName="Forms/Form"/>            <FieldRefs>              <FieldRef ID="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Name="ContentType"/>              <FieldRef ID="{5f47e085-2150-41dc-b661-442f3027f552}" Name="SelectFilename"/>              <FieldRef ID="{8553196d-ec8d-4564-9861-3dbe931050c8}" Name="FileLeafRef" Required="TRUE"/>              <FieldRef ID="{8c06beca-0777-48f7-91c7-6da68bc07b69}" Name="Created" Hidden="TRUE"/>              <FieldRef ID="{28cf69c5-fa48-462a-b5cd-27b6f9d2bd5f}" Name="Modified" Hidden="TRUE"/>              <FieldRef ID="{822c78e3-1ea9-4943-b449-57863ad33ca9}" Name="Modified\_x0020\_By" Hidden="FALSE"/>              <FieldRef ID="{4dd7e525-8d6b-4cb4-9d3e-44ee25f973eb}" Name="Created\_x0020\_By" Hidden="FALSE"/>              <FieldRef ID="{e52012a0-51eb-4c0c-8dfb-9b8a0ebedcb6}" Name="Combine"/>              <FieldRef ID="{086f2b30-460c-4251-b75a-da88a5b205c1}" Name="ShowCombineView"/>              <FieldRef ID="{5d36727b-bcb2-47d2-a231-1f0bc63b7439}" Name="RepairDocument"/>              <FieldRef ID="{11851948-b05e-41be-9d9f-bc3bf55d1de3}" Name="ShowRepairView"/>              <FieldRef ID="{4b1bf6c6-4f39-45ac-acd5-16fe7a214e5e}" Name="TemplateUrl"/>              <FieldRef ID="{cd1ecb9f-dd4e-4f29-ab9e-e9ff40048d64}" Name="xd\_ProgID"/>            </FieldRefs>            <DocumentTemplate TargetName="Forms/Form/template.dotx"/>            <XmlDocuments>              <XmlDocument NamespaceURI="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms">PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+RG9jdW1lbnRMaWJyYXJ5Rm9ybTwvRGlzcGxheT48RWRpdD5Eb2N1bWVudExpYnJhcnlGb3JtPC9FZGl0PjxOZXc+RG9jdW1lbnRMaWJyYXJ5Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4=</XmlDocument>            </XmlDocuments>          </ContentType>        </ContentTypes>        <Forms>          <Form Type="DisplayForm" Name="{F62DF496-96AC-47EC-923B-C85A77D77790}" Url="Test Document Library/Forms/DispForm.aspx" Default="TRUE" FormID="0"/>          <Form Type="EditForm" Name="{833EB309-9AE9-49C3-A9C3-E4CE9E6F81B5}" Url="Test Document Library/Forms/EditForm.aspx" Default="TRUE" FormID="0"/>          <Form Type="NewForm" Name="{72B72D78-E83D-4652-B723-0ECB85F12097}" Url="Test Document Library/Forms/Upload.aspx" Default="TRUE" FormID="0"/>        </Forms>        <Security>          <ReadSecurity>1</ReadSecurity>          <WriteSecurity>1</WriteSecurity>          <SchemaSecurity>0</SchemaSecurity>        </Security>        <DocumentLibraryTemplate>Test Document Library/Forms/template.dotx</DocumentLibraryTemplate>        <Receivers>          <Receiver>            <Name>Content Following Item Updated Event Receiver 101</Name>            <Synchronization>2</Synchronization>            <Type>10002</Type>            <SequenceNumber>10000</SequenceNumber>            <Assembly>Microsoft.Office.Server.UserProfiles, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c</Assembly>            <Class>Microsoft.Office.Server.UserProfiles.ContentFollowingItemEventReceiver</Class>            <Data></Data>          </Receiver>        </Receivers>      </MetaData>    </List>    <List Name="{592B2A0B-45C5-4A2D-9F05-04444E50150E}" Title="Test1WithMultipleContentTypes" Description="" Direction="0" BaseType="0" FeatureId="{00BFEA71-DE22-43B2-A848-C05709900100}" ServerTemplate="100" Url="Lists/Test1WithMultipleContentTypes" FolderCreation="FALSE" EnableContentTypes="TRUE" NavigateForFormsPages="TRUE" BrowserFileHandling="permissive" Version="14">      <MetaData>        <Views>          <View Name="{C09088E3-0D25-4B40-9AA2-33033143CC0C}" DefaultView="TRUE" MobileView="TRUE" MobileDefaultView="TRUE" Type="HTML" DisplayName="All Items" Url="Lists/Test1WithMultipleContentTypes/AllItems.aspx" Level="1" BaseViewID="1" ContentTypeID="0x" ImageUrl="/\_layouts/15/images/generic.png?rev=23">            <XslLink Default="TRUE">main.xsl</XslLink>            <JSLink>clienttemplates.js</JSLink>            <RowLimit Paged="TRUE">30</RowLimit>            <Toolbar Type="Standard"/>            <ViewFields>              <FieldRef Name="LinkTitle"/>            </ViewFields>            <Query>              <OrderBy>                <FieldRef Name="ID"/>              </OrderBy>            </Query>            <ParameterBindings>              <ParameterBinding Name="NoAnnouncements" Location="Resource(wss,noXinviewofY\_LIST)"/>              <ParameterBinding Name="NoAnnouncementsHowTo" Location="Resource(wss,noXinviewofY\_DEFAULT)"/>            </ParameterBindings>          </View>          <View Name="{58A38F2B-BCFC-44F6-BD1C-5E44E014AFAF}" MobileView="TRUE" Type="HTML" DisplayName="testview" Url="Lists/Test1WithMultipleContentTypes/testview.aspx" Level="1" BaseViewID="1" ContentTypeID="0x" ImageUrl="/\_layouts/15/images/generic.png?rev=23">            <ViewFields>              <FieldRef Name="EDRMSAction"/>              <FieldRef Name="EDRMSActualCloseddate"/>              <FieldRef Name="EDRMSAuditContact"/>              <FieldRef Name="EDRMSD\_TSpendValuerequested"/>              <FieldRef Name="ItemChildCount"/>            </ViewFields>            <ViewData/>            <Query/>            <Aggregations Value="Off"/>            <RowLimit Paged="TRUE">30</RowLimit>            <Mobile MobileItemLimit="3" MobileSimpleViewField="EDRMSAction"/>            <XslLink Default="TRUE">main.xsl</XslLink>            <JSLink>clienttemplates.js</JSLink>            <Toolbar Type="Standard"/>            <ParameterBindings>              <ParameterBinding Name="NoAnnouncements" Location="Resource(wss,noXinviewofY\_LIST)"/>              <ParameterBinding Name="NoAnnouncementsHowTo" Location="Resource(wss,noXinviewofY\_DEFAULT)"/>            </ParameterBindings>          </View>          <View Name="{7D342A66-6550-413A-BAD0-B63FFE9C2E44}" MobileView="TRUE" Type="CALENDAR" Scope="Recursive" DisplayName="calendar test" Url="Lists/Test1WithMultipleContentTypes/calendar test.aspx" Level="1" BaseViewID="1" ContentTypeID="0x" ImageUrl="/\_layouts/15/images/generic.png?rev=23">            <ViewFields>              <FieldRef Name="EDRMSActualCloseddate"/>              <FieldRef Name="EDRMSActualCloseddate"/>              <FieldRef Name="Title"/>            </ViewFields>            <CalendarViewStyles>&lt;CalendarViewStyle  Title=39;Day39; Type=39;day39; Template=39;CalendarViewdayChrome39; Sequence=39;139; Default=39;FALSE39; /&gt;&lt;CalendarViewStyle  Title=39;Week39; Type=39;week39; Template=39;CalendarViewweekChrome39; Sequence=39;239; Default=39;FALSE39; /&gt;&lt;CalendarViewStyle  Title=39;Month39; Type=39;month39; Template=39;CalendarViewmonthChrome39; Sequence=39;339; Default=39;TRUE39; /&gt;</CalendarViewStyles>            <ViewData>              <FieldRef Name="EDRMSAction" Type="CalendarMonthTitle"/>              <FieldRef Name="EDRMSCurrentstatus" Type="CalendarWeekTitle"/>              <FieldRef Name="EDRMSD\_TSpendValuerequested" Type="CalendarWeekLocation"/>              <FieldRef Name="EDRMSCurrentstatus" Type="CalendarDayTitle"/>              <FieldRef Name="EDRMSD\_TSpendValuerequested" Type="CalendarDayLocation"/>            </ViewData>            <Query>              <Where>                <And>                  <DateRangesOverlap>                    <FieldRef Name="EDRMSActualCloseddate"/>                    <FieldRef Name="EDRMSActualCloseddate"/>                    <Value Type="DateTime">                      <Month/>                    </Value>                  </DateRangesOverlap>                  <Eq>                    <FieldRef Name="EDRMSCurrentstatus"/>                    <Value Type="Text">Active</Value>                  </Eq>                </And>              </Where>            </Query>            <Aggregations Value="Off"/>            <RowLimit>0</RowLimit>            <Mobile MobileItemLimit="3"/>            <XslLink Default="TRUE">main.xsl</XslLink>            <JSLink>clienttemplates.js</JSLink>            <Toolbar Type="Standard"/>            <ParameterBindings>              <ParameterBinding Name="NoAnnouncements" Location="Resource(wss,noXinviewofY\_LIST)"/>              <ParameterBinding Name="NoAnnouncementsHowTo" Location="Resource(wss,noXinviewofY\_DEFAULT)"/>            </ParameterBindings>          </View>        </Views>        <Fields>          <Field ID="{03e45e84-1992-4d42-9116-26f756012634}" RowOrdinal="0" Type="ContentTypeId" Sealed="TRUE" ReadOnly="TRUE" Hidden="TRUE" DisplayName="Content Type ID" Name="ContentTypeId" DisplaceOnUpgrade="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ContentTypeId" ColName="tp\_ContentTypeId" FromBaseType="TRUE"/>          <Field ID="{fa564e0f-0c70-4ab9-b863-0177e6ddd247}" Type="Text" Name="Title" DisplayName="Title" Required="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Title" FromBaseType="TRUE" ColName="nvarchar1"/>          <Field ID="{34ad21eb-75bd-4544-8c73-0e08330291fe}" ReadOnly="TRUE" Type="Note" Name="\_ModerationComments" DisplayName="Approver Comments" Hidden="TRUE" CanToggleHidden="TRUE" Filterable="FALSE" Sortable="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_ModerationComments" FromBaseType="TRUE" ColName="ntext1"/>          <Field ID="{39360f11-34cf-4356-9945-25c44e68dade}" ReadOnly="TRUE" Hidden="TRUE" Type="Text" Name="File\_x0020\_Type" DisplaceOnUpgrade="TRUE" DisplayName="File Type" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="File\_x0020\_Type" FromBaseType="TRUE" ColName="nvarchar2"/>          <Field Type="Note" Name="EDRMSAction" ID="{f4f3e946-d979-4ff5-8223-c8f01ebc3347}" Description="" DisplayName="Action" StaticName="EDRMSAction" Group="EDRMS" Hidden="FALSE" Required="FALSE" Sealed="FALSE" ShowInDisplayForm="TRUE" ShowInEditForm="TRUE" ShowInListSettings="TRUE" ShowInNewForm="TRUE" SourceID="{f52ffc71-99ff-42c2-ad9c-88f88574918c}" AllowDeletion="TRUE" Customization="" ColName="ntext2" RowOrdinal="0"/>          <Field Type="DateTime" Name="EDRMSActualCloseddate" ID="{e67580c2-9226-40bf-8f34-5973dba37397}" Description="" DisplayName="Actual Closed date" StaticName="EDRMSActualCloseddate" Group="EDRMS" Hidden="FALSE" Required="FALSE" Sealed="FALSE" ShowInDisplayForm="TRUE" ShowInEditForm="TRUE" ShowInListSettings="TRUE" ShowInNewForm="TRUE" Format="DateOnly" SourceID="{f52ffc71-99ff-42c2-ad9c-88f88574918c}" AllowDeletion="TRUE" Customization="" ColName="datetime1" RowOrdinal="0"/>          <Field Type="Note" DisplayName="Assurance Type\_0" StaticName="p1ddcb168b464625aa2588fc1e161f6c" Name="p1ddcb168b464625aa2588fc1e161f6c" ID="{11bfcac9-af7d-48dc-b893-37fe198ac105}" ShowInViewForms="FALSE" Required="FALSE" CanToggleHidden="TRUE" SourceID="{f52ffc71-99ff-42c2-ad9c-88f88574918c}" AllowDeletion="TRUE" Hidden="TRUE" Customization="" ColName="ntext3" RowOrdinal="0"/>          <Field Type="LookupMulti" DisplayName="Taxonomy Catch All Column" StaticName="TaxCatchAll" Name="TaxCatchAll" ID="{f3b0adf9-c1a2-4b02-920d-943fba4b3611}" ShowInViewForms="FALSE" List="{15c95bf2-06cb-45a2-b11e-e96180856bbd}" WebId="39f2fce7-342b-4950-ad06-4a30a163b30f" Required="FALSE" CanToggleHidden="TRUE" ShowField="CatchAllData" SourceID="{f52ffc71-99ff-42c2-ad9c-88f88574918c}" Mult="TRUE" Sortable="FALSE" AllowDeletion="TRUE" Sealed="TRUE" Hidden="TRUE" Version="2" Customization="" ColName="int1" RowOrdinal="0"/>          <Field Type="LookupMulti" DisplayName="Taxonomy Catch All Column1" StaticName="TaxCatchAllLabel" Name="TaxCatchAllLabel" ID="{8f6b6dd8-9357-4019-8172-966fcd502ed2}" ShowInViewForms="FALSE" List="{15c95bf2-06cb-45a2-b11e-e96180856bbd}" WebId="39f2fce7-342b-4950-ad06-4a30a163b30f" Required="FALSE" CanToggleHidden="TRUE" ShowField="CatchAllDataLabel" FieldRef="{F3B0ADF9-C1A2-4b02-920D-943FBA4B3611}" SourceID="{f52ffc71-99ff-42c2-ad9c-88f88574918c}" Mult="TRUE" Sortable="FALSE" AllowDeletion="TRUE" Sealed="TRUE" Hidden="TRUE" ReadOnly="TRUE" Version="2" Customization=""/>          <Field Type="Text" Name="EDRMSAuditContact" ID="{978a583c-0ee7-4423-8db5-d63860d6f814}" Description="" DisplayName="Audit Contact" StaticName="EDRMSAuditContact" Group="EDRMS" Hidden="FALSE" Required="FALSE" Sealed="FALSE" ShowInDisplayForm="TRUE" ShowInEditForm="TRUE" ShowInListSettings="TRUE" ShowInNewForm="TRUE" SourceID="{f52ffc71-99ff-42c2-ad9c-88f88574918c}" AllowDeletion="TRUE" Customization="" ColName="nvarchar3" RowOrdinal="0"/>          <Field Type="Choice" Name="EDRMSCurrentstatus" ID="{30f91207-2921-489a-8d95-4aff394692b3}" Description="" DisplayName="Current status" StaticName="EDRMSCurrentstatus" Group="EDRMS" Hidden="FALSE" Required="FALSE" Sealed="FALSE" ShowInDisplayForm="TRUE" ShowInEditForm="TRUE" ShowInListSettings="TRUE" ShowInNewForm="TRUE" Format="Dropdown" SourceID="{f52ffc71-99ff-42c2-ad9c-88f88574918c}" AllowDeletion="TRUE" Customization="" ColName="nvarchar4" RowOrdinal="0">            <Default>Open</Default>            <CHOICES>              <CHOICE>Open</CHOICE>              <CHOICE>Closed</CHOICE>              <CHOICE>In Prohgress</CHOICE>            </CHOICES>          </Field>          <Field Type="Currency" Name="EDRMSD\_TSpendValuerequested" ID="{db74e757-efb1-4735-b300-e232027c891a}" Description="" DisplayName="D  T Spend Value requested" StaticName="EDRMSD\_TSpendValuerequested" Group="EDRMS" Hidden="FALSE" Required="FALSE" Sealed="FALSE" ShowInDisplayForm="TRUE" ShowInEditForm="TRUE" ShowInListSettings="TRUE" ShowInNewForm="TRUE" SourceID="{f52ffc71-99ff-42c2-ad9c-88f88574918c}" AllowDeletion="TRUE" Customization="" ColName="float1" RowOrdinal="0"/>          <Field ID="{1d22ea11-1e32-424e-89ab-9fedbadb6ce1}" ColName="tp\_ID" RowOrdinal="0" ReadOnly="TRUE" Type="Counter" Name="ID" PrimaryKey="TRUE" DisplayName="ID" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ID" FromBaseType="TRUE"/>          <Field ID="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Type="Computed" DisplayName="Content Type" Name="ContentType" DisplaceOnUpgrade="TRUE" RenderXMLUsingPattern="TRUE" Sortable="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ContentType" Group="\_Hidden" PITarget="MicrosoftWindowsSharePointServices" PIAttribute="ContentTypeID" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="ContentTypeId"/>            </FieldRefs>            <DisplayPattern>              <MapToContentType>                <Column Name="ContentTypeId"/>              </MapToContentType>            </DisplayPattern>          </Field>          <Field ID="{28cf69c5-fa48-462a-b5cd-27b6f9d2bd5f}" ColName="tp\_Modified" RowOrdinal="0" ReadOnly="TRUE" Type="DateTime" Name="Modified" DisplayName="Modified" StorageTZ="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Modified" FromBaseType="TRUE"/>          <Field ID="{8c06beca-0777-48f7-91c7-6da68bc07b69}" ColName="tp\_Created" RowOrdinal="0" ReadOnly="TRUE" Type="DateTime" Name="Created" DisplayName="Created" StorageTZ="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Created" FromBaseType="TRUE"/>          <Field ID="{1df5e554-ec7e-46a6-901d-d85a3881cb18}" ColName="tp\_Author" RowOrdinal="0" ReadOnly="TRUE" Type="User" List="UserInfo" Name="Author" DisplayName="Created By" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Author" FromBaseType="TRUE"/>          <Field ID="{d31655d1-1d5b-4511-95a1-7a09e9b75bf2}" ColName="tp\_Editor" RowOrdinal="0" ReadOnly="TRUE" Type="User" List="UserInfo" Name="Editor" DisplayName="Modified By" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Editor" FromBaseType="TRUE"/>          <Field ID="{26d0756c-986a-48a7-af35-bf18ab85ff4a}" ColName="tp\_HasCopyDestinations" RowOrdinal="0" Sealed="TRUE" Hidden="TRUE" ReadOnly="TRUE" Type="Boolean" Name="\_HasCopyDestinations" DisplaceOnUpgrade="TRUE" DisplayName="Has Copy Destinations" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_HasCopyDestinations" FromBaseType="TRUE"/>          <Field ID="{6b4e226d-3d88-4a36-808d-a129bf52bccf}" ColName="tp\_CopySource" RowOrdinal="0" Sealed="TRUE" Hidden="TRUE" ReadOnly="TRUE" Type="Text" Name="\_CopySource" DisplaceOnUpgrade="TRUE" DisplayName="Copy Source" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_CopySource" FromBaseType="TRUE"/>          <Field ID="{d4e44a66-ee3a-4d02-88c9-4ec5ff3f4cd5}" ColName="tp\_Version" RowOrdinal="0" Hidden="TRUE" ReadOnly="TRUE" Type="Integer" SetAs="owshiddenversion" Name="owshiddenversion" DisplayName="owshiddenversion" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="owshiddenversion" FromBaseType="TRUE"/>          <Field ID="{f1e020bc-ba26-443f-bf2f-b68715017bbc}" ColName="tp\_WorkflowVersion" RowOrdinal="0" Hidden="TRUE" ReadOnly="TRUE" Type="Integer" Name="WorkflowVersion" DisplaceOnUpgrade="TRUE" DisplayName="Workflow Version" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="WorkflowVersion" FromBaseType="TRUE"/>          <Field ID="{7841bf41-43d0-4434-9f50-a673baef7631}" ColName="tp\_UIVersion" RowOrdinal="0" ReadOnly="TRUE" Type="Integer" Name="\_UIVersion" DisplaceOnUpgrade="TRUE" DisplayName="UI Version" Hidden="TRUE" CanToggleHidden="TRUE" Required="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_UIVersion" FromBaseType="TRUE"/>          <Field ID="{dce8262a-3ae9-45aa-aab4-83bd75fb738a}" ColName="tp\_UIVersionString" RowOrdinal="0" ReadOnly="TRUE" Type="Text" Name="\_UIVersionString" DisplaceOnUpgrade="TRUE" DisplayName="Version" CanToggleHidden="TRUE" Required="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_UIVersionString" FromBaseType="TRUE"/>          <Field ID="{67df98f4-9dec-48ff-a553-29bece9c5bf4}" ColName="tp\_HasAttachment" RowOrdinal="0" Type="Attachments" Name="Attachments" DisplayName="Attachments" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Attachments" FromBaseType="TRUE"/>          <Field ID="{fdc3b2ed-5bf2-4835-a4bc-b885f3396a61}" ColName="tp\_ModerationStatus" RowOrdinal="0" ReadOnly="TRUE" Type="ModStat" Name="\_ModerationStatus" DisplayName="Approval Status" Hidden="TRUE" CanToggleHidden="TRUE" Required="FALSE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_ModerationStatus" FromBaseType="TRUE">            <CHOICES>              <CHOICE>0;#Approved</CHOICE>              <CHOICE>1;#Rejected</CHOICE>              <CHOICE>2;#Pending</CHOICE>              <CHOICE>3;#Draft</CHOICE>              <CHOICE>4;#Scheduled</CHOICE>            </CHOICES>            <Default>0</Default>          </Field>          <Field ID="{503f1caa-358e-4918-9094-4a2cdc4bc034}" ReadOnly="TRUE" Type="Computed" Name="Edit" Sortable="FALSE" Filterable="FALSE" DisplayName="Edit" ClassInfo="Icon" AuthoringInfo="(link to edit item)" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Edit" FromBaseType="TRUE">            <DisplayPattern>              <IfHasRights>                <RightsChoices>                  <RightsGroup PermEditListItems="required"/>                </RightsChoices>                <Then>                  <HTML><!\[CDATA\[<a href="\]\]></HTML>                  <URL Cmd="Edit"/>                  <HTML><!\[CDATA\[" onclick="EditLink(this, \]\]></HTML>                  <Counter Type="View"/>                  <HTML><!\[CDATA\[);return false;" target="\_self">\]\]></HTML>                  <HTML><!\[CDATA\[<img border="0" alt="\]\]></HTML>                  <HTML>Edit</HTML>                  <HTML><!\[CDATA\[" src="/\_layouts/15/images/edititem.gif?rev=23"/>\]\]></HTML>                  <HTML><!\[CDATA\[</a>\]\]></HTML>                </Then>                <Else>                  <HTML><!\[CDATA\[160;\]\]></HTML>                </Else>              </IfHasRights>            </DisplayPattern>          </Field>          <Field ID="{bc91a437-52e7-49e1-8c4e-4698904b2b6d}" ReadOnly="TRUE" Type="Computed" Name="LinkTitleNoMenu" DisplayName="Title" Dir="" DisplayNameSrcField="Title" AuthoringInfo="(linked to item)" EnableLookup="TRUE" ListItemMenuAllowed="Prohibited" LinkToItemAllowed="Prohibited" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="LinkTitleNoMenu" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="Title"/>              <FieldRef Name="LinkFilenameNoMenu"/>            </FieldRefs>            <DisplayPattern>              <IfEqual>                <Expr1>                  <LookupColumn Name="FSObjType"/>                </Expr1>                <Expr2>1</Expr2>                <Then>                  <Field Name="LinkFilenameNoMenu"/>                </Then>                <Else>                  <HTML><!\[CDATA\[<a onfocus="OnLink(this)" href="\]\]></HTML>                  <URL/>                  <HTML><!\[CDATA\[" onclick="EditLink2(this,\]\]></HTML>                  <Counter Type="View"/>                  <HTML><!\[CDATA\[);return false;" target="\_self">\]\]></HTML>                  <Column HTMLEncode="TRUE" Name="Title" Default="(no title)"/>                  <IfEqual>                    <Expr1>                      <GetVar Name="ShowAccessibleIcon"/>                    </Expr1>                    <Expr2>1</Expr2>                    <Then>                      <HTML><!\[CDATA\[<img src="/\_layouts/15/images/blank.gif?rev=23" class="ms-hidden" border="0" width="1" height="1" alt="Use SHIFT+ENTER to open the menu (new window)."/>\]\]></HTML>                    </Then>                  </IfEqual>                  <HTML><!\[CDATA\[</a>\]\]></HTML>                  <IfNew>                    <HTML><!\[CDATA\[<img src="/\_layouts/1033/images/new.gif" alt="\]\]></HTML>                    <HTML>New</HTML>                    <HTML><!\[CDATA\[" class="ms-newgif" />\]\]></HTML>                  </IfNew>                </Else>              </IfEqual>            </DisplayPattern>          </Field>          <Field ID="{82642ec8-ef9b-478f-acf9-31f7d45fbc31}" ReadOnly="TRUE" Type="Computed" Name="LinkTitle" DisplayName="Title" DisplayNameSrcField="Title" ClassInfo="Menu" AuthoringInfo="(linked to item with edit menu)" ListItemMenuAllowed="Required" LinkToItemAllowed="Prohibited" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="LinkTitle" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="Title"/>              <FieldRef Name="LinkTitleNoMenu"/>              <FieldRef Name="\_EditMenuTableStart2"/>              <FieldRef Name="\_EditMenuTableEnd"/>            </FieldRefs>            <DisplayPattern>              <FieldSwitch>                <Expr>                  <GetVar Name="FreeForm"/>                </Expr>                <Case Value="TRUE">                  <Field Name="LinkTitleNoMenu"/>                </Case>                <Default>                  <HTML><!\[CDATA\[

                                    \]\]>                                    \]\]></HTML>                  <HTML><!\[CDATA\[

\]\]>                   \]\]>                  \]\]>                   \]\]>                  \]\]></HTML>                </Default>              </FieldSwitch>            </DisplayPattern>          </Field>          <Field ID="{5f190d91-3dbc-4489-9878-3c092caf35b6}" Hidden="TRUE" ReadOnly="TRUE" Type="Computed" Name="LinkTitle2" DisplayName="Title" DisplayNameSrcField="Title" ClassInfo="Menu" AuthoringInfo="(linked to item with edit menu) (old)" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="LinkTitle2" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="Title"/>              <FieldRef Name="LinkTitleNoMenu"/>              <FieldRef Name="\_EditMenuTableStart"/>              <FieldRef Name="\_EditMenuTableEnd"/>            </FieldRefs>            <DisplayPattern>              <FieldSwitch>                <Expr>                  <GetVar Name="FreeForm"/>                </Expr>                <Case Value="TRUE">                  <Field Name="LinkTitleNoMenu"/>                </Case>                <Default>                  <Field Name="\_EditMenuTableStart"/>                  <SetVar Name="ShowAccessibleIcon" Value="1"/>                  <Field Name="LinkTitleNoMenu"/>                  <SetVar Name="ShowAccessibleIcon" Value="0"/>                  <Field Name="\_EditMenuTableEnd"/>                </Default>              </FieldSwitch>            </DisplayPattern>          </Field>          <Field ID="{b1f7969b-ea65-42e1-8b54-b588292635f2}" ReadOnly="TRUE" Type="Computed" Sortable="FALSE" Filterable="FALSE" Name="SelectTitle" Hidden="TRUE" CanToggleHidden="TRUE" DisplayName="Select" Dir="" AuthoringInfo="(web part connection)" HeaderImage="blank.gif" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="SelectTitle" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="ID"/>            </FieldRefs>            <DisplayPattern>              <IfEqual>                <Expr1>                  <GetVar Name="SelectedID"/>                </Expr1>                <Expr2>                  <Column Name="ID"/>                </Expr2>                <Then>                  <HTML><!\[CDATA\[<img border="0" align="absmiddle" style="cursor: pointer" src="/\_layouts/15/images/rbsel.gif?rev=23" alt="\]\]></HTML>                  <HTML>Selected</HTML>                  <HTML><!\[CDATA\["/>\]\]></HTML>                </Then>                <Else>                  <HTML><!\[CDATA\[<a href="javascript:SelectField('\]\]></HTML>                  <GetVar Name="View"/>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Column Name="ID"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[');return false;" onclick="javascript:SelectField('\]\]></HTML>                  <GetVar Name="View"/>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Column Name="ID"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[');return false;" target="\_self">\]\]></HTML>                  <HTML><!\[CDATA\[<img border="0" align="absmiddle" style="cursor: pointer" src="/\_layouts/15/images/rbunsel.gif?rev=23"  alt="\]\]></HTML>                  <HTML>Normal</HTML>                  <HTML><!\[CDATA\["/>\]\]></HTML>                  <HTML><!\[CDATA\[</a>\]\]></HTML>                </Else>              </IfEqual>            </DisplayPattern>          </Field>          <Field ID="{50a54da4-1528-4e67-954a-e2d24f1e9efb}" Name="InstanceID" DisplayName="Instance ID" ColName="tp\_InstanceID" RowOrdinal="0" ReadOnly="TRUE" Hidden="TRUE" Type="Integer" Min="0" Max="99991231" Filterable="TRUE" Sortable="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="InstanceID" FromBaseType="TRUE"/>          <Field ID="{ca4addac-796f-4b23-b093-d2a3f65c0774}" ColName="tp\_ItemOrder" RowOrdinal="0" Name="Order" DisplayName="Order" Type="Number" Hidden="TRUE" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Order" FromBaseType="TRUE"/>          <Field ID="{ae069f25-3ac2-4256-b9c3-15dbc15da0e0}" ColName="tp\_GUID" RowOrdinal="0" ReadOnly="TRUE" Hidden="TRUE" Type="Guid" Name="GUID" DisplayName="GUID" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="GUID" FromBaseType="TRUE"/>          <Field ID="{de8beacf-5505-47cd-80a6-aa44e7ffe2f4}" ColName="tp\_WorkflowInstanceID" RowOrdinal="0" ReadOnly="TRUE" Hidden="TRUE" Type="Guid" Name="WorkflowInstanceID" DisplaceOnUpgrade="TRUE" DisplayName="Workflow Instance ID" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="WorkflowInstanceID" FromBaseType="TRUE"/>          <Field ID="{94f89715-e097-4e8b-ba79-ea02aa8b7adb}" Name="FileRef" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" Type="Lookup" DisplayName="URL Path" List="Docs" FieldRef="ID" ShowField="FullUrl" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="FileRef" FromBaseType="TRUE"/>          <Field ID="{56605df6-8fa1-47e4-a04c-5b384d59609f}" Name="FileDirRef" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" Type="Lookup" DisplayName="Path" List="Docs" FieldRef="ID" ShowField="DirName" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="FileDirRef" FromBaseType="TRUE"/>          <Field ID="{173f76c8-aebd-446a-9bc9-769a2bd2c18f}" Name="Last\_x0020\_Modified" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" DisplayName="Modified" Type="Lookup" List="Docs" FieldRef="ID" ShowField="TimeLastModified" Format="TRUE" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Last\_x0020\_Modified" FromBaseType="TRUE"/>          <Field ID="{998b5cff-4a35-47a7-92f3-3914aa6aa4a2}" Name="Created\_x0020\_Date" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" DisplayName="Created" Type="Lookup" List="Docs" FieldRef="ID" ShowField="TimeCreated" Format="TRUE" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="Created\_x0020\_Date" FromBaseType="TRUE"/>          <Field ID="{30bb605f-5bae-48fe-b4e3-1f81d9772af9}" Name="FSObjType" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="Item Type" List="Docs" FieldRef="ID" ShowField="FSType" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="FSObjType" FromBaseType="TRUE"/>          <Field ID="{423874f8-c300-4bfb-b7a1-42e2159e3b19}" Name="SortBehavior" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="Sort Type" List="Docs" FieldRef="ID" ShowField="SortBehavior" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="SortBehavior" FromBaseType="TRUE"/>          <Field ID="{ba3c27ee-4791-4867-8821-ff99000bac98}" Name="PermMask" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" RenderXMLUsingPattern="TRUE" ShowInFileDlg="FALSE" Type="Computed" DisplayName="Effective Permissions Mask" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="PermMask" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="ID"/>            </FieldRefs>            <DisplayPattern>              <CurrentRights/>            </DisplayPattern>          </Field>          <Field ID="{8553196d-ec8d-4564-9861-3dbe931050c8}" Hidden="TRUE" ShowInFileDlg="FALSE" ShowInVersionHistory="FALSE" Type="File" Name="FileLeafRef" DisplaceOnUpgrade="TRUE" DisplayName="Name" AuthoringInfo="(for use in forms)" List="Docs" FieldRef="ID" ShowField="LeafName" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="FileLeafRef" FromBaseType="TRUE"/>          <Field ID="{4b7403de-8d94-43e8-9f0f-137a3e298126}" Name="UniqueId" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="Unique Id" List="Docs" FieldRef="ID" ShowField="UniqueId" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="UniqueId" FromBaseType="TRUE"/>          <Field ID="{6d2c4fde-3605-428e-a236-ce5f3dc2b4d4}" Name="SyncClientId" DisplaceOnUpgrade="TRUE" Hidden="TRUE" ReadOnly="TRUE" DisplayName="Client Id" ShowInFileDlg="FALSE" Type="Lookup" List="Docs" FieldRef="ID" ShowField="SyncClientId" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="SyncClientId" FromBaseType="TRUE"/>          <Field ID="{c5c4b81c-f1d9-4b43-a6a2-090df32ebb68}" Name="ProgId" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="ProgId" List="Docs" FieldRef="ID" ShowField="ProgId" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ProgId" FromBaseType="TRUE"/>          <Field ID="{dddd2420-b270-4735-93b5-92b713d0944d}" Name="ScopeId" DisplaceOnUpgrade="TRUE" ReadOnly="TRUE" Hidden="TRUE" ShowInFileDlg="FALSE" Type="Lookup" DisplayName="ScopeId" List="Docs" FieldRef="ID" ShowField="ScopeId" JoinColName="DoclibRowId" JoinRowOrdinal="0" JoinType="INNER" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="ScopeId" FromBaseType="TRUE"/>          <Field ReadOnly="TRUE" ID="{4ef1b78f-fdba-48dc-b8ab-3fa06a0c9804}" Hidden="TRUE" Type="Computed" Name="HTML\_x0020\_File\_x0020\_Type" DisplaceOnUpgrade="TRUE" DisplayName="HTML File Type" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="HTML\_x0020\_File\_x0020\_Type" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="File\_x0020\_Type"/>            </FieldRefs>            <DisplayPattern/>          </Field>          <Field ID="{3c6303be-e21f-4366-80d7-d6d0a3b22c7a}" Hidden="TRUE" ReadOnly="TRUE" Type="Computed" Name="\_EditMenuTableStart" DisplaceOnUpgrade="TRUE" DisplayName="Edit Menu Table Start" ClassInfo="Menu" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="\_EditMenuTableStart" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="FileLeafRef"/>              <FieldRef Name="FileDirRef"/>              <FieldRef Name="FSObjType"/>              <FieldRef Name="ID"/>              <FieldRef Name="ServerUrl"/>              <FieldRef Name="HTML\_x0020\_File\_x0020\_Type"/>              <FieldRef Name="File\_x0020\_Type"/>              <FieldRef Name="PermMask"/>              <FieldRef Name="\_HasCopyDestinations"/>              <FieldRef Name="\_CopySource"/>              <FieldRef Name="ContentType"/>              <FieldRef Name="ContentTypeId"/>              <FieldRef Name="\_ModerationStatus"/>              <FieldRef Name="\_UIVersion"/>            </FieldRefs>            <DisplayPattern>              <HTML><!\[CDATA\[

                            " id="                            " Url="                            " DRef="                            " Perm="                            " type="                            " Ext="                            " Icon="                                              |                                            " OType="                            " COUId="              " HCD="                            " CSrc="                            " MS="                                                                                    " UIS="                                          " SUrl="              \]\]>                                                                                                              " id="                                                                                                              \]\]></HTML>              <HTML><!\[CDATA\[

\]\]>               \]\]>              \]\]>               \]\]>              \]\]></HTML>            </DisplayPattern>          </Field>          <Field ID="{9d30f126-ba48-446b-b8f9-83745f322ebe}" ReadOnly="TRUE" Type="Computed" Name="LinkFilenameNoMenu" DisplaceOnUpgrade="TRUE" DisplayName="Name" Hidden="TRUE" DisplayNameSrcField="FileLeafRef" Filterable="FALSE" AuthoringInfo="(linked to document)" ListItemMenuAllowed="Prohibited" LinkToItemAllowed="Prohibited" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="LinkFilenameNoMenu" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="FileLeafRef"/>              <FieldRef Name="FSObjType"/>              <FieldRef Name="Created\_x0020\_Date"/>              <FieldRef Name="FileRef"/>              <FieldRef Name="File\_x0020\_Type"/>              <FieldRef Name="HTML\_x0020\_File\_x0020\_Type"/>              <FieldRef Name="ContentTypeId"/>              <FieldRef Name="PermMask"/>            </FieldRefs>            <DisplayPattern>              <IfEqual>                <Expr1>                  <LookupColumn Name="FSObjType"/>                </Expr1>                <Expr2>1</Expr2>                <Then>                  <FieldSwitch>                    <Expr>                      <GetVar Name="RecursiveView"/>                    </Expr>                    <Case Value="1">                      <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/>                    </Case>                    <Default>                      <SetVar Name="UnencodedFilterLink">                        <SetVar Name="RootFolder">                          <HTML>/</HTML>                          <LookupColumn Name="FileRef"/>                        </SetVar>                        <SetVar Name="SkipHost">1</SetVar>                        <SetVar Name="FolderCTID">                          <FieldSwitch>                            <Expr>                              <ListProperty Select="EnableContentTypes"/>                            </Expr>                            <Case Value="1">                              <Column Name="ContentTypeId"/>                            </Case>                          </FieldSwitch>                        </SetVar>                        <FilterLink Default="" Paged="FALSE"/>                      </SetVar>                      <HTML><!\[CDATA\[<a onfocus="OnLink(this)" href="\]\]></HTML>                      <GetVar Name="UnencodedFilterLink" HTMLEncode="TRUE"/>                      <HTML><!\[CDATA\[" onmousedown="javascript:VerifyFolderHref(this,event, '\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <GetVar Name="UnencodedFilterLink"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ListProperty Select="DefaultItemOpen"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <MapToControl>                          <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                          <HTML>|</HTML>                          <Column Name="File\_x0020\_Type"/>                        </MapToControl>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <HTML><!\[CDATA\[')"\]\]></HTML>                      <HTML><!\[CDATA\[" onclick="return HandleFolder(this,event, '\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <GetVar Name="UnencodedFilterLink"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ServerProperty Select="HtmlTransform"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ListProperty Select="DefaultItemOpen"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <MapToControl>                          <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                          <HTML>|</HTML>                          <Column Name="File\_x0020\_Type"/>                        </MapToControl>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <UserID AllowAnonymous="TRUE"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <ListProperty Select="ForceCheckout"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <HTML><!\[CDATA\[','\]\]></HTML>                      <ScriptQuote NotAddingQuote="TRUE">                        <Field Name="PermMask"/>                      </ScriptQuote>                      <HTML><!\[CDATA\[');">\]\]></HTML>                      <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/>                      <IfEqual>                        <Expr1>                          <GetVar Name="ShowAccessibleIcon"/>                        </Expr1>                        <Expr2>1</Expr2>                        <Then>                          <HTML><!\[CDATA\[<img src="/\_layouts/15/images/blank.gif?rev=23" class="ms-hidden" border="0" width="1" height="1" alt="Use SHIFT+ENTER to open the menu (new window)."/>\]\]></HTML>                        </Then>                      </IfEqual>                      <HTML><!\[CDATA\[</a>\]\]></HTML>                    </Default>                  </FieldSwitch>                </Then>                <Else>                  <HTML><!\[CDATA\[<a onfocus="OnLink(this)" href="\]\]></HTML>                  <Field Name="ServerUrl" URLEncodeAsURL="TRUE"/>                  <HTML><!\[CDATA\[" onclick="return DispEx(this,event,'\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ServerProperty Select="HtmlTransform"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ListProperty Select="DefaultItemOpen"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <MapToControl>                      <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                      <HTML>|</HTML>                      <Column Name="File\_x0020\_Type"/>                    </MapToControl>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <UserID AllowAnonymous="TRUE"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <ListProperty Select="ForceCheckout"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <HTML><!\[CDATA\[','\]\]></HTML>                  <ScriptQuote NotAddingQuote="TRUE">                    <Field Name="PermMask"/>                  </ScriptQuote>                  <HTML><!\[CDATA\[')">\]\]></HTML>                  <UrlBaseName HTMLEncode="TRUE">                    <LookupColumn Name="FileLeafRef"/>                  </UrlBaseName>                  <IfEqual>                    <Expr1>                      <GetVar Name="ShowAccessibleIcon"/>                    </Expr1>                    <Expr2>1</Expr2>                    <Then>                      <HTML><!\[CDATA\[<img src="/\_layouts/15/images/blank.gif?rev=23" class="ms-hidden" border="0" width="1" height="1" alt="Use SHIFT+ENTER to open the menu (new window)."/>\]\]></HTML>                    </Then>                  </IfEqual>                  <HTML><!\[CDATA\[</a>\]\]></HTML>                  <IfNew Name="Created\_x0020\_Date">                    <HTML><!\[CDATA\[<img src="/\_layouts/15/1033/images/new.gif" alt="\]\]></HTML>                    <HTML>New</HTML>                    <HTML><!\[CDATA\[" class="ms-newgif" />\]\]></HTML>                  </IfNew>                </Else>              </IfEqual>            </DisplayPattern>          </Field>          <Field ID="{5cc6dc79-3710-4374-b433-61cb4a686c12}" ReadOnly="TRUE" Type="Computed" Name="LinkFilename" DisplaceOnUpgrade="TRUE" Hidden="TRUE" DisplayName="Name" DisplayNameSrcField="FileLeafRef" Filterable="FALSE" ClassInfo="Menu" AuthoringInfo="(linked to document with edit menu)" ListItemMenuAllowed="Required" LinkToItemAllowed="Prohibited" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="LinkFilename" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="LinkFilenameNoMenu"/>              <FieldRef Name="\_EditMenuTableStart2"/>              <FieldRef Name="\_EditMenuTableEnd"/>            </FieldRefs>            <DisplayPattern>              <FieldSwitch>                <Expr>                  <GetVar Name="FreeForm"/>                </Expr>                <Case Value="TRUE">                  <Field Name="LinkFilenameNoMenu"/>                </Case>                <Default>                  <HTML><!\[CDATA\[

                                    \]\]>                                    \]\]></HTML>                  <HTML><!\[CDATA\[

\]\]>                   \]\]>                  \]\]>                   \]\]>                  \]\]></HTML>                </Default>              </FieldSwitch>            </DisplayPattern>          </Field>          <Field ID="{224ba411-da77-4050-b0eb-62d422f13d3e}" Hidden="TRUE" ReadOnly="TRUE" Type="Computed" Name="LinkFilename2" DisplaceOnUpgrade="TRUE" DisplayName="Name" DisplayNameSrcField="FileLeafRef" Filterable="FALSE" ClassInfo="Menu" AuthoringInfo="(linked to document with edit menu) (old)" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="LinkFilename2" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="LinkFilenameNoMenu"/>              <FieldRef Name="\_EditMenuTableStart"/>              <FieldRef Name="\_EditMenuTableEnd"/>            </FieldRefs>            <DisplayPattern>              <FieldSwitch>                <Expr>                  <GetVar Name="FreeForm"/>                </Expr>                <Case Value="TRUE">                  <Field Name="LinkFilenameNoMenu"/>                </Case>                <Default>                  <Field Name="\_EditMenuTableStart"/>                  <SetVar Name="ShowAccessibleIcon" Value="1"/>                  <Field Name="LinkFilenameNoMenu"/>                  <SetVar Name="ShowAccessibleIcon" Value="0"/>                  <Field Name="\_EditMenuTableEnd"/>                </Default>              </FieldSwitch>            </DisplayPattern>          </Field>          <Field ID="{081c6e4c-5c14-4f20-b23e-1a71ceb6a67c}" Type="Computed" ReadOnly="TRUE" Name="DocIcon" DisplaceOnUpgrade="TRUE" DisplayName="Type" TextOnly="TRUE" ClassInfo="Icon" AuthoringInfo="(icon linked to document)" SourceID="http://schemas.microsoft.com/sharepoint/v3" StaticName="DocIcon" FromBaseType="TRUE">            <FieldRefs>              <FieldRef Name="File\_x0020\_Type"/>              <FieldRef Name="FSObjType"/>              <FieldRef Name="FileRef"/>              <FieldRef Name="FileLeafRef"/>              <FieldRef Name="HTML\_x0020\_File\_x0020\_Type"/>              <FieldRef Name="PermMask"/>              <FieldRef Name="IconOverlay"/>            </FieldRefs>            <DisplayPattern>              <SetVar Name="DocIconImg">                <SetVar Name="DocIconAltText">                  <IfEqual>                    <Expr1>                      <LookupColumn Name="FSObjType"/>                    </Expr1>                    <Expr2>1</Expr2>                    <Then>                      <IfSubString>                        <Expr1>0x0120D5</Expr1>                        <Expr2>                          <Column Name="ContentTypeId"/>                        </Expr2>                        <Then>                          <HTML>Document Collection: </HTML>                          <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/>                        </Then>                        <Else>                          <HTML>Folder: </HTML>                          <LookupColumn Name="FileLeafRef" HTMLEncode="TRUE"/>                        </Else>                      </IfSubString>                    </Then>                    <Else>                      <LookupColumn Name="Title" HTMLEncode="TRUE"/>                    </Else>                  </IfEqual>                </SetVar>                <SetVar Name="DocIconFileName">                  <IfEqual>                    <Expr1>                      <Column Name="IconOverlay"/>                    </Expr1>                    <Expr2/>                    <Then>                      <IfEqual>                        <Expr1>                          <LookupColumn Name="FSObjType"/>                        </Expr1>                        <Expr2>1</Expr2>                        <Then>                          <IfEqual>                            <Expr1>                              <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                              <HTML>|</HTML>                              <Column Name="File\_x0020\_Type"/>                            </Expr1>                            <Expr2>                              <HTML>|</HTML>                            </Expr2>                            <Then>                              <HTML>folder.gif</HTML>                            </Then>                            <Else>                              <SetVar Name="FolderIconFromMap">                                <MapToIcon>                                  <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                                  <HTML>|</HTML>                                  <Column Name="File\_x0020\_Type"/>                                </MapToIcon>                              </SetVar>                              <IfEqual>                                <Expr1>                                  <GetVar Name="FolderIconFromMap"/>                                </Expr1>                                <Expr2>                                  <MapToIcon/>                                </Expr2>                                <Then>                                  <HTML>folder.gif</HTML>                                </Then>                                <Else>                                  <GetVar Name="FolderIconFromMap"/>                                </Else>                              </IfEqual>                            </Else>                          </IfEqual>                        </Then>                        <Else>                          <MapToIcon>                            <Column Name="HTML\_x0020\_File\_x0020\_Type"/>                            <HTML>|</HTML>                            <Column Name="File\_x0020\_Type"/>                          </MapToIcon>                        </Else>                      </IfEqual>                    </Then>                    <Else>                      <MapToIcon>                        <Column Name="IconOverlay"/>                      </MapToIcon>                    </Else>                  </IfEqual>                </SetVar>                <HTML><!\[CDATA\[<img border="0" alt="\]\]></HTML>                <GetVar Name="DocIconAltText"/>                <HTML><!\[CDATA\[" title="\]\]></HTML>                <GetVar Name="DocIconAltText"/>                <HTML><!\[CDATA\[" src="/\_layouts/15/images/\]\]></HTML>                <GetVar Name="DocIconFileName"/>                <HTML><!\[CDATA\[" />\]\]></HTML>              </SetVar>              <SetVar Name="DocIconOverlayImg">                <IfEqual>                  <Expr1>                    <Column Name="IconOverlay"/>                  </Expr1>                  <Expr2/>                  <Then/>                  <Else>                    <HTML><!\[CDATA\[<img class="ms-vb-icon-overlay" alt="\*" src="/\_layouts/15/images/\]\]></HTML>                    <MapToOverlay>                      <Column Name="IconOverlay"/>                    </MapToOverlay>                    <HTML><!\[CDATA\[" />\]\]></HTML>                  </Else>                </IfEqual>              </SetVar>              <IfEqual>                <Expr1>                  <LookupColumn Name="FSObjType"/>                </Expr1>                <Expr2>1</Expr2>                <Then>                  <FieldSwitch>                    <Expr>                      <GetVar Name="RecursiveView"/>                    </Expr>                    <Case Value="1">                      <GetVar Name="DocIconImg"/>                      <GetVar Name="DocIconOverlayImg"/>                    </Case>                    <Default>                      <SetVar Name="UnencodedFilterLink">                        <SetVar Name="RootFolder">                          <HTML>/</HTML>                          <LookupColumn Name="FileRef"/>                        </SetVar>                        <SetVar Name="SkipHost">1</SetVar>                        <SetVar Name="FolderCTID">                          <FieldSwitch>                            <Expr>                              <ListProperty Select="EnableContentTypes"/>                            </Expr>                            <Case Value="1">                              <Column Name="ContentTypeId"/>                            </Case>                          </FieldSwitch>                        </SetVar>                        <FilterLink Default="" Paged="FALSE"/>                      </SetVar>                      <FieldSwitch>                        <Expr>                          <GetVar Name="FileDialog"/>                        </Expr>                        <Case Value="1">                          <GetVar Name="DocIconImg"/>                          <GetVar Name="DocIconOverlayImg"/>                        </Case>                        <Default>                          <HTML><!\[CDATA\[<a href="\]\]></HTML>                          <GetVar Name="UnencodedFilterLink" HTMLEncode="TRUE"/>                          <HTML><!\[CDATA\[" onclick="javascript:EnterFolder('\]\]></HTML>                          <ScriptQuote NotAddingQuote="TRUE">                            <GetVar Name="UnencodedFilterLink"/>                          </ScriptQuote>                          <HTML><!\[CDATA\[');javascript:return false;">\]\]></HTML>                          <GetVar Name="DocIconImg"/>                          <GetVar Name="DocIconOverlayImg"/>                          <HTML><!\[CDATA\[</a>\]\]></HTML>                        </Default>                      </FieldSwitch>                    </Default>                  </FieldSwitch>                </Then>                <Else>                  <HTML><!\[CDATA\[<a onfocus="OnLink(this)" href="\]\]></HTML>                  <URL/>                  <HTML><!\[CDATA\[" onclick="GoToLink(this);return false;" target="\_self">\]\]></HTML>                  <GetVar Name="DocIconImg"/>                  <GetVar Name="DocIconOverlayImg"/>                  <HTML><!\[CDATA\[</a>\]\]></HTML>                </Else>              </IfEqual>            </DisplayPattern>          </Field>          <Field ID\="{105f76ce-724a-4bba-aece-f81f2fce58f5}" ReadOnly\="TRUE" Hidden\="TRUE" Type\="Computed" Name\="ServerUrl" DisplaceOnUpgrade\="TRUE" DisplayName\="Server Relative URL" Filterable\="FALSE" RenderXMLUsingPattern\="TRUE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="ServerUrl" FromBaseType\="TRUE"\>           <FieldRefs\>             <FieldRef Name\="FileRef"/>            </FieldRefs>            <DisplayPattern\>             <HTML\>/</HTML>              <LookupColumn Name\="FileRef"/>            </DisplayPattern>          </Field>          <Field ID\="{7177cfc7-f399-4d4d-905d-37dd51bc90bf}" ReadOnly\="TRUE" Hidden\="TRUE" Type\="Computed" Name\="EncodedAbsUrl" DisplaceOnUpgrade\="TRUE" DisplayName\="Encoded Absolute URL" Filterable\="FALSE" RenderXMLUsingPattern\="TRUE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="EncodedAbsUrl" FromBaseType\="TRUE"\>           <FieldRefs\>             <FieldRef Name\="FileRef"/>            </FieldRefs>            <DisplayPattern\>             <HttpHost URLEncodeAsURL\="TRUE"/>              <HTML\>/</HTML>              <LookupColumn Name\="FileRef" IncludeVersions\="TRUE" URLEncodeAsURL\="TRUE"/>            </DisplayPattern>          </Field>          <Field ID\="{7615464b-559e-4302-b8e2-8f440b913101}" ReadOnly\="TRUE" Hidden\="TRUE" Type\="Computed" Name\="BaseName" DisplaceOnUpgrade\="TRUE" DisplayName\="File Name" Filterable\="FALSE" RenderXMLUsingPattern\="TRUE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="BaseName" FromBaseType\="TRUE"\>           <FieldRefs\>             <FieldRef Name\="FileLeafRef"/>              <FieldRef Name\="FSObjType"/>            </FieldRefs>            <DisplayPattern\>             <IfEqual\>               <Expr1\>                 <LookupColumn Name\="FSObjType"/>                </Expr1>                <Expr2\>1</Expr2>                <Then\>                 <LookupColumn Name\="FileLeafRef" HTMLEncode\="TRUE"/>                </Then>                <Else\>                 <UrlBaseName HTMLEncode\="TRUE"\>                   <LookupColumn Name\="FileLeafRef"/>                  </UrlBaseName>                </Else>              </IfEqual>            </DisplayPattern>          </Field>          <Field ID\="{687c7f94-686a-42d3-9b67-2782eac4b4f8}" Name\="MetaInfo" DisplaceOnUpgrade\="TRUE" Hidden\="TRUE" ShowInFileDlg\="FALSE" Type\="Lookup" DisplayName\="Property Bag" List\="Docs" FieldRef\="ID" ShowField\="MetaInfo" JoinColName\="DoclibRowId" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="MetaInfo" FromBaseType\="TRUE"/>          <Field ID\="{43bdd51b-3c5b-4e78-90a8-fb2087f71e70}" ColName\="tp\_Level" RowOrdinal\="0" ReadOnly\="TRUE" Type\="Integer" Name\="\_Level" DisplaceOnUpgrade\="TRUE" DisplayName\="Level" Hidden\="TRUE" Required\="FALSE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="\_Level" FromBaseType\="TRUE"/>          <Field ID\="{c101c3e7-122d-4d4d-bc34-58e94a38c816}" ColName\="tp\_IsCurrentVersion" DisplaceOnUpgrade\="TRUE" RowOrdinal\="0" ReadOnly\="TRUE" Type\="Boolean" Name\="\_IsCurrentVersion" DisplayName\="Is Current Version" Hidden\="TRUE" Required\="FALSE" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="\_IsCurrentVersion" FromBaseType\="TRUE"/>          <Field ID\="{b824e17e-a1b3-426e-aecf-f0184d900485}" Name\="ItemChildCount" DisplaceOnUpgrade\="TRUE" ReadOnly\="TRUE" ShowInFileDlg\="FALSE" Type\="Lookup" DisplayName\="Item Child Count" List\="Docs" FieldRef\="ID" ShowField\="ItemChildCount" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="ItemChildCount" FromBaseType\="TRUE"/>          <Field ID\="{960ff01f-2b6d-4f1b-9c3f-e19ad8927341}" Name\="FolderChildCount" DisplaceOnUpgrade\="TRUE" ReadOnly\="TRUE" ShowInFileDlg\="FALSE" Type\="Lookup" DisplayName\="Folder Child Count" List\="Docs" FieldRef\="ID" ShowField\="FolderChildCount" JoinColName\="DoclibRowId" JoinRowOrdinal\="0" JoinType\="INNER" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="FolderChildCount" FromBaseType\="TRUE"/>          <Field ID\="{6bfaba20-36bf-44b5-a1b2-eb6346d49716}" ColName\="tp\_AppAuthor" RowOrdinal\="0" ReadOnly\="TRUE" Hidden\="FALSE" Type\="Lookup" List\="AppPrincipals" Name\="AppAuthor" DisplayName\="App Created By" ShowField\="Title" JoinColName\="Id" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="AppAuthor" FromBaseType\="TRUE"/>          <Field ID\="{e08400f3-c779-4ed2-a18c-ab7f34caa318}" ColName\="tp\_AppEditor" RowOrdinal\="0" ReadOnly\="TRUE" Hidden\="FALSE" Type\="Lookup" List\="AppPrincipals" Name\="AppEditor" DisplayName\="App Modified By" ShowField\="Title" JoinColName\="Id" SourceID\="http://schemas.microsoft.com/sharepoint/v3" StaticName\="AppEditor" FromBaseType\="TRUE"/>        </Fields>        <ContentTypes\>         <ContentType ID\="0x0100C24F1844B7B3264B92EBC5B3F0110A9F" Name\="Item" Group\="List Content Types" Description\="Create a new list item." Version\="0" FeatureId\="{695b6570-a48b-4a8e-8ea5-26ea7fc1d162}"\>           <Folder TargetName\="Item"/>            <FieldRefs\>             <FieldRef ID\="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Name\="ContentType"/>              <FieldRef ID\="{fa564e0f-0c70-4ab9-b863-0177e6ddd247}" Name\="Title" Required\="TRUE" ShowInNewForm\="TRUE" ShowInEditForm\="TRUE"/>            </FieldRefs>            <XmlDocuments\>             <XmlDocument NamespaceURI\="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms"\>PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+TGlzdEZvcm08L0Rpc3BsYXk+PEVkaXQ+TGlzdEZvcm08L0VkaXQ+PE5ldz5MaXN0Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4=</XmlDocument>            </XmlDocuments>          </ContentType>          <ContentType ID\="0x012000DF65443D88AF084791B2BCB044BE0BF7" Name\="Folder" Group\="Folder Content Types" Description\="Create a new folder." Sealed\="TRUE" Version\="0" FeatureId\="{695b6570-a48b-4a8e-8ea5-26ea7fc1d162}"\>           <FieldRefs\>             <FieldRef ID\="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Name\="ContentType"/>              <FieldRef ID\="{fa564e0f-0c70-4ab9-b863-0177e6ddd247}" Name\="Title" Required\="FALSE" Hidden\="TRUE"/>              <FieldRef ID\="{8553196d-ec8d-4564-9861-3dbe931050c8}" Name\="FileLeafRef" Required\="TRUE" Hidden\="FALSE"/>              <FieldRef ID\="{b824e17e-a1b3-426e-aecf-f0184d900485}" Name\="ItemChildCount"/>              <FieldRef ID\="{960ff01f-2b6d-4f1b-9c3f-e19ad8927341}" Name\="FolderChildCount"/>            </FieldRefs>            <XmlDocuments\>             <XmlDocument NamespaceURI\="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms"\>PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+TGlzdEZvcm08L0Rpc3BsYXk+PEVkaXQ+TGlzdEZvcm08L0VkaXQ+PE5ldz5MaXN0Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4=</XmlDocument>            </XmlDocuments>          </ContentType>          <ContentType ID\="0x010041CFF0FB0E8DF3488B80375585352211004905F4EC8EC2044B8FBDDDA5030A8A0D" Name\="Audit" Group\="EDRMS Portfolio DB" Description\="Create a new list item." ReadOnly\="TRUE" Version\="1"\>           <Folder TargetName\="Audit"/>            <FieldRefs\>             <FieldRef ID\="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Name\="ContentType"/>              <FieldRef ID\="{fa564e0f-0c70-4ab9-b863-0177e6ddd247}" Name\="Title" Required\="FALSE" Hidden\="FALSE" ShowInNewForm\="TRUE" ShowInEditForm\="TRUE" ReadOnly\="FALSE" PITarget\="" PrimaryPITarget\="" PIAttribute\="" PrimaryPIAttribute\="" Aggregation\="" Node\=""/>            </FieldRefs>            <XmlDocuments\>             <XmlDocument NamespaceURI\="Microsoft.SharePoint.Taxonomy.ContentTypeSync"\>PFNoYXJlZENvbnRlbnRUeXBlIHhtbG5zPSJNaWNyb3NvZnQuU2hhcmVQb2ludC5UYXhvbm9teS5Db250ZW50VHlwZVN5bmMiIFNvdXJjZUlkPSJkYWFhNjk3Ny02YzQwLTRmYjgtOGIwOC04NDk3YjcwN2JlMDgiIENvbnRlbnRUeXBlSWQ9IjB4MDEwMDQxQ0ZGMEZCMEU4REYzNDg4QjgwMzc1NTg1MzUyMjExIiBQcmV2aW91c1ZhbHVlPSJmYWxzZSIgLz4=</XmlDocument>              <XmlDocument NamespaceURI\="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms"\>PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+TGlzdEZvcm08L0Rpc3BsYXk+PEVkaXQ+TGlzdEZvcm08L0VkaXQ+PE5ldz5MaXN0Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4=</XmlDocument>            </XmlDocuments>          </ContentType>          <ContentType ID\="0x01005A3C6BBD0C84504FA03752D3919FCB5D00D6851EFE1788A74EB97B1705C72B89B7" Name\="Audit Action" Group\="EDRMS Portfolio DB" Description\="Create a new list item." ReadOnly\="TRUE" Version\="1"\>           <Folder TargetName\="Audit Action"/>            <FieldRefs\>             <FieldRef ID\="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Name\="ContentType"/>              <FieldRef ID\="{fa564e0f-0c70-4ab9-b863-0177e6ddd247}" Name\="Title" Required\="FALSE" Hidden\="FALSE" ShowInNewForm\="TRUE" ShowInEditForm\="TRUE" ReadOnly\="FALSE" PITarget\="" PrimaryPITarget\="" PIAttribute\="" PrimaryPIAttribute\="" Aggregation\="" Node\=""/>              <FieldRef ID\="{f4f3e946-d979-4ff5-8223-c8f01ebc3347}" Name\="EDRMSAction"/>              <FieldRef ID\="{e67580c2-9226-40bf-8f34-5973dba37397}" Name\="EDRMSActualCloseddate"/>              <FieldRef ID\="{11bfcac9-af7d-48dc-b893-37fe198ac105}" Name\="p1ddcb168b464625aa2588fc1e161f6c" Hidden\="TRUE"/>              <FieldRef ID\="{f3b0adf9-c1a2-4b02-920d-943fba4b3611}" Name\="TaxCatchAll" Hidden\="TRUE"/>              <FieldRef ID\="{8f6b6dd8-9357-4019-8172-966fcd502ed2}" Name\="TaxCatchAllLabel" Hidden\="TRUE"/>              <FieldRef ID\="{978a583c-0ee7-4423-8db5-d63860d6f814}" Name\="EDRMSAuditContact"/>              <FieldRef ID\="{30f91207-2921-489a-8d95-4aff394692b3}" Name\="EDRMSCurrentstatus"/>              <FieldRef ID\="{db74e757-efb1-4735-b300-e232027c891a}" Name\="EDRMSD\_TSpendValuerequested"/>            </FieldRefs>            <XmlDocuments\>             <XmlDocument NamespaceURI\="Microsoft.SharePoint.Taxonomy.ContentTypeSync"\>PFNoYXJlZENvbnRlbnRUeXBlIHhtbG5zPSJNaWNyb3NvZnQuU2hhcmVQb2ludC5UYXhvbm9teS5Db250ZW50VHlwZVN5bmMiIFNvdXJjZUlkPSJkYWFhNjk3Ny02YzQwLTRmYjgtOGIwOC04NDk3YjcwN2JlMDgiIENvbnRlbnRUeXBlSWQ9IjB4MDEwMDVBM0M2QkJEMEM4NDUwNEZBMDM3NTJEMzkxOUZDQjVEIiBQcmV2aW91c1ZhbHVlPSJmYWxzZSIgLz4=</XmlDocument>              <XmlDocument NamespaceURI\="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms"\>PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+TGlzdEZvcm08L0Rpc3BsYXk+PEVkaXQ+TGlzdEZvcm08L0VkaXQ+PE5ldz5MaXN0Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4=</XmlDocument>            </XmlDocuments>          </ContentType>          <ContentType ID\="0x010017AD3FB874451C4F8DD58309771F3D5800A446B1790750624282F9DFE39EF75A09" Name\="Audit Exception" Group\="EDRMS Portfolio DB" Description\="Create a new list item." ReadOnly\="TRUE" Version\="1"\>           <Folder TargetName\="Audit Exception"/>            <FieldRefs\>             <FieldRef ID\="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Name\="ContentType"/>              <FieldRef ID\="{fa564e0f-0c70-4ab9-b863-0177e6ddd247}" Name\="Title" Required\="FALSE" Hidden\="FALSE" ShowInNewForm\="TRUE" ShowInEditForm\="TRUE" ReadOnly\="FALSE" PITarget\="" PrimaryPITarget\="" PIAttribute\="" PrimaryPIAttribute\="" Aggregation\="" Node\=""/>            </FieldRefs>            <XmlDocuments\>             <XmlDocument NamespaceURI\="Microsoft.SharePoint.Taxonomy.ContentTypeSync"\>PFNoYXJlZENvbnRlbnRUeXBlIHhtbG5zPSJNaWNyb3NvZnQuU2hhcmVQb2ludC5UYXhvbm9teS5Db250ZW50VHlwZVN5bmMiIFNvdXJjZUlkPSJkYWFhNjk3Ny02YzQwLTRmYjgtOGIwOC04NDk3YjcwN2JlMDgiIENvbnRlbnRUeXBlSWQ9IjB4MDEwMDE3QUQzRkI4NzQ0NTFDNEY4REQ1ODMwOTc3MUYzRDU4IiBQcmV2aW91c1ZhbHVlPSJmYWxzZSIgLz4=</XmlDocument>              <XmlDocument NamespaceURI\="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms"\>PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+TGlzdEZvcm08L0Rpc3BsYXk+PEVkaXQ+TGlzdEZvcm08L0VkaXQ+PE5ldz5MaXN0Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4=</XmlDocument>            </XmlDocuments>          </ContentType>          <ContentType ID\="0x01001845946689C1004B9290F0B4BEBCB6E20072925977D27F5E42AC4CB9872A95396F" Name\="EDRMS Contacts" Group\="EDRMS Portfolio DB" Description\="Create a new list item." ReadOnly\="TRUE" Version\="1"\>           <Folder TargetName\="EDRMS Contacts"/>            <FieldRefs\>             <FieldRef ID\="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Name\="ContentType"/>              <FieldRef ID\="{fa564e0f-0c70-4ab9-b863-0177e6ddd247}" Name\="Title" Required\="FALSE" Hidden\="FALSE" ShowInNewForm\="TRUE" ShowInEditForm\="TRUE" ReadOnly\="FALSE" PITarget\="" PrimaryPITarget\="" PIAttribute\="" PrimaryPIAttribute\="" Aggregation\="" Node\=""/>            </FieldRefs>            <XmlDocuments\>             <XmlDocument NamespaceURI\="Microsoft.SharePoint.Taxonomy.ContentTypeSync"\>PFNoYXJlZENvbnRlbnRUeXBlIHhtbG5zPSJNaWNyb3NvZnQuU2hhcmVQb2ludC5UYXhvbm9teS5Db250ZW50VHlwZVN5bmMiIFNvdXJjZUlkPSJkYWFhNjk3Ny02YzQwLTRmYjgtOGIwOC04NDk3YjcwN2JlMDgiIENvbnRlbnRUeXBlSWQ9IjB4MDEwMDE4NDU5NDY2ODlDMTAwNEI5MjkwRjBCNEJFQkNCNkUyIiBQcmV2aW91c1ZhbHVlPSJmYWxzZSIgLz4=</XmlDocument>              <XmlDocument NamespaceURI\="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms"\>PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+TGlzdEZvcm08L0Rpc3BsYXk+PEVkaXQ+TGlzdEZvcm08L0VkaXQ+PE5ldz5MaXN0Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4=</XmlDocument>            </XmlDocuments>          </ContentType>          <ContentType ID\="0x010030AD093A85649846B59710F4D05342F80049E722FACD80644EBE748BF114D659F4" Name\="EDRMS Item" Group\="EDRMS Portfolio DB" Description\="Create a new list item." ReadOnly\="TRUE" Version\="1"\>           <Folder TargetName\="EDRMS Item"/>            <FieldRefs\>             <FieldRef ID\="{c042a256-787d-4a6f-8a8a-cf6ab767f12d}" Name\="ContentType"/>              <FieldRef ID\="{fa564e0f-0c70-4ab9-b863-0177e6ddd247}" Name\="Title" Required\="FALSE" Hidden\="FALSE" ShowInNewForm\="TRUE" ShowInEditForm\="TRUE" ReadOnly\="FALSE" PITarget\="" PrimaryPITarget\="" PIAttribute\="" PrimaryPIAttribute\="" Aggregation\="" Node\=""/>            </FieldRefs>            <XmlDocuments\>             <XmlDocument NamespaceURI\="Microsoft.SharePoint.Taxonomy.ContentTypeSync"\>PFNoYXJlZENvbnRlbnRUeXBlIHhtbG5zPSJNaWNyb3NvZnQuU2hhcmVQb2ludC5UYXhvbm9teS5Db250ZW50VHlwZVN5bmMiIFNvdXJjZUlkPSJkYWFhNjk3Ny02YzQwLTRmYjgtOGIwOC04NDk3YjcwN2JlMDgiIENvbnRlbnRUeXBlSWQ9IjB4MDEwMDMwQUQwOTNBODU2NDk4NDZCNTk3MTBGNEQwNTM0MkY4IiBQcmV2aW91c1ZhbHVlPSJmYWxzZSIgLz4=</XmlDocument>              <XmlDocument NamespaceURI\="http://schemas.microsoft.com/sharepoint/v3/contenttype/forms"\>PEZvcm1UZW1wbGF0ZXMgeG1sbnM9Imh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vc2hhcmVwb2ludC92My9jb250ZW50dHlwZS9mb3JtcyI+PERpc3BsYXk+TGlzdEZvcm08L0Rpc3BsYXk+PEVkaXQ+TGlzdEZvcm08L0VkaXQ+PE5ldz5MaXN0Rm9ybTwvTmV3PjwvRm9ybVRlbXBsYXRlcz4=</XmlDocument>            </XmlDocuments>          </ContentType>        </ContentTypes>        <Forms\>         <Form Type\="DisplayForm" Name\="{E6BC58D2-03C9-443E-A9E9-1457A0EE9170}" Url\="Lists/Test1WithMultipleContentTypes/DispForm.aspx" Default\="TRUE" FormID\="0"/>          <Form Type\="EditForm" Name\="{88B97F4D-A183-4D27-8360-37CB895D7412}" Url\="Lists/Test1WithMultipleContentTypes/EditForm.aspx" Default\="TRUE" FormID\="0"/>          <Form Type\="NewForm" Name\="{4373EC06-A519-496D-8712-E116E0E8D3BE}" Url\="Lists/Test1WithMultipleContentTypes/NewForm.aspx" Default\="TRUE" FormID\="0"/>        </Forms>        <Security\>         <ReadSecurity\>1</ReadSecurity>          <WriteSecurity\>1</WriteSecurity>          <SchemaSecurity\>0</SchemaSecurity>        </Security>      </MetaData>    </List>  </Lists>