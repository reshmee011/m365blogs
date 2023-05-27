---
title: 'SharePoint Get list items and Effective Permissions on a list via REST API from SharePoint Add-in'
date: Mon, 16 Nov 2015 09:41:59 +0000
draft: false
tags: ['CSOM', 'JavaScript', 'jQuery', 'SharePoint add-in', 'SharePoint Online']
---


#### 
[Ofer Gal](https://plus.google.com/+OferGalAbaShelTomer "ofergal@gmail.com") - <time datetime="2017-03-15 14:56:15">Mar 3, 2017</time>

I get error: Uncaught (in promise) TypeError: SP.BasePermissions is not a constructor What else should I load?
<hr />
#### 
[reshmee011](http://reshmeeauckloo.wordpress.com "reshmee011@gmail.com") - <time datetime="2017-03-15 22:45:01">Mar 3, 2017</time>

Sp.js needs to be loaded. Add the following to your page. [/\_layouts/15/sp.runtime.js](/_layouts/15/sp.runtime.js) [/\_layouts/15/sp.js](/_layouts/15/sp.js)
<hr />
