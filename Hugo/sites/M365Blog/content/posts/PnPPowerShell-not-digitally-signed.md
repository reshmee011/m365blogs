---
title: "Resolving the 'PnP PowerShell Not Digitally Signed' Issue"
date: 2024-06-15T14:49:19+01:00
tags: ["PnP PowerShell", "Digital Signature", "Execution Policy"]
featured_image: '/posts/images/PnPPowerShell-not-digitally-signed/Notdigitallysigned.png'
draft: false
---

# Resolving the 'PnP PowerShell Not Digitally Signed' Issue

If you've recently upgraded PnP PowerShell to the latest nightly build and are encountering errors when trying to execute any PnP PowerShell cmdlets, this guide is for you.

```PowerShell
PS C:\Users\RAuckloo> connect-pnponline -url https://contoso-admin.sharepoint.com -Interactive
connect-pnponline: The 'connect-pnponline' command was found in the module 'PnP. PowerShell', but the module could not be loaded due to the following error: [Errors occurred while loading the format data file:
\\contoso-it.org.uk\userdata\Profiles$\rauckloo\Documents\PowerShell\Modules\PnP.PowerShel1\2.4.78\PnP.PowerShell.Format.ps1xml, , \\contoso-it.org.uk\userdata\Profiles$\rauckloo\Documents\PowerShell\Modules\PnP.PowerShel1\2.4.78\PnP.PowerShell.
Format.ps1xml: The file was skipped because of the following validation exception: File \\contoso-it.org.uk\userdata\Profiles$\rauckloo\Documents\PowerShell\Modules\PnP.PowerShell\2.4.78\PnP. PowerShell.Format.ps1xml cannot be loaded. The file \\contoso-it.org.uk\userdata\Profiles$\rauckloo\Documents\PowerShel1\Modules\PnP. PowerShell\2.4.78\PnP. PowerShell. Format.ps1xml is not digitally signed. You cannot run this script on the current system. For more information about runnings
cripts and setting execution policy, see about_Execution_Policies at https://go.microsoft.com/fwlink/?LinkID=135170 ..

For more information, run 'Import-Module PnP. PowerShell'.
```

![Not digitally signed](../images/PnPPowerShell-not-digitally-signed/Notdigitallysigned.png)

he error message indicates that the PnP PowerShell module is not digitally signed, which prevents it from being loaded on your system. This is a security measure to prevent the execution of malicious scripts.

The solution is to change the execution policy of your PowerShell session. This can be done using the **set-executionpolicy** cmdlet. The following command sets the execution policy for the current process to Bypass, which allows scripts to run regardless of their signing status:

```powerShell
set-executionpolicy -scope process -executionpolicy bypass
```

![After execution policy](../images/PnPPowerShell-not-digitally-signed/AfterExecutionPolicy.png)

Please note that changing the execution policy can have security implications. It's recommended to only use Bypass for trusted scripts and modules. Always ensure that the scripts and modules you're running are from a trusted source. `
