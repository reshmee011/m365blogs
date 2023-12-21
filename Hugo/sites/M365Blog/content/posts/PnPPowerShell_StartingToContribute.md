---
title: "A Guide to Contributing to PnP PowerShell"
date: 2023-12-11T01:52:50Z
tags: ["PnP","PowerShell","Git","Github", "Open Source","Contributions"]
featured_image: '/posts/images/PnPPowerShell_StartingToContribute/ForkYourOwnRepository.png'
draft: false
---

# A Guide to Contributing to PnP PowerShell

Contributing to PnP PowerShell is a rewarding journey. Whether you're a seasoned contributor or a beginner, this guide aims to simplify the process and keep it handy for your next contribution.

## Prerequisites

1. Install [Git](https://git-scm.com/downloads)
2. Install [PowerShell 7](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell)
3. Install [Visual Code](https://code.visualstudio.com) or **Visual Studio Code**
4. Install .NET SDK 6 (https://dotnet.microsoft.com/download/dotnet/6.0)

 
## Forking and cloning 

First things first, make a copy of the repository to start working on your changes. While making minor changes via GitHub's web editor or Codespaces is an option, I prefer having the repository on my local machine for more flexibility.

### Fork Your Repository: This creates your copy of the repository.

Navigate to [PnP PowerShell](https://github.com/pnp/powershell) and fork the repository

![Fork your own repository](../images/PnPPowerShell_StartingToContribute/ForkYourOwnRepository.png)

### Clone the Repository: Clone the forked repository onto your local machine using the following commands:
```powershell
# Navigate to the path where the repository will be cloned
cd C:\Users\reshm\source\repos
# Find a nice directory on your filesystem and...    
git clone https://github.com/reshmee011/powershell # Clone the repo    
cd powershell # Enter the directory    
code . # Open the directory in VS Code
```

![Clone your repo](../images/PnPPowerShell_StartingToContribute/RepoClone.png)

## Configure upstream
Setting up an upstream allows you to sync changes with the source repository.

```powershell
# Configure an upstream so you can sync changes with the source repo
git remote add upstream https://github.com/pnp/powershell

# Create a branch for your issue or feature and check it out
git checkout -b command-xyz
```

## Start hacking

With your environment set up, dive into making your changes!

## Debug and Test changes

1. **Builds a debug version** : Use PowerShell 7 or higher to build a debug version for local testing

```powershell
c:\Users\reshm\source\repos\powershell\build\Build-Debug.ps1
```

![Build debug successful](../images/PnPPowerShell_StartingToContribute/BuildDebug.png)

Build debug may fail if PowerShell file is already in use
![Build debug failes](../images/PnPPowerShell_StartingToContribute/BuildFailedBecauseofFileInUse.png)

The resolution is to close any PowerShell code editor loading the PnP.PowerShell.psd1 file including the PowerShell core used to build the file.
![Kill the terminal](../images/PnPPowerShell_StartingToContribute/KilltheTerminal.png)

2. **Import the debug version**: Import the debug version of PnP PowerShell

```powershell
import-module "C:\Users\reshm\Documents\PowerShell\Modules\PnP.PowerShell\PnP.PowerShell.psd1"
connect-pnpOnline -Url https://reshmeeauckloo-admin.sharepoint.com -Interactive

$container = get-pnpcontainer -OwningApplicationId a187e399-0c36-4b98-8f04-1edc167a0996
```
3. **Add Breakpoints and Debug**: Use breakpoints and attach a debugger to pwsh.exe to pause and debug your code.

![Attach to process](../images/PnPPowerShell_StartingToContribute/AttachToProcessor_pwshexe.png)

Running code will code cause it to pause at breakpoints 
![Attach to process](../images/PnPPowerShell_StartingToContribute/BuildDebug.png)

## Commit Changes

Once your changes are ready:

```powershell
# Edit files, add and commit. Then push with the -u (short for --set-upstream) option
 git add -A
 git commit -m "new cmdlet for remove-pnpcontainer"
 git push --set-upstream origin command-xyz 
``` 

![Publish branch with changed](../images/PnPPowerShell_StartingToContribute/CreateAndPublishbranch.png)

## Squash Multiple Commits 

Squash multiple commits into one for a cleaner history:

```powershell
git rebase -i HEAD~n
```

## Create Pull request 


Sync your branch with the main repository and ensure it's up to date:
## To ensure latest before creating a PR

```powershell
# Sync and rebase main branch to get the latest changes
git fetch upstream
git checkout dev
git pull --rebase upstream dev

# Rebase feature branch to get the latest changes
git checkout command-xyz
git rebase dev
```

![rebase](../images/PnPPowerShell_StartingToContribute/RebaseCode.png)

Navigate to the branch on the browser and create a pull request referencing your feature branch and the main branch of the central repository.

![branch with commits ahead](../images/PnPPowerShell_StartingToContribute/branchwithcommitahead.png)

Once ready, create the pull request via the GitHub site and fill in the details.

Create Pull Request by clicking the button **Create Pull Request**
![Create PR](../images/PnPPowerShell_StartingToContribute/CreatePR.png)

Fill in details
![Create PR with details](../images/PnPPowerShell_StartingToContribute/CreatePRDetails.png)

## Troubleshooting
Should any issues arise during the pull request process, inspect the PR checks for detailed feedback. If needed, resolve conflicts or errors encountered.

![Some checks were not successful](../images/PnPPowerShell_StartingToContribute/Somecheckswerenotsuccessful.png)

Drilling on the failed step by clicking on details may reveal the issue why it failed. In that instance there were no issues with the Pull Request submitted but was due to a new version being released. Reach out to any of the PnP PowerShell Maintainers : Gautam Sheth, Veronique Lengelle or Koen Zomers to understand if you may need to amend anything in your codebase to ensure a successful build.

![buildfail](../images/PnPPowerShell_StartingToContribute/BuildMayFail.png)


You may delete the repository after the PR is merged and recreated the fork each time you need to contribute to ensure latest.
 ![Delete Repository](../images/PnPPowerShell_StartingToContribute/DeleteRepository.png)

## Final Touches

Once you have made the changes to a repository, you are ready to amend your commit. You can do this by using the –no-edit flag:

```powershell
git commit --amend --no-edit
```

To undo changes in local branch, use git reset --hard or git -clean -fd with caution, view the [post](https://github.blog/2015-06-08-how-to-undo-almost-anything-with-git/) for more undo cmdlets

```powershell
git reset --hard
```

git reset --hard will not remove untracked files and might want to use git clean to remove any files from the tracked root directory that are not under Git tracking.

```powershell
git clean -fd
```

Please check [Getting Started with Git-]([https://www.linkedin.com/posts/brijpandeyji_getting-started-with-git-%3F%3F%3F-%3F%3F%3F-activity-7141752823460896768-MHJe?utm_source=share&utm_medium=member_desktop)
for helpful git cmdlets you may use 

## Conclusion

Contributing to PnP PowerShell is a learning process. Remember, each contribution counts, and this guide is your roadmap to make impactful changes seamlessly. Happy coding!

## References

[Rewriting history](https://www.atlassian.com/git/tutorials/rewriting-history#:~:text=To%20review%2C%20git%20commit%20%2D%2D,the%20last%20commit%20message%20log)
[Contributing as a holiday season present](https://www.blimped.nl/contributing-as-a-holiday-season-present/)
[Contribution guidance](https://pnp.github.io/powershell/articles/gettingstartedcontributing.html#:~:text=Open%20your%20browser%20and%20go,you%20have%20changed%20and%20why.)
[GitHub Universe Cloud Skills Challenge](https://learn.microsoft.com/en-gb/collections/kkqrhmxoqn54?WT.mc_id=cloudskillschallenge_ef5f9f41-0818-4895-9217-79d19827a322)
[Getting Started with Git-](https://www.linkedin.com/posts/brijpandeyji_getting-started-with-git-%3F%3F%3F-%3F%3F%3F-activity-7141752823460896768-MHJe?utm_source=share&utm_medium=member_desktop)
