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

Sometimes it is necessary to access the underlying SharePoint object that represents the Channel folder in SharePoint to perform specific SharePoint operations. You can get the SharePoint url of the Team by accessing the Graph API at

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

The webUrl property stores the url of the folder in SharePoint. This is also very useful for navigating to Private channels in SharePoint which are hosted in their own dedicated site collection just for hosting the private channel folder.

This nows gives us API access to Teams Channels and SharePoint folders using the Graph API so now regardless of whether the channel has been renamed or is a Private channel, you can always get the Url to the folder in SharePoint. [channel resource type](https://learn.microsoft.com/en-us/graph/api/resources/channel?view=graph-rest-beta?wt.mc_id=MVP_308367)

## References

[Get channel](https://learn.microsoft.com/en-us/graph/api/channel-get?wt.mc_id=MVP_308367)
[List channels](https://learn.microsoft.com/en-us/graph/api/channel-list?view=graph-rest-beta&tabs=http&wt.mc_id=MVP_308367)
[channel resource type](https://learn.microsoft.com/en-us/graph/api/resources/channel?view=graph-rest-beta?wt.mc_id=MVP_308367)
[Get a Teams channel SharePoint Url using Graph API](https://colinjwood.wordpress.com/2020/02/18/get-a-teams-channel-sharepoint-url-using-graph-api/)
