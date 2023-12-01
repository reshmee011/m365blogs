---
title: "PowerAutomate_EmailHtmlGotchas"
date: 2023-11-29T16:15:32Z
draft: true
---

# Email HTML in Outlook

Want a nicely formatted post to attract recipient's attention so that it's not just another email and tried to use div to render the email for responsiveness and what modern html should be formatted.

You tried something like

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
Looks fine on email opened in browser 


but in Outlook destop all formatting is gone 

which renders well when you tried on html editor

However in Outlook desktop all formatting is not rendered having just plain text 

Unfortunately had to admit defeat to go back to use table structure with css to render a better way 

Even for the image using width and height in the style property would not make the email render as I wanted and to use the width and height HTML property

```
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
<td> <h1>New annoucement </h1></td>

</tr>
<tr><td><img data-imagetype="External" src="https://cdn.hubblecontent.osi.office.net/m365content/publish/aa2dc6cd-0710-4b94-b49d-3464108dd918/905395820.jpg" border="0" id="x__x0000_i1025" width="400" height="200"></td>
</tr>
<tr><td>
<h2>@{outputs('Get_details_for_selected_item')?['body/AnnouncementTitle']}</h2></td></tr>
<tr style="height:11pt;text-align: center;"> 
<td style="padding:40px">  <h3> <a href="@{outputs('Get_details_for_selected_item')?['body/{Link}']}">Go to item</a></h3></td>
</tr>
</tbody>
</table>
</body>
```

Email picture from outlook desktop and outlook on the web


Not very happy with my HTML hack , anyone who can get it done a better way with div, th

Transformed into table 

''