---
title: "Executing a Flow from Column Formatting from a page using the Embed Webpart"
date: 2023-12-08T15:01:42Z
tags: ["List formatting","JSON","HTML","CSS", "Embed WebPart","Trigger Flow"]
featured_image: '/posts/images/ListFormatting_StartFlowFromEmbedWebPart/EmbedWebPartForFlow.PNG'
draft: false
---

# Executing a Flow from Column Formatting from a page using the Embed Webpart

I encountered an intriguing challenge: triggering a flow from column formatting within a List View Webpart. Initially, I successfully created a button within a list to execute a flow on a specific item, following the steps outlined in Microsoft's documentation [here](https://docs.microsoft.com/en-us/sharepoint/dev/declarative-customization/column-formatting#create-a-button-to-launch-a-flow). It worked seamlessly within the list interface directly.

However, when attempting to add the list with the formatting column using the list view webpart on a page to allow end-users to trigger the flow the results were different. The button failed to execute the flow within the webpart, leading to an investigation into the underlying causes.

It became apparent that the infrastructure disparity between the actual list and the list webparts explained the discrepancy. Notably, the List View webpart lacks support for this specific feature, hindering its ability to trigger the flow.

Seeking a solution, I discovered an alternative approach using the Embed Webpart. By leveraging the Embed Webpart, as elucidated in [this insightful article by Jennifer Hersko](https://jennyssharepointtips.wordpress.com/2022/05/02/embed-a-library-or-list-from-another-site-collection-on-a-site-page/), it became feasible to display a list or library. Utilizing HTML code within the webpart, such as the snippet below, allows embedding a list or library (replace URL with your information): 

```html
<iframe width="402" height="346" frameborder="0" scrolling="no" src="https://YOURURL"></iframe>
```

![Email Action](../images/PowerAutomate_EmailHtmlGotchas/EmailAction.png)

## References

[Embed a library or list – from another site collection! – on a Site Page](https://jennyssharepointtips.wordpress.com/2022/05/02/embed-a-library-or-list-from-another-site-collection-on-a-site-page/)

[Triggering power automate flow from a list embeded... - Power Platform Community (microsoft.com)](https://docs.microsoft.com/en-us/sharepoint/dev/declarative-customization/column-formatting#create-a-button-to-launch-a-flow)

[Column formatting not showing when view shown in a List Web Part (Modern Page and List) - Page 2 - Microsoft Community Hub](https://techcommunity.microsoft.com/t5/sharepoint-developer/column-formatting-not-showing-when-view-shown-in-a-list-web-part/m-p/161460/page/2)

[Format list view to customize SharePoint | Microsoft Learn](https://learn.microsoft.com/en-us/sharepoint/dev/declarative-customization/view-list-formatting)