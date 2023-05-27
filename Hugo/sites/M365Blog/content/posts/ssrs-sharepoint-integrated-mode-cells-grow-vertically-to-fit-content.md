---
title: 'SSRS SharePoint Integrated Mode - Cells grow vertically to fit content'
date: Tue, 19 Jan 2016 17:54:34 +0000
draft: false
tags: ['Project Server 2013', 'SharePoint 2013', 'SSRS', 'SSRS SharePoint Integrated Mode']
---

I have come across this issue several times that even the "Can Grow" property  of the cell is set to false, the cell grows vertically to fit contents forcing the same row to stretch to the height of the cell distorting the rendering of the report on the browser though it might display correctly in Report Builder or SQL data tools in Visual Studio. Can Grow is set to False from the cell property. ![CanGrow_False](https://reshmeeauckloo.files.wordpress.com/2016/01/cangrow_false.png) Cell displayed correctly Report Builder ![DisplayCorrectly_ReportBuilder](https://reshmeeauckloo.files.wordpress.com/2016/01/displaycorrectly_reportbuilder.png) Report does not display correctly from report library in SharePoint 2013 ![DisplayIncorrectly_Browser](https://reshmeeauckloo.files.wordpress.com/2016/01/displayincorrectly_browser.png) **Fix**

1.  Clear the cell, i.e. delete the textbox and right click on the cell to add a rectangle.

![Insert_Rectangle](https://reshmeeauckloo.files.wordpress.com/2016/01/insert_rectangle.png) 2. After adding the rectangle control, right click to add a textbox ![Insert_TextBox](https://reshmeeauckloo.files.wordpress.com/2016/01/insert_textbox.png) 3. Select the field to display in the textbox and set writingmode to 270. ![TextBoxInsideRectangleSetTo270](https://reshmeeauckloo.files.wordpress.com/2016/01/textboxinsiderectanglesetto270.png) 4. Save the report from Report Builder and run the report from browser. ![fixed_Resize](https://reshmeeauckloo.files.wordpress.com/2016/01/fixed_resize.png)