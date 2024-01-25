
Controls
classic control Combo box 

```
Set(varManager,Office365Users.Manager(cmbAssessee.Selected.Mail).Mail); 
Set(varManagerManager,Office365Users.Manager(varManager).Mail);

Office365Outlook.SendEmailV2(cmbAssessee.Selected.Mail, "360 Feedback Review",HtmlText1.HtmlText);   
```

```html$"
<style> 
h1 {{ 
  text-align: center;
  margin-bottom: 20px;
}}
a {{
  text-decoration: none;
  color: white;
  background-color: #357EC7;
  padding:10px 10px 20px 10px;
  border-radius: 5px;
  transition: background-color 0.3s ease;
}}
body {{
display:flex;
justify-content:center;
align-items: center;
height:100vh;
}}
table {{
background-color:white;
max-width:500pt;
border:1pt solid #C8C8C8;
margin:auto;
border-collapse:collapse;
}}
td,th {{
 text-align:center;
 vertical-align:middle;
 padding:10px 10px;
}}
</style>
<body>
<table>
<tbody>
<tr style='height:11pt;'> 
<td> <h1>'360 Feedback Review' </h1></td>
</tr>
<tr><td style='padding:40px'><img data-imagetype='External' src='https://cdn.hubblecontent.osi.office.net/m365content/publish/aa2dc6cd-0710-4b94-b49d-3464108dd918/905395820.jpg' border='0' width='400' height='200'></td>
</tr>
<tr><td> {nfLoggedonUserDisplayName} has submitted rating of <b> {Rating1.Value} </b> out of for {cmbAssessee.Selected.displayName}
<h2> {Concatenate(Concat(Sequence(Rating1.Value), "<span style='font-size:300%;color:gold;'>&starf;</span>"),Concat(Sequence(Rating1.Max - Rating1.Value), "<span style='font-size:300%;color:gold;'>&star;</span>")) }</h2></td></tr>
<tr> 
<td ><p> {txtRatingJustificationAnswer.HtmlText} </p></td>
</tr>
</tbody>
</table>
</body>"
```

### String Interpolation with HTML Text 
Use of {{}} for CSS style

### Sequence function 
Concatenate(Concat(Sequence(Rating1.Value), "<span style='font-size:300%;color:gold;'>&starf;</span>"),Concat(Sequence(Rating1.Max - Rating1.Value), "<span style='font-size:300%;color:gold;'>&star;</span>"))

### HTML Star Code
Use 
&#9734;
&#9733;

instead of 

&star;
&starf;
&bigstar;

### Send Email 
IfError (Office365Outlook.SendEmailV2(cmbManager.Selected.mail, "360 Feedback Review",hiddenHtml.HtmlText),Notify("Failed to send email",NotificationType.Error,400),Navigate(scrSuccess))

SendEmailV2 instead of SendEmailV which does not support HTML formatted text.

IfError (Office365Outlook.SendEmailV2(cmbManager.Selected.mail, "360 Feedback Review",hiddenHtml.HtmlText),Notify("Failed to send email",NotificationType.Error,400),Navigate(scrSuccess))  
## References

[Power Apps Send Email Using Outlook – The Complete Guide](https://www.matthewdevaney.com/power-apps-send-email-using-outlook-the-complete-guide/)
[HTML Star Code](https://www.html.am/html-codes/character-codes/html-star-code.cfm)
