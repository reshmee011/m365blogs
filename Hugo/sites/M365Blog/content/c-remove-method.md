---
title: 'C# Remove Method'
date: Thu, 26 Nov 2015 18:41:14 +0000
draft: false
tags: ['.net', 'C#', 'C# .net', 'Uncategorized']
---

The Remove method is a nice alternative to substring method For example, string strStringToTruncate = "This is a beautiful house"; string strTruncatedString = strStringToTruncate.Remove(5,10); Console.Write(strTruncatedString); strTruncatedString = strStringToTruncate.Remove(5); Console.Write(strTruncatedString); strTruncatedString = strStringToTruncate.Remove(0,5); Console.Write(strTruncatedString); The result "This iful house" "This" "is a beautiful house" More information can be found https://msdn.microsoft.com/en-us/library/d8d7z2kk(v=vs.110).aspx The method removes a number of characters based on a start index.