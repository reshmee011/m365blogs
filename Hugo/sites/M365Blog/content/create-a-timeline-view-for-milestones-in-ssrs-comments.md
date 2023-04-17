---
title: 'Create a TimeLine View for Milestones and Project Duration using Matrix in SSRS'
date: Tue, 12 Jan 2016 00:01:31 +0000
draft: false
tags: ['Project Server', 'Project Server 2013', 'SQL', 'SSRS', 'SSRS TimelineView Milestones Projects ProjectServer2010 ProjectServer2013 Dataset Parameters']
---


#### 
[Julia]( "huorn@gmx.de") - <time datetime="2017-07-27 14:20:21">Jul 4, 2017</time>

Is it possible to insert a Linebreak after each Milestone? So if you have a bunch of Milestones that you can read it more easily.
<hr />
#### 
[Julia]( "huorn@gmx.de") - <time datetime="2017-07-28 06:51:32">Jul 5, 2017</time>

It's quit easy to add a Linebreak in Tooltipp. Just edited the Statement as below: ,(STUFF( ( SELECT ' ' + convert(nvarchar(3), day(TaskFinishDate)) + ' ' + + convert(char(3), TaskFinishDate , 0) + ' ' + + convert(nvarchar(6), year(TaskFinishDate)) + ' - ' + T.TaskName +@NewLineChar FROM rpt\_MS\_Task T At the beginning of the Statement i Declare @NewLineChar as follwos: DECLARE @NewLineChar AS CHAR(2) = CHAR(13) + CHAR(10) This works well.
<hr />
