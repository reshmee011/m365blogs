#Parameters
$siteUrl = Read-Host -Prompt "Enter site collection URL";
$dateTime = (Get-Date).toString("dd-MM-yyyy-hh-ss")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "SharingLinkDeletionReport-" + $dateTime + ".csv"
$ReportOutput = $directorypath + "\Logs\"+ $fileName

$global:Results = @();

#Exclude certain libraries
$ExcludedLists = @("Form Templates", "Preservation Hold Library", "Site Assets", "Images", "Pages", "Settings", "Videos","Timesheet"
    "Site Collection Documents", "Site Collection Images", "Style Library", "AppPages", "Apps for SharePoint", "Apps for Office")

Function QueryDeleteSharingLinksByObject($_web,$_object,$_type,$_relativeUrl,$_siteUrl,$_siteTitle,$_listTitle)
{
  $roleAssignments = Get-PnPProperty -ClientObject $_object -Property RoleAssignments
  
  foreach($roleAssign in $roleAssignments){
      Get-PnPProperty -ClientObject $roleAssign -Property RoleDefinitionBindings,Member;
      #Sharing link is in the format SharingLinks.03012675-2057-4d1d-91e0-8e3b176edd94.OrganizationView.20d346d3-d359-453b-900c-633c1551ccaa
    If ($roleAssign.Member.Title -like "SharingLinks*")
      {
        $global:Results += New-Object PSObject -property $([ordered]@{
            object= $_object.Title
            type = $_type          
            relativeURL = $_relativeURL
            siteUrl = $_siteUrl 
            siteTitle = $_siteTitle
            listTitle = $_listTitle 
            sharinglink = $roleAssign.Member.Title
        })
           #$object.RoleAssignments.GetPrincipalId($roleAssign.PrincipalId).Delete
       #Invoke-PnPQuery
       remove-pnpgroup -identity $roleAssign.Member.Title -force
       
    }
 # return $_sharingLinks;
   }
}

  
Connect-PnPOnline -Url $siteUrl -Interactive

$web= Get-PnPWeb

Write-Host "Processing site $siteUrl"  -Foregroundcolor "Red"; 

$ll = Get-PnPList -Includes BaseType, Hidden, Title,HasUniqueRoleAssignments,RootFolder | Where-Object {$_.Hidden -eq $False -and $_.Title -notin $ExcludedLists } #$_.BaseType -eq "DocumentLibrary" 
  Write-Host "Number of lists $($ll.Count)";

  foreach($list in $ll)
  {
    $listUrl = $list.RootFolder.ServerRelativeUrl;       
    $listTitle = $list.Title; 
    #Get all list items in batches
    $ListItems = Get-PnPListItem -List $list -PageSize 2000 
        #Iterate through each list item
        ForEach($item in $ListItems)
        {
            $ItemCount = $ListItems.Count
            #Check if the Item has unique permissions
            $HasUniquePermissions = Get-PnPProperty -ClientObject $Item -Property "HasUniqueRoleAssignments"
            If($HasUniquePermissions)
            {       
                #Get Shared Links
                if($list.BaseType -eq "DocumentLibrary")
                {
                    $type= "File";
                    $fileUrl = $item.FieldValues.FileRef;
                }
                else
                {
                    $type= "Item";
                    $fileUrl = "$siteurl/lists/$listTitle/AllItems.aspx?FilterField1=ID&FilterValue1=$($item.id)"
                }
                QueryDeleteSharingLinksByObject $web $item $Type $fileUrl $siteUrl $web.Title $listTitle;
            }
        }
    }
 
  $global:Results | Export-CSV $ReportOutput -NoTypeInformation
Write-host -f Green "Sharing Links for user generated Successfully!"