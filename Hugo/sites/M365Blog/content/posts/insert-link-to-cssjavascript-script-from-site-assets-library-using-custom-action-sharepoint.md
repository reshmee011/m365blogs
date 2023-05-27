---
title: 'Insert link to CSS/JavaScript script from "Site Assets" library using custom action SharePoint'
date: Tue, 05 Jan 2016 23:27:38 +0000
draft: false
tags: ['JavaScript', 'SharePoint', 'SharePoint 2013', 'SharePoint CustomAction Css JavaScript XML', 'Uncategorized']
---

I had a requirement from a client to apply custom css and JavaScript to a site collection including sub sites. The JavaScript file and Css file can be uploaded to "Site Assets" library using Module Element in SharePoint solution Module element to deploy CSS file <Elements xmlns\="http://schemas.microsoft.com/sharepoint/"\> <Module Name\="CustomCss" Url\="SiteAssets"\> <File Path\="CustomCss\\custom.css" Url\="CustomCss\\custom.css" /> </Module\> </Elements\> Module element to deploy CSS file <Elements xmlns\="http://schemas.microsoft.com/sharepoint/"\> <Module Name\="Jquery" Url\="SiteAssets"\> <File Path\="Jquery\\jquery-1.11.1.min.js" Url\="Jquery/jquery-1.11.1.min.js" /> </Module\> </Elements\> There are two options how to add the reference o the js file and css file

1.  The masterpage file can be modified to insert links to the JavaScript and css files.
2.  Use [customaction](https://msdn.microsoft.com/en-us/library/office/ms460194.aspx) to insert a link

I chose the second option which can be deployed as a sandboxed solution. To include a reference to  JavaScript in all pages in a SharePoint site from "Site Assets" library the customAction element is used with the ScriprSrc attribute. <?xml version\="1.0" encoding\="utf-8"?> <Elements xmlns\="http://schemas.microsoft.com/sharepoint/"\> <CustomAction ScriptSrc\="~SiteCollection/SiteAssets/jquery/jquery-1.11.1.min.js" Location\="ScriptLink" Sequence\="100"/\> </Elements\> To apply a Custom CSS deployed to "Site Assets " library to a site the custom action element is used with ScriptBlock tag. The ScriptBlock generates the JavaScript code on the page which adds the reference to the css file when executed. <?xml version\="1.0" encoding\="utf-8"?> <Elements xmlns\="http://schemas.microsoft.com/sharepoint/"\> <CustomAction Id\="CssSiteCol" Location\="ScriptLink" ScriptBlock\="document.write('&lt;link rel=&quot;stylesheet&quot; After=&quot;Corev15.css&quot; type=&quot;text/css&quot; href=&quot;/PWA/SiteAssets/CustomCss/custom.css&quot;&gt;&lt;/' + 'link&gt;');" Sequence\="202" /> </Elements\>