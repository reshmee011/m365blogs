---
title: "Retrieving SharePoint Site URL for Teams Channels"
date: 2024-04-21T06:19:42Z
tags: ["SharePoint","Teams","Graph","Private Channels" ,"Shared Channels", "Microsoft Graph", "Invoke-PnPGraphMethod"]
featured_image: '/posts/images/powershell-get-teams-channel-sharepoint-site/ChannelSharePointUrl.png'
omit_header_text: true
draft: false
---

# Retrieving SharePoint Site URL for Teams Channels

Have you ever needed to construct the SharePoint site URL for a private or shared channel using just the channel name? This post will guide you through the process.

## Private Channels

The default SharePoint URL for a private channel follows the format: `<ParentSharePointSiteUrl><ChannelRelativeUrl>`. Note that if the channel is renamed, the corresponding SharePoint URL remains unchanged.

![Private Channel Renamed](../images/powershell-get-teams-channel-sharepoint-site/PrivateChannelRenamed.png)

## Shared Channels 

Similarly, the default SharePoint URL for a shared channel follows the format: `<ParentM365GroupName><ChannelRelativeUrl>`. Again, renaming the channel does not alter the SharePoint URL.

![Shared Channel Rename](../images/powershell-get-teams-channel-sharepoint-site/SharedChannelRenamed.png)

At times, you may need to access the underlying SharePoint object representing the Channel folder in SharePoint for specific operations. You can retrieve the SharePoint URL of the channel by querying the Graph API at `beta/teams/$m365GroupId/channels/$($channel.Id)`.

## PowerShell Script to Retrieve All Teams Channel SharePoint Site URLs Linked to M365 Connected Teams Site

```PowerShell
$m365SiteUrl = "https://contoso.sharepoint.com/sites/test"
Connect-PnPOnline -url $url -interactive
$m365GroupId = (get-pnpsite -include RelatedGroupId).RelatedGroupId.Guid
 
$RestMethodUrl = "beta/teams/$m365GroupId/channels"
 
$channels = (Invoke-PnPGraphMethod -Url $RestMethodUrl -Method Get -ConsistencyLevelEventual).value
 
$channels | ForEach-Object {
  write-host  ($_.filesfolderweburl).split("Shared Documents")[0]
}
```
![list of channel SharePoint Urls](../images/powershell-get-teams-channel-sharepoint-site/ChannelSharePointUrl.png)

## PowerShell Script to Retrieve SharePoint Site URL for a Specific Teams Channel

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
 
get-channelsiteurl $m365GroupId "Ash-Test"
```

## Conclusion

The Graph API can be used to access Teams Channels and SharePoint folders. Regardless of whether the channel has been renamed or is a private channel, you can always retrieve the URL to the folder in SharePoint using the endpoint `beta/teams/$m365GroupId/channels/$($channel.Id).`

## References

[Get channel](https://learn.microsoft.com/en-us/graph/api/channel-get?wt.mc_id=MVP_308367)

[List channels](https://learn.microsoft.com/en-us/graph/api/channel-list?view=graph-rest-beta&tabs=http&wt.mc_id=MVP_308367)

[Channel resource type](https://learn.microsoft.com/en-us/graph/api/resources/channel?view=graph-rest-beta&wt.mc_id=MVP_308367)

[Get a Teams channel SharePoint Url using Graph API](https://colinjwood.wordpress.com/2020/02/18/get-a-teams-channel-sharepoint-url-using-graph-api/)
