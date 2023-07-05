---
title: "Embed Single Instagram Post Into Modern SharePoint Page"
date: 2023-07-04T14:49:19+01:00
tags: ["Embed, Instagram","SharePoint Page"]
draft: true
---

# Embed Single Instagram Post Into Modern SharePoint Page

I was investigating on how to display an instagram post within a Modern SharePoint News Page.

The generic embed code from instagram uses the script tag which is not allowed with SharePoint Page.  

1.Navigate to the instagram post, e.g. https://www.instagram.com/p/CqN3lWpNMpI/ and click on ellipsis. 
2. Click on embed post.
![Embed Post](../images/Embed-Single-Instagram-Post-Into-Modern-SharePoint-Page/Embed_Post.png)
3. Copy the embed code.
![Copy Embed Code](../images/Embed-Single-Instagram-Post-Into-Modern-SharePoint-Page/CopyEmbedCode.png)
4. Paste the embed code

Once pasted within the embed code , the error message that script is not supported appears

![Script Not Supported](../images/Embed-Single-Instagram-Post-Into-Modern-SharePoint-Page/ScriptNotSupported.png.png)

I found the solution from an image from a webpart,[Introducing Web Embedding Instagram Content on Websites](https://about.instagram.com/blog/announcements/introducing-web-embedding-instagram-content-on-websites) the solution is simply to append the embed at the post https://www.instagram.com/p/CqN3lWpNMpI/embed to include within the Iframe tag

To whitelist the instagram domain, add www.instagram.com in HTML Field Security in site settings, follow the [Allow or restrict the ability to embed content on SharePoint pages](https://support.microsoft.com/en-us/office/allow-or-restrict-the-ability-to-embed-content-on-sharepoint-pages-e7baf83f-09d0-4bd1-9058-4aa483ee137b?ui=en-us&rs=en-gb&ad=gb)

![HTMLSecurity](../images/HTMLSecurity.png)

To embed an individual post , use the following embed code below specifying scrolling and frameborder properties. Replace <postRef> with instagram post reference.

```
<iframe width="724" height="724" src="https://www.instagram.com/p/<postRef>/embed" player"="" frameborder="0" scrolling="no"></iframe>
```

![Embed Instagram post within Page](../images/Embed-Single-Instagram-Post-Into-Modern-SharePoint-Page/Embed_InstagramPost.png)

To embed an individual instagram account , use the following embed code below specifying scrolling and frameborder properties. Replace <accountname> with instagram accountname.

```html
<iframe width="1024px" height="1200px" src="https://www.instagram.com/<accoutname>/embed" scrolling="no" frameborder="0"></iframe>
```
![Embed Instagram account within Page](../images/Embed-Single-Instagram-Post-Into-Modern-SharePoint-Page/Embed_InstagramAccount.png)



