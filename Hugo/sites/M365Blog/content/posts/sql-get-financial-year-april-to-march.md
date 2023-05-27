---
title: 'SQL Get Financial Year (April to March)'
date: Wed, 02 Dec 2015 17:59:00 +0000
draft: false
tags: ['SQL', 'SQL SSRS', 'SSRS']
---

The following query will return the financial year starting April to March YEAR(DATEADD(Month,-((DATEPART(Month,StartDate)+8) %12),StartDate)) SELECT YEAR(DATEADD(Month,-((DATEPART(Month,'2014-03-31')+8) %12),'2014-03-31')) SELECT YEAR(DATEADD(Month,-((DATEPART(Month,'2014-04-01')+8) %12),'2014-04-01')) SELECT YEAR(DATEADD(Month,-((DATEPART(Month,'2014-12-31')+8) %12),'2014-12-31')) SELECT YEAR(DATEADD(Month,-((DATEPART(Month,'2015-03-31')+8) %12),'2015-03-31')) SELECT YEAR(DATEADD(Month,-((DATEPART(Month,'2015-04-01')+8) %12),'2015-04-01')) The results are   ![FinancialYearAprilMarch.JPG](https://reshmeeauckloo.files.wordpress.com/2015/12/financialyearaprilmarch.jpg)