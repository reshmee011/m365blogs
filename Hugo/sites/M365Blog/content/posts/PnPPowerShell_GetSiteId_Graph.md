

```powershell
$url = "https://contoso-admin.sharepoint.com"
Connect-PnPOnline -url $url -interactive
$RestMethodUrl = "v1.0/sites/contoso.sharepoint.com:/teams/app-m365?$select=id"
$site = (Invoke-PnPGraphMethod -Url $RestMethodUrl -Method Get -ConsistencyLevelEventual)
write-host  $site.id
```
Sample output 
```powershell
contoso.sharepoint.com,18440dad-4514-4946-bcb5-ba2f7e033ed1,dbfeb5ba-2bf1-4ff8-bf61-af1d984fceb1
```

## References

[How to find {sitesId} to pass to MS Graph API call?](https://pankajsurti.com/2021/09/27/how-to-find-sitesid-to-pass-to-ms-graph-api-call/)