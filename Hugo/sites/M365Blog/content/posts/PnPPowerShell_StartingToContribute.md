---
title: "PnPPowerShell_StartingToContribute"
date: 2023-12-11T01:52:50Z
draft: true
---

## Forking and cloning 

```powershell
# Find a nice directory on your filesystem and...    
git clone https://github.com/reshmee011/pnppowershell # Clone the repo    
cd cli-microsoft365 # Enter the directory    
code . # Open the directory in VS Code
```

## Configure upstream

```powershell
# Configure an upstream so you can sync changes with the source repo
git remote add upstream https://github.com/pnp/powershell

# Create a branch for your issue or feature and check it out
git checkout -b command-xyz
```

## Start hacking

## Debug and Test changes

1. Builds a debug version for local testing  

```powershell
c:\Users\reshm\source\repos\powershell\build\Build-Debug.ps1
```

![Build debug version](../images/PnPPowerShell_StartingToContribute/BuildDebug.png)

2. Import the debug version of PnP PowerShell

```powershell
import-module "C:\Users\reshm\Documents\PowerShell\Modules\PnP.PowerShell\PnP.PowerShell.psd1"
connect-pnpOnline -Url https://reshmeeauckloo-admin.sharepoint.com -Interactive

$container = get-pnpcontainer -OwningApplicationId a187e399-0c36-4b98-8f04-1edc167a0996
```
3. Add breakpoint to code

4. Attach debugger to pwsh.exe 

5. Running code will code cause it to pause at breakpoints 
1. 
## To ensure latest before creating a PR

```powershell
# Sync and rebase main branch to get the latest changes
git fetch upstream
git checkout main
git pull --rebase upstream main

# Rebase feature branch to get the latest changes
git checkout command-xyz
git rebase main
```

## References
