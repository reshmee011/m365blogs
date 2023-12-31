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

To set up the development to start hacking on PnP PowerShell, install the following

1. Install [Git](https://git-scm.com/downloads)
2. Install [PowerShell 7](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell)
3. Install [Visual Code](https://code.visualstudio.com) or **Visual Studio Code**
4. Install .NET SDK 6 (https://dotnet.microsoft.com/download/dotnet/6.0)

 
## Forking and cloning 

First things first, make a copy of the repository to start working on your changes. While making minor changes via GitHub's web editor or Codespaces is an option, I prefer having the repository on my local machine for more flexibility.

### Fork Your Repository: This creates your copy of the repository.

Navigate to [PnP PowerShell](https://github.com/pnp/powershell) and fork the repository

![Fork your own repository](../images/PnPPowerShell_StartingToContribute/ForkYourOwnRepository.png)

### Clone the Repository 

Clone the forked repository onto your local machine using the following commands

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

Setting up an upstream allows you to sync changes with the source repository. Create a seperate branch for each feature or issue to work on as it will make easier to create a pull request for the individual feature or issue. 

```powershell
# Configure an upstream so you can sync changes with the source repo
git remote add upstream https://github.com/pnp/powershell

# Create a branch for your issue or feature and check it out
git checkout -b command-xyz
```

## Start hacking

With your environment set up, dive into making your changes!

Explore the [Folder structure](https://pnp.github.io/powershell/articles/buildingfolderstructure.html) to grasp the project's layout.

If a cmdlet is created or updated the corresponding documentation needs to be created or updated to reflect the changes. 

Remember, whenever a cmdlet is added or modified, it's vital to mirror those changes in the documentation. For instance, upon introducing a new cmdlet like Remove-PnPContainer for container deletion, ensure that corresponding .cs and .md files are created or updated to document its functionality.

## Debug and Test changes

1. **Builds a debug version** : Use PowerShell 7 or higher to build a debug version for local testing

```powershell
c:\Users\reshm\source\repos\powershell\build\Build-Debug.ps1
```

![Build debug successful](../images/PnPPowerShell_StartingToContribute/BuildDebug.png)

Build debug may fail if PowerShell file is already in use
![Build debug failed](../images/PnPPowerShell_StartingToContribute/BuildFailedBecauseofFileInUse.png)

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
![Breakpointhit](../images/PnPPowerShell_StartingToContribute/hitdebugger.png)

## Commit Changes

Once your changes are ready, run the following cmdlets to commit all changes changing the message to reflect the commit.

```powershell
# Edit files, add and commit. Then push with the -u (short for --set-upstream) option
 git add -A
 git commit -m "new cmdlet for remove-pnpcontainer"
 git push --set-upstream origin command-xyz 
``` 

Once above cmdlets are run , you may be prompted to create pull request from the branch created.

![Publish branch with changed](../images/PnPPowerShell_StartingToContribute/CreateAndPublishbranch.png)

## Create Pull request 

Sync your branch with the main repository and ensure it's up to date:

### To ensure latest before creating a PR

The code below is a set of Git commands used to synchronize your local repository with the main repository before creating a Pull Request (PR). This is an important step to ensure that your changes are based on the most recent version of the project.

```powershell
# Sync and rebase main branch to get the latest changes
#git fetch upstream, retrieves all the branches and their respective commits from the 'upstream' repository, which is typically the original repository you forked. However, this command doesn't merge any changes into your local branches.
git fetch upstream
#switch to the 'dev' branch
git checkout dev
#pulls the latest changes from the 'dev' branch of the upstream repository and rebases your local 'dev' branch on top of those changes. Rebasing is a way to integrate changes from one branch into another. It moves or combines a sequence of commits to a new base commit.
git pull --rebase upstream dev

# Rebase feature branch to get the latest changes
#switch to feature branch
git checkout command-xyz
#git rebase dev is used to rebase your feature branch onto the 'dev' branch. This means that the changes in your feature branch are applied on top of the changes in the 'dev' branch, ensuring that your feature branch is up-to-date with the latest changes in the 'dev' branch.
git rebase dev

#push all local changes back onto the origin 
#Modifies the most recent commit with any staged changes and/or updates the commit message. In this case, --no-edit is used to keep the existing commit message.
git commit --amend --no-edit
#This command updates your current branch with any new changes from the remote repository.
git pull
# This command pushes all local changes in your current branch to the remote repository.
git push
```

![rebase](../images/PnPPowerShell_StartingToContribute/RebaseCode.png)

After running these commands, you can navigate to the branch on your browser and create a pull request. This pull request should reference your feature branch and the main(dev) branch of the central repository.

![branch with commits ahead](../images/PnPPowerShell_StartingToContribute/branchwithcommitahead.png)

### Squash Multiple Commits 

You may have more than 1 commit ahead of the upstream repository and it is a good idea to squash multiple commits into one for a cleaner history:

### Git rebase

![3commits ahead](../images/PnPPowerShell_StartingToContribute/3CommitsAhead.png)

```powershell
# -i stands for 'interactive rebase'. HEAD~n is a reference to the 'n-th' commit before the current commit (HEAD). For example, if 'n' is 3, this command will let you modify the last three commits.
git rebase -i HEAD~n
```

Interactive rebasing allows you to modify commits as they are moved to the new base commit. This is particularly useful when you want to clean up a series of commits before merging them into a main branch.

Git opens a text editor with a list of the last 'n' commits, each on a separate line, and some commands for how to handle each commit. You can change these commands to tell Git what to do with each commit. For example, you can squash commits (combine them), reorder them, modify the commit messages, or even delete them. 

![Interactive rebasing](../images/PnPPowerShell_StartingToContribute/InteractiveRebasingForLast3Commits.png)

After you save and close the file. Please note the command to save and close depends on the tool the text editor was opened. In my scenerio using git via the terminal from visual studio code opened the editor in vim. 

*** How to Exit Vim ***

1. Press ESC once (sometimes twice)
2. Type :q and press Enter/return if no changes
3. Type :wq and press Enter/return to keep changes
4. Type :q! and press Enter/return to discard any changes made.

 Git will apply the changes you've specified. View post [Git Tools - Rewriting History](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History) for more details.

![Rebased and Updated](../images/PnPPowerShell_StartingToContribute/RebasedandUpdated.png)

This can help you create a clean, understandable commit history.
Once ready, create the pull request via the GitHub site and fill in the details.

### Git reset

An alternative to rebase is to use git reset soft, view [difference between git reset hard and git reset soft](https://stackoverflow.com/questions/24568936/what-is-difference-between-git-reset-hard-head1-and-git-reset-soft-head) for more details.

```PowerShell
git reset --soft HEAD~7
git add -A
git commit -m "new message"
git pull
git push
```

Outcome of above command
![Git Reset Soft](../images/PnPPowerShell_StartingToContribute/GitResetSoft.png)

After the rebase or reset, push the changes to the remote repository with the git push command.

```powershell
 git push –force-with-lease 
```

The –force-with-lease flag confirms whether the remote branch matches the version of the one you are merging. It ensures that any new commits pushed in the interim are acknowledged and rejects the push if the remote branch has been modified in between.

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

Once you have made minor changes to a repository, you are ready to amend your commit. You can do this by using the –no-edit flag:

```powershell
git add -A
git commit --amend --no-edit
git pull
git push
```

Please check [Getting Started with Git](https://www.linkedin.com/posts/brijpandeyji_getting-started-with-git-%3F%3F%3F-%3F%3F%3F-activity-7141752823460896768-MHJe?utm_source=share&utm_medium=member_desktop)
for helpful git cmdlets you may use.

## Conclusion

Contributing to PnP PowerShell is a learning process. Remember, each contribution counts, and this guide is your roadmap to make impactful changes seamlessly. Happy coding!

## References

[Rewriting history](https://www.atlassian.com/git/tutorials/rewriting-history#:~:text=To%20review%2C%20git%20commit%20%2D%2D,the%20last%20commit%20message%20log)

[Contributing as a holiday season present](https://www.blimped.nl/contributing-as-a-holiday-season-present/)

[Contribution guidance](https://pnp.github.io/powershell/articles/gettingstartedcontributing.html)

[GitHub Universe Cloud Skills Challenge](https://learn.microsoft.com/en-gb/collections/kkqrhmxoqn54?WT.mc_id=cloudskillschallenge_ef5f9f41-0818-4895-9217-79d19827a322)

[Getting Started with Git-](https://www.linkedin.com/posts/brijpandeyji_getting-started-with-git-%3F%3F%3F-%3F%3F%3F-activity-7141752823460896768-MHJe?utm_source=share&utm_medium=member_desktop)

[What Are Squash Commits in Git: A How-To Guide for 2024](https://www.redswitches.com/blog/squash-commits/)