---
title: "Links not Working in Embedded Power Apps"
date: 2023-07-04T11:04:28+01:00
draft: true
---


# Links Not Working in Embedded Power Apps

Links to SharePoint pages within PowerApps embedded using the "Microsoft Power Apps webpart" seem to be blocked. However links to power apps work. 
![Microsoft Power Apps](../images/Link-to-SP-Pages-blocked.png)

"Unsafe attempt to initiate navigation for frame with origin 'domain.sharepoint.com' from frame with URL 'https://pa-static-ms.azureegde.net/resource/webplayerdynamic/publishedapp/...'.The frame attempting navigation of the top-level window is sandboxed, but the flag of 'allow-top-navigation' or 'allow-top-navigation-by-user-activation' is not set."

Microsoft Power Apps webpart is still in preview. 

![Microsoft Power Apps](../images/links-not-working-in-embedded-power-apps/Microsoft-PowerApps-Preview.png)

It allows to embed Power Apps into modern SharePoint pages referring the app Id. The app link Id can be obtained by navigating to the power apps portal, choosing an app and click on details and extract the web link.

![PowerApps ID](../images/links-not-working-in-embedded-power-apps/PowerApps-ID.png)

## Solutions

1. Use the embed webpart which adds the app as an iframe content. However the experience of using embed webpart is different taking longer at times 10secs to load which was unacceptable. 

```html
<iframe width="1024px" height="768px" src="https://web.powerapps.com/webplayer/iframeapp?source=iframe&amp;screenColor=rgba(104,101,171,1)&amp;appId=/providers/Microsoft.PowerApps/apps/<appID>&amp;ItemId="></iframe>
```  

![Embed Webpart](../images/links-not-working-in-embedded-power-apps/Microsoft-PowerApps-Preview.png)

2. HTML control with the img tag

The link to the SharePoint Page within the HTMLText control was not blocked within the Microsoft Power Apps webpart webpart 

```html
"<span style='cursor:pointer; color:#503291;style=font-family:verdana'><b><a href='"&  varSiteUrl & "/SitePages/Page-Discover.aspx' target='_blank'><img width=100% , height= 100% src='" & varSiteUrl & "/SiteAssets/PowerApps_Images/Dashboard/stepDiscover.png'><img></span>"
```