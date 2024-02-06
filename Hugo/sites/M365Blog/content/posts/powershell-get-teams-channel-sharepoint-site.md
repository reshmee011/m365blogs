---
title: "Get SharePoint site Url for channel"
date: 2024-01-20T06:19:42Z
tags: ["SharePoint","Teams Site","Private Channels" ,"Shared Channels", "Microsoft Graph", "Invoke-PnPGraphMethod"]
featured_image: '/posts/images/powershell-get-teams-channel-sharepoint-site/ChannelSharePointUrl.png'
draft: true
---

# Get SharePoint site Url for channel 

Ever wondered how to reconsruct the sharepoint site url for a private channel and shared channel knowing the channel name. 

for Private Channel
Parent site url followed by the channel relative ul 

For shared channel 
Parent site m365 group name followed by the channel relative url 


```PowerShell
$url = "https://contoso.sharepoint.com/sites/test"
Connect-PnPOnline -url $url -interactive
$m365GroupId = (get-pnpsite -include RelatedGroupId).RelatedGroupId.Guid
 
$RestMethodUrl = "beta/teams/$m365GroupId/channels"
 
$channels = (Invoke-PnPGraphMethod -Url $RestMethodUrl -Method Get -ConsistencyLevelEventual).value
 
$channels | ForEach-Object {
  write-host  ($_.filesfolderweburl).split("Shared Documents")[0]
}
```

```PowerShell
function get-channelsiteurl ($m365GroupId, $channelName){
  $channel= get-pnpteamschannel -team $m365GroupId -Identity $channelName
  $RestMethodUrl = "beta/teams/$m365GroupId/channels/$($channel.Id)"        
  $channel = Invoke-PnPGraphMethod -Url $RestMethodUrl -Method Get -ConsistencyLevelEventual
  return  ($channel.filesfolderweburl).split("Shared Documents")[0]
}
 
$url = "https://contoso.sharepoint.com/teams/d-TEAM-DemoTeam"
Connect-PnPOnline -url $url -interactive
$m365GroupId = (get-pnpsite -include RelatedGroupId).RelatedGroupId.Guid
 
get-channelsiteurl  $m365GroupId "Ash-Test"
 
```

##References

[Get channel](https://learn.microsoft.com/en-us/graph/api/channel-get?wt.mc_id=MVP_308367)
[List channels](https://learn.microsoft.com/en-us/graph/api/channel-list?view=graph-rest-beta&tabs=http)