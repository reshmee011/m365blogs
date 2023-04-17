---
title: 'Stripping HTML tags from SSRS text fields'
date: Tue, 27 Oct 2015 09:05:02 +0000
draft: false
tags: ['SSRS', 'SSRS VB', 'VB']
---


#### 
[TimK]( "kwiatkowski@gmail.com") - <time datetime="2017-06-08 23:09:28">Jun 4, 2017</time>

Appreciate the code. One question; Why is this required? Dim objRegExp AS NEW System.Text.RegularExpressions.Regex( “”) and RETURN objRegExp.Replace(sInput, “”) Wouldn’t a simple return of sInput suffice? What am I missing here? Been racking my brain for hours. Thanks!
<hr />
#### 
[reshmee011](http://reshmeeauckloo.wordpress.com "reshmee011@gmail.com") - <time datetime="2017-06-09 08:33:30">Jun 5, 2017</time>

Hi TimK The HTML text I had to work with had encoded characters , for example "&" instead of "&". The regex expression will remove all HTML tags but for my requirements, I wanted to preserve line breaks () , so I am handling these tags individually to be replaced with line feed (chr(10)). Also had to transform to bulleted string , in the example above I am replacing "" with "-". Depending on your requirements, thus you might have to amend the code further. Thanks Reshmee
<hr />
#### 
[Riad]( "riad@xentrion.com") - <time datetime="2017-06-07 04:08:53">Jun 3, 2017</time>

Hi, I tried to use your function and I get an error " String constants must end with a double quote". thanks. Using Visual Studio 2013 for report design.
<hr />
#### 
[reshmee011](http://reshmeeauckloo.wordpress.com "reshmee011@gmail.com") - <time datetime="2017-06-07 17:52:46">Jun 3, 2017</time>

Thanks for trying the code. Unfortunately the original code which has encoded characters got converted into plain text. I have updated the post with a link to gist. Can you please try the code again?
<hr />
#### 
[olga Melo]( "omelo@pinchin.com") - <time datetime="2018-02-26 18:15:00">Feb 1, 2018</time>

How can I use your expression along with truncating the text to 150 characters
<hr />
#### 
[Porsha Douet](https://extraproxies.com/tag/harder/ "DorotheaTweedie@mail.com") - <time datetime="2021-08-30 18:16:19">Aug 1, 2021</time>

Howdy, i read your blog from time to time and i own a similar one and i was just wondering if you get a lot of spam comments? If so how do you stop it, any plugin or anything you can recommend? I get so much lately it's driving me insane so any support is very much appreciated.
<hr />
#### 
[mail53.aspx](http://delaprovidencia.edu.ec/mail53.aspx "noyakstokwd@gmail.com") - <time datetime="2020-06-06 11:14:27">Jun 6, 2020</time>

The first time I saw him my attention was drawn to the technique he had, the fluidity of his technique.
<hr />
