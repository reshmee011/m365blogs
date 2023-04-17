---
title: 'Nintex SharePoint Online Create SharePoint Group with Contribute permissions'
date: Tue, 03 May 2016 14:53:03 +0000
draft: false
tags: ['Nintex', 'REST', 'REST POST Nintex SharePoint Online', 'SharePoint', 'SharePoint 2013', 'SharePoint Online']
---


#### 
[Mixons]( "pdonoso@latinshare.com") - <time datetime="2016-11-14 13:26:02">Nov 1, 2016</time>

@Dub do you found the solution ?? i have the same problem
<hr />
#### 
[Jeroen]( "jeroen.wetzels@geone.cloud") - <time datetime="2016-08-22 09:30:16">Aug 1, 2016</time>

Hello Reshmee, Great tutorial! I could confirm that the content type has to be text/xml (as described above). But when I try to create the Group. I got the following error message: "Access denied. You do not have permission to perform this action or access this resource"
<hr />
#### 
[Sam]( "shk21@hotmail.com") - <time datetime="2017-02-15 23:08:22">Feb 3, 2017</time>

Hi, has anyone found a solution to this Access denied error? thanks.
<hr />
#### 
[Dub]( "dub.six66@gmail.com") - <time datetime="2016-10-21 00:26:17">Oct 5, 2016</time>

Hi, I've done both the workflow app permissions feature and also the app permissions to full control as above but i'm getting "Unauthorized" being returned from the Create Group HTTP call? Any ideas?
<hr />
#### 
[reshmee011](http://reshmeeauckloo.wordpress.com "reshmee011@gmail.com") - <time datetime="2016-10-21 08:24:53">Oct 5, 2016</time>

@Dub: Does the user you used to get the request digest info has access to create groups ?
<hr />
#### 
[reshmee011](http://reshmeeauckloo.wordpress.com "reshmee011@gmail.com") - <time datetime="2016-06-28 06:18:23">Jun 2, 2016</time>

It looks as if the result is returned in JSON format. Can you please try to change the content type to text/xml when making request to /\_api/ContextInfo and see if it works?
<hr />
#### 
[reshmee011](http://reshmeeauckloo.wordpress.com "reshmee011@gmail.com") - <time datetime="2016-08-22 09:47:35">Aug 1, 2016</time>

Hi Jeroen Thanks for the comment. The permissions of the workflow app need to be configured. I have described it at the beginning of another post https://reshmeeauckloo.wordpress.com/2015/11/28/nintex-workflow-sharepoint-online-create-sub-site-http-web-service/ In summary, workflow needs to be enabled to use app permissions and be given full control from appinv.aspx page. Follow the instructions from msdn link below. https://msdn.microsoft.com/en-us/library/office/jj822159(v=office.15).aspx
<hr />
#### 
[Victor]( "vcamarabados@gmail.com") - <time datetime="2016-06-28 05:04:50">Jun 2, 2016</time>

Hello I am following your steps but in the step 2 ◾Add “Query XML” action I get the following error: XML content is invalid. 6/28/2016 6:55 AM Comment {"d":{"GetContextWebInformation":{"\_\_metadata":{"type":"SP.ContextWebInformation"},"FormDigestTimeoutSeconds":1800,"FormDigestValue":"0xF62EFE5E7 6/28/2016 6:56 AM Comment XML content is invalid. Data at the root level is invalid. Line 1, position 1..
<hr />
#### 
[Brian W]( "bmwatson@beckman.com") - <time datetime="2016-12-22 18:12:37">Dec 4, 2016</time>

Same issue for me, receive the message "Access denied. You do not have permission to perform this action or access this resource." on the create group HTTP call. All actions and the workflow itself are running with site collection admin credentials. Stuck on next steps......great tutorial thus far though!
<hr />
#### 
[reshmee011](http://reshmeeauckloo.wordpress.com "reshmee011@gmail.com") - <time datetime="2017-02-16 17:16:05">Feb 4, 2017</time>

The effective permissions of the workflow are an intersection of user permissions and the app permissions similar to SharePoint apps. If the user running the workflow does not have permissions to create group then access denied error will be thrown.
<hr />
#### 
[Sam]( "shk21@hotmail.com") - <time datetime="2017-02-16 23:06:02">Feb 4, 2017</time>

Hi, the user is set up as site collection admin. Also, the workflow actions are running within app step with elevated privileges.
<hr />
