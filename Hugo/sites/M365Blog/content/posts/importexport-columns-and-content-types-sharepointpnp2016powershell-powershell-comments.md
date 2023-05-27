---
title: 'Import/Export Columns and Content Types using PnP PowerShell'
date: Fri, 24 Jun 2016 11:52:05 +0000
draft: false
tags: ['PnP', 'PnP Migration Site Columns Content Types SharePoint 2013 SharePoint Online', 'PowerShell', 'SharePoint 2013', 'SharePoint Online', 'Uncategorized']
---


#### 
[reshmee011](http://reshmeeauckloo.wordpress.com "reshmee011@gmail.com") - <time datetime="2017-10-05 14:42:47">Oct 4, 2017</time>

I will update the scripts at some point. Thanks for the comment.
<hr />
#### 
[Karl G. Schneider]( "k.schneider@ipi-gmbh.com") - <time datetime="2017-10-05 14:50:05">Oct 4, 2017</time>

The Imported TaxonomyFields seems not working, because of missing the second Taxonomy-Note-Field. I always get "The SPListItem being updated was not retrieved with all taxonomy fields" in Edit-Form of the imported Managed Metadata Columns. I have observed this. http://blog.bugrapostaci.com/2015/07/03/the-splistitem-being-updated-was-not-retrieved-with-all-taxonomy-fields/
<hr />
#### 
[Karl G. Schneider]( "k.schneider@ipi-gmbh.com") - <time datetime="2017-10-05 11:42:01">Oct 4, 2017</time>

Actually you must use the new PnP-PowerShell-Namespace in the PowerShell-Commands. Example: Connect-PnPOnline instead of Connect-SPOnline and so on...
<hr />
