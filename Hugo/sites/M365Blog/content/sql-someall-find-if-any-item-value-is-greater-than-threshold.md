---
title: 'SQL Some|All:  Find if any item value is greater than threshold'
date: Wed, 02 Dec 2015 22:17:34 +0000
draft: false
tags: ['Uncategorized']
---

I had a requirement to return a warning message if any rating exceeds a number. I came across the msdn article describing the use of keyword SOME ALL https://msdn.microsoft.com/en-us/library/ms175064.aspx `DECLARE @RiskRating TABLE (ID int,Rating int) ; INSERT @RiskRating VALUES (1,20) ; INSERT @RiskRating VALUES (2,5) ; INSERT @RiskRating VALUES (3,6) ; INSERT @RiskRating VALUES (4,10) ; INSERT @RiskRating VALUES (5,25) ; INSERT @RiskRating VALUES (6,16) ; INSERT @RiskRating VALUES (7,1) ; IF 25 <= SOME (SELECT Rating FROM @RiskRating) PRINT 'One or more Risk Rating is equal or above 25' ELSE PRINT 'No Risk Rating is equal or above 25' ; IF 25 <= ALL (SELECT Rating FROM @RiskRating) PRINT 'All Risk Rating is equal or above 25' ELSE PRINT 'Not all Risk Rating is equal or above 25' ;` Results One or more Risk Rating is equal or above 25 Not all Risk Rating is equal or above 25