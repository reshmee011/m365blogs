---
title: 'Stripping HTML tags from SSRS text fields'
date: Tue, 27 Oct 2015 09:05:02 +0000
draft: false
tags: ['SSRS', 'SSRS VB', 'VB']
---

I have worked extensively with SSRS. One of the common requirements is to strip HTML from text values displayed in reports and apply standard formatting to the text. HTML have many encoded characters which end users don't want to see. The function below replaces encoded characters to the correct string http://www.w3schools.com/tags/ref\_urlencode.asp The requirement was to add a carriage return whenever a "<br>" is found. Because in some text the "<br>" was "<BR>" and "<br/>", additional code has been added to cater for them. The <li>tag needed to be preserve somehow explaining why it's been converted to "-" Finally the function uses the Regex expression "<(.|\\n)+?>" to strip any html https://gist.github.com/reshmee011/ed47bb595fbfec4427440a7b3e68f212 To use the above function, follow the steps below 1. Navigate to report properties>code tab. Copy and paste the above code. \[gallery size="full" ids="24"\] 2.Call the function from any textbox value property where needed as follows: =Code.StripHTML() \[gallery ids="25"\]