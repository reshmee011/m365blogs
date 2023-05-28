---
title: 'GetFieldValueAsHTML on multiline textboxes return  instead of'
date: Fri, 19 Sep 2014 16:02:31 +0000
draft: false
tags: ['SharePoint', 'Uncategorized']
---

The <br/> gets inserted whenever shift>enter is pressed  inside a multiline text box in SharePoint list. [![Pic_ColTBD](http://reshmeeauckloo.files.wordpress.com/2014/09/pic_coltbd.png?w=300)](https://reshmeeauckloo.files.wordpress.com/2014/09/pic_coltbd.png) Above is a sample text entered in multiline column Col\_ToBeDel. [![Pic_GetFieldValueAsHTML](http://reshmeeauckloo.files.wordpress.com/2014/09/pic_getfieldvalueashtml.png?w=300)](https://reshmeeauckloo.files.wordpress.com/2014/09/pic_getfieldvalueashtml.png) When the getfieldvalueashtml method is called against the column value , <br> is returned instead of <br/> tag. If LoadXML method is called against the value returned by getfieldvalueashtml , it will complain about invalid XML. The fix can be to replace <br> with <br/> before calling LoadXML.