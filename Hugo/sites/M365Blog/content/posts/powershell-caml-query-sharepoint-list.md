
```PowerShell
$SiteCollectionUrl = Read-Host -Prompt "Enter site collection URL: ";
$csvPath = Read-Host -Prompt "Enter timesheet csv path ";
$list = "Timesheet"
 
connect-pnponline -url $SiteCollectionUrl -Interactive
$ids = [System.Collections.ArrayList]@();
if(Test-Path $csvPath){
  Import-Csv -Path $csvPath | foreach-object{
$row = $_
    $q = @"
<View>
    <Query>
        <Where>
           <Eq><FieldRef Name='Category'/><Value Type='Text'>$($row.OldValue)</Value></Eq>
        </Where>
    </Query>
    <ViewFields>
        <FieldRef Name='Id' />
        <FieldRef Name='Category' />
    </ViewFields>
</View>
"@
 
$items = Get-PnPListItem -Connection $Connection -List $list -Query $q  | Select-Object Id, Category
$items | ForEach-Object{
   # $global:Results += New-Object PSObject -property $([ordered]@{
$ids.Add([PSCustomObject]@{
    Id = $_.Id
    OldValue = $row.OldValue
    NewValue = $row.NewValue
}) | out-null;
 
   Set-PnPListItem -List $list -Identity $_.Id -Values @{$Category= $row.NewValue;} -UpdateType SystemUpdate
  }
 
  }
 } 
 
 
$ids | Export-Csv -Path "C:\temp\timesheet_id_toupdate.csv" -NoTypeInformation -Force -Delimiter "|"
```