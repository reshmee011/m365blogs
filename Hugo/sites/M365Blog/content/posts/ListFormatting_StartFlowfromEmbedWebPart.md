---
title: "Execute a flow from column formatting within Embed Webpart"
date: 2023-12-08T15:01:42Z
tags: ["Power Automate","Email Action","Column Formatting","Embed WebPart"]
featured_image: '/posts/images/PowerAutomate_EmailHtmlGotchas/div_email_owa.PNG'
draft: true
---

## Execute a flow from column formatting within Embed Webpart

I have followed the instructions Creating "https://docs.microsoft.com/en-us/sharepoint/dev/declarative-customization/column-formatting#create-a-button-to-launch-a-flow" 
 to create a button in a list to run a flow on a specific list item, and this works perfectly within the list.  I have used the list view webpart to display the list on a page to allow end user to trigger the flow however it is not working. This doesn't seem to be possible as the button does nothing when clicked on the web part.  
 The infrastructure for the actual list and the list web parts are different explaining the difference. List View webpart does not support that feature. 

The embed web part https://jennyssharepointtips.wordpress.com/2022/05/02/embed-a-library-or-list-from-another-site-collection-on-a-site-page/ can be used to display a list or library in an Embed web part, add this code to the web part (replace URL with your information): 

```html
<iframe width="402" height="346" frameborder="0" scrolling="no" src="https://YOURURL"></iframe>
```

![Email Action](../images/PowerAutomate_EmailHtmlGotchas/EmailAction.png)

## References
https://jennyssharepointtips.wordpress.com/2022/05/02/embed-a-library-or-list-from-another-site-collection-on-a-site-page/
Triggering power automate flow from a list embeded... - Power Platform Community (microsoft.com)
https://docs.microsoft.com/en-us/sharepoint/dev/declarative-customization/column-formatting#create-a-button-to-launch-a-flow
