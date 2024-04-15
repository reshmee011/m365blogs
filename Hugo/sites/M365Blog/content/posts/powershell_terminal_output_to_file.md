---
title: "How to Redirect PowerShell Terminal Output to a File"
date: 2024-04-15T07:10:39+01:00
tags: ["PowerShell", "Terminal", "Console", "buffer limit"]
featured_image: '/images/powershell_terminal_output_to_file/output.png'
draft: false
---

# How to Redirect PowerShell Terminal Output to a File

PowerShell console buffers have practical limitations, and displaying thousands of lines of output in the console can lead to performance issues or truncation of older output. There's a simple trick to retain all output for review: redirecting it to a file.

To achieve this, you can use the `>` operator followed by the desired output file name whi.

```PowerShell
Connect-SPOService -Url https://reshmeeauckloo-admin.sharepoint.com
Get-SPOTenant > C:\temp\sposettings.txt
```

Executing this command will save the output to the specified file path.

![Output](../images/powershell_terminal_output_to_file/output.png)

To view the saved output on the terminal use the cmdlet `Get-Content`

```powershell
Get-Content -Path C:\temp\sposettings.txt
```
![Output](../images/powershell_terminal_output_to_file/viewsavedoutput.png)


## Conclusion

By redirecting the output to a file, no information is lost, making it easier to analyse results and troubleshoot any issues that may arise, especially when dealing with output exceeding the console terminal's buffer limit.
