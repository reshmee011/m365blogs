---
title: 'SSRS Export to PDF results in sentence to be concatenated into one long word'
date: Tue, 27 Oct 2015 14:09:42 +0000
draft: false
tags: ['SQL', 'SQL SSRS', 'SSRS']
---

Sometimes invalid ASCII code invisible to naked eye are stored in the SQL database. If the value with the invalid ASCII is displayed in a SSRS report and exported to PDF, the set of words become concatenated into one word Example "The director of the company has given approval" displayed in a textbox in SSRS become "Thedirectorofthecompanyhasgivenapproval" in PDF. The first character of the text value was ASCII code 63. Unfortunately there are other characters that are mapped to 63. http://stackoverflow.com/questions/15982747/char-returns-the-wrong-value-for-29-unicode-charachters-need-net-cast-conve To remove the first character a replace function can be used select Replace(CAST(COLUMN\_NAME as varchar(max)) COLLATE SQL\_Latin1\_General\_CP1\_CI\_AS, CHAR(63), '') The value is displayed correctly in PDF after the fix is applied.