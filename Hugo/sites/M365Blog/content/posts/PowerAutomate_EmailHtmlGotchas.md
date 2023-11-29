---
title: "PowerAutomate_EmailHtmlGotchas"
date: 2023-11-29T16:15:32Z
draft: true
---

# Email HTML in Outlook

Want a nicely formatted post to attract recipient's attention so that it's not just another email and tried to use div to render the email for responsiveness and what modern html should be formatted.

You tried something like

<div style="max-width: 500px; margin: auto; display: flex; justify-content: center; align-items: center; height: 100px; border: 1px solid black;">
  <h1 style="text-align: center;">Announcement Title</h1>
  <button style="font-size: 16px; background-color: blue; color: white;">Go to item</button>
</div>

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
