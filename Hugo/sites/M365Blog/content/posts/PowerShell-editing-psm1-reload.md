---
title: "Working with psm1 Files in PowerShell"
date: 2024-02-07T17:32:36Z
tags: ["psm1","PowerShell"]
featured_image: '/posts/images/PnPPowerShell_StartingToContribute/ForkYourOwnRepository.png'
draft: true
---

# Working with psm1 Files in PowerShell

Have you ever utilized a psm1 file to create reusable functions across multiple ps1 files? If so, you might have encountered the frustration of edits not reflecting immediately due to module caching.

I recently faced this issue when updating a psm1 file; despite making changes, the updates didn't take effect immediately. It took some experimentation, including trying different string concatenation methods like Join, string interpolation, and using the + sign, to realize that the issue stemmed from module caching.

The only solution I found to load the updated psm1 file was to terminate the PowerShell session and start a new one. This hiccup highlighted the importance of understanding how PowerShell handles module caching and the necessity of refreshing the session to apply changes.

If you prefer not to install the psm1 module permanently, you can call it using the 'using' keyword:

```
using module ..\..\get-files.psm1
```

This approach allows you to utilize the module without installing it permanently, facilitating easier testing and experimentation.

# Conclusion

