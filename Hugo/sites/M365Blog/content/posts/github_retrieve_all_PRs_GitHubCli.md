---
title: "Get a CSV of all my pull requests from Github using Github CLI and PowerShell"
date: 2023-07-11T17:00:15+01:00
tags: ["GitHub", "CLI","PowerShell"]
draft: false
---

Github does not provide an easy way to export all your pull requests for review or sharing with anyone. Below is a great post to show how to do it using BASH. 

[GitHub: Get a CSV containing my pull requests (PRs)](https://www.markhneedham.com/blog/2023/06/12/github-list-pull-requests-csv/)

If you are using windows machine, PowerShell is your friend. 

Install [GitHub CLI](https://github.com/cli/cli) via command line

```dos
winget install --id GitHub.cli
```
Enter Y to the question

Do you agree to all the source agreements terms?
[Y] Yes  [N] No: Y

Message "Successfully installed" will display once installed.
 ![Install GitHubCli](../images/github_retrieve_all_PRs_GitHubCli/InstallGithubCLI.png)

Open PowerShell and authenticate with the cmdlet
```powershell
gh auth login
```
![Device Login](../images/github_retrieve_all_PRs_GitHubCli/gh_auth_login.png)

Open this URL to continue in your web browser: https://github.com/login/device. Enter the code generated from the PowerShell cmdlet above.

Authorize github to trust the device to access your account.
![Authorise GitHubCli](../images/github_retrieve_all_PRs_GitHubCli/AuthoriseGithubCLI.png)

Run the search query against github and export csv file

```powershell
(gh search prs --author=@me --json title,repository,closedAt,url --limit 100 | ConvertFrom-Json) | Export-Csv -Path c:\temp\contributions.csv -NoTypeInformation -Force
```
![Sample Output](../images/github_retrieve_all_PRs_GitHubCli/SampleOutput.png)

[Examples using gh search prs](https://cli.github.com/manual/gh_search_prs)

You can also search for all your commits if you commit directly to the main branch

```powershell
(gh search commits --author=@me --author-date=>"2022-06-01" --order desc --sort author-date --visibility public --json repository,sha,commit  --limit 100 | ConvertFrom-Json) |
    ForEach-Object {
        [PsCustomObject]@{
            'commit.author.date' = $_.commit.author.date
            'commit.message' = $_.commit.message
            'repository.fullName' = $_.repository.fullname
        }
    } | Export-Csv -Path c:\temp\commits.csv -NoTypeInformation -Force 
```