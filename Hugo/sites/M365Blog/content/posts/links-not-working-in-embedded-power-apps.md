---
title: "Troubleshooting Blocked Links to SharePoint Pages in Embedded Power Apps"
date: 2023-07-04T11:04:28+01:00
tags: ["Embed, Power Apps","Links to SharePoint Pages"]
draft: false
---


# Troubleshooting Blocked Links to SharePoint Pages in Embedded Power Apps

When embedding Power Apps using the "Microsoft Power Apps web part," you may encounter an issue where links to SharePoint pages are blocked. This issue does not affect links to Power Apps themselves.
![Microsoft Power Apps](../images/links-not-working-in-embedded-power-apps/Link-to-SP-Pages-blocked.png)

The error message you might encounter is as follows:

"Unsafe attempt to initiate navigation for frame with origin 'domain.sharepoint.com' from frame with URL 'https://pa-static-ms.azureegde.net/resource/webplayerdynamic/publishedapp/...'.The frame attempting navigation of the top-level window is sandboxed, but the flag of 'allow-top-navigation' or 'allow-top-navigation-by-user-activation' is not set."

It's important to note that the Microsoft Power Apps web part is still in preview.

![Microsoft Power Apps](../images/links-not-working-in-embedded-power-apps/Microsoft-PowerApps-Preview.png)

This web part enables you to embed Power Apps into modern SharePoint pages by referring to the app ID. To obtain the app ID, you can navigate to the Power Apps portal, select an app, click on "Details," and extract the web link.

![PowerApps ID](../images/links-not-working-in-embedded-power-apps/PowerApps-ID.png)

## Solutions

1. Use the embed web part: This method adds the app as an iframe content. However, it's worth noting that the loading experience using the embed web part may be slower, sometimes taking up to 10 seconds, which might not be acceptable in certain scenarios.

```html
<iframe width="1024px" height="768px" src="https://web.powerapps.com/webplayer/iframeapp?source=iframe&amp;screenColor=rgba(104,101,171,1)&amp;appId=/providers/Microsoft.PowerApps/apps/<appID>&amp;ItemId="></iframe>
```  

![Embed Webpart](../images/links-not-working-in-embedded-power-apps/Microsoft-PowerApps-Preview.png)

2. HTML control with the img tag: Links to SharePoint pages within the HTMLText control are not blocked when used within the Microsoft Power Apps web part.

By implementing these solutions, you can troubleshoot and overcome the issue of blocked links in embedded Power Apps.

```html
"<span style='cursor:pointer; color:#503291;style=font-family:verdana'><b><a href='"&  varSiteUrl & "/SitePages/Page-Discover.aspx' target='_blank'><img width=100% , height= 100% src='" & varSiteUrl & "/SiteAssets/PowerApps_Images/Dashboard/stepDiscover.png'><img></span>"
```