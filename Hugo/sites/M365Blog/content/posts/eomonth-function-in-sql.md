---
title: 'EOMONTH Function in SQL'
date: Tue, 27 Oct 2015 12:05:27 +0000
draft: false
tags: ['SQL']
---

EOMONTH returns the last day of the month that is the indicated number of months before or after start\_date. Syntax SELECT EOMONTH(start\_date, months) The EOMONTH function syntax has the following arguments: 1. Start\_date: Required argument. A date that represents the starting date. 2. Months: Optional argument. The number of months before or after start\_date. A positive value for months returns a future date; a negative value returns a past date. Please note that if months is not an integer, it is truncated. Examples select EOMONTH('2015-10-27') - returns '2015-10-31' select EOMONTH('2015-10-27',1) - returns '2015-11-30' select EOMONTH('2015-10-27',1.5) - returns '2015-11-30' select EOMONTH('2015-10-27',10) - returns '2016-08-31' select EOMONTH('2015-10-27',-10) - returns '2014-12-31'