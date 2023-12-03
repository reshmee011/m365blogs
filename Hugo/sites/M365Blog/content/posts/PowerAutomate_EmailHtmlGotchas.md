---
title: "Optimizing Email HTML for Outlook"
date: 2023-11-29T16:15:32Z
tags: ["Power Automate","Email Action","HTML"]
featured_image: '/posts/images/PowerAutomate_EmailHtmlGotchas/div_email_owa.PNG'
draft: false
---

# Optimizing Email HTML for Outlook using the 'Send Email' action within Power Automate 

When crafting attention-grabbing emails, the HTML structure plays a pivotal role. I recently encountered challenges in achieving consistent formatting across Outlook desktop and web versions using the **Send Email** action within Power Automate.

![Email Action](../images/PowerAutomate_EmailHtmlGotchas/EmailAction.png)

## Using div Tags for Responsiveness

At first, I experimented with divs to ensure responsiveness and modern HTML formatting. The code appeared as follows:

```html
<style>
.center {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.box {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 20px;
  border: 1px solid black;
  border-radius: 5px;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
}

a {
  text-decoration: none;
  color: white;
  background-color: #4CAF50;
  padding: 10px 20px;
  border-radius: 5px;
  transition: background-color 0.3s ease;
}

a:hover {
  background-color: #3e8e41;
}
img
{
 width:200px;
 height:100px;
 object-fit:cover;
}
</style>
</head>
<body>

<div class="center">
  <div class="box">
    <h1>Heading</h1>
    <img src="https://media.akamai.odsp.cdn.office.net/ukwest1-mediap.svc.ms/transform/thumbnail?provider=url&inputFormat=jpg&docid=https%3A%2F%2Fcdn.hubblecontent.osi.office.net%2Fm365content%2Fpublish%2F005292d6-9dcc-4fc5-b50b-b2d0383a411b%2Fimage.jpg&w=400"></img>
    <a href="https://www.google.com">Go to item</a>
  </div>
</div>

```
While this worked well in **Outlook on the Web**, rendering beautifully as expected:
![div rendered in OWA](../images/PowerAutomate_EmailHtmlGotchas/div_email_owa.PNG)

The same layout, unfortunately, didn’t maintain its formatting in **Outlook Desktop**:
![div not maintained in OWA](../images/PowerAutomate_EmailHtmlGotchas/div_email_outlookdesktop.PNG)

## Table HTML Structure for consistency 

To ensure a consistent display, especially in Outlook Desktop, I pivoted towards utilizing a table structure with CSS:

```html
<style>
h1 {
  text-align: center;
  margin-bottom: 20px;
}

a {
  text-decoration: none;
  color: white;
  background-color: #357EC7;
  padding:10px 10px 20px 10px;
  border-radius: 5px;
  transition: background-color 0.3s ease;
}
body {
display:flex;
justify-content:center;
align-items: center;
height:100vh;
}
table {
background-color:white;
max-width:500pt;
border:1pt solid #C8C8C8;
margin:auto;
border-collapse:collapse;
}
td,th {
 text-align:center;
 vertical-align:middle;
 padding:10px 10px;
}
img{
}
</style>
<body>
<table>
<tbody>
<tr style="height:11pt;"> 
<td> <h1>Heading </h1></td>

</tr>
<tr><td><img data-imagetype="External" src="https://cdn.hubblecontent.osi.office.net/m365content/publish/aa2dc6cd-0710-4b94-b49d-3464108dd918/905395820.jpg" border="0" width="400" height="200"></td>
</tr>
<tr><td>
<h2>@{outputs('Get_details_for_selected_item')?['body/AnnouncementTitle']}</h2></td></tr>
<tr style="height:11pt;text-align: center;"> 
<td style="padding:40px">  <h3> <a href="https://www.google.com">Go to item</a></h3></td>
</tr>
</tbody>
</table>
</body>
```

This adjustment resulted in better consistency across both Outlook on the Web and Outlook Desktop:

Email with picture within outlook on the web
![table rendered in OWA](../images/PowerAutomate_EmailHtmlGotchas/div_email_owa.PNG)

Email with picture within outlook on the desktop
![table rendered in desktop](../images/PowerAutomate_EmailHtmlGotchas/table_email_outlookdesktop.PNG)

## Addressing Image Properties

Attempting to set image properties via CSS, like width and height, didn't consistently render. 

### Using CSS

```html
<style>
img{
    width:400px; 
    height:200px;
}
</style>
```

### Applying properties directly to the image HTML:

Using width and height properties within the img HTML tag helped to achieve more consistent rendering.

```html
<img data-imagetype="External" src="https://cdn.hubblecontent.osi.office.net/m365content/publish/aa2dc6cd-0710-4b94-b49d-3464108dd918/905395820.jpg" border="0" id="x__x0000_i1025" width="400" height="200">
```

## Conclusion

While tables offered a more stable layout for Outlook emails, I'm still on the lookout for more effective HTML solutions. If you’ve cracked this or have alternative suggestions, I’d love to explore further!