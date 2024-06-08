---
title: "Optimizing AzureDevOps Pipelines with Self-Hosted Build Agents for Power Platform Managed Solutions"
date: 2024-06-07T10:56:16+01:00
tags: ["AzureDevOps","Self hosted Agents","Federated authentication","PAT","github","git","git fetch","git switch", "git checkout"]
featured_image: '/posts/images/SharePoint-Restricted-SharePoint-Search/rss_enabled.png'
draft: true
---

# Optimizing AzureDevOps Pipelines with Self-Hosted Build Agents for Power Platform Managed Solutions

Application Lifecycle Management (ALM) for Power Platform solutions can be effectively managed using Azure DevOps, providing a robust framework for automating deployments, version control, and continuous integration and delivery, thereby enhancing productivity and reducing manual errors. Refer to the posts for more details: [Power Platform ALM & Pipelines w/ Matt Devaney](https://www.youtube.com/watch?v=wQe7n62RRNU) and [Converting to Modern YAML Pipeline: Application Lifecycle Management in Azure DevOps for Power Platform](https://reshmee.netlify.app/posts/powerplatform-convert-classic-pipeline-to-modern-pipeline/)

ALM depends on build agents : Microsoft host agents and self hosted agents. Self-hosted agents and Microsoft-hosted agents each have their own advantages, depending on your specific needs and circumstances. This post focuses on using self-hosted agent hence sepcifying the benefits of using self-hosted agents:

**Control**: With self-hosted agents, there is more control over the environment. The hardware, operating system, and installed software can be chosen and the configuration fine-tuned to meet specific needs.

**Performance**: Since the agent is dedicated to specific tasks, there might beimproved performance, especially for larger projects or more complex builds that require significant resources.

**Security**: Self-hosted agents can be set up within own network, allowing handling of sensitive code or data without exposing it to external networks. This can be crucial for projects with strict security requirements.

**Availability**: Unlike Microsoft-hosted agents, which can occasionally run out of capacity during peak times, self-hosted agents are always available for builds and releases.

**Federated Authentication**: Self-hosted agents can use federated authentication, which can be more secure and convenient than using Personal Access Tokens (PATs). Read more using the links below

- [Running Azure DevOps Self-hosted agent on AKS without using PAT](https://eggboy.medium.com/running-azure-devops-self-hosted-agent-on-aks-without-using-pat-1b90f714c147)
- [Using Managed Identity in Azure DevOps Pipeline with Federated Identity](https://eggboy.medium.com/using-managed-identity-in-azure-devops-pipeline-with-federated-identity-72813873b933)
- [Register an agent using a service principal](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/service-principal-agent-registration?view=azure-devops?wt.mc_id=MVP_308367)

However, transitioning to self-hosted agents with Federated Authentication can introduce challenges not encountered with Microsoft Hosted Agents. This article discusses these challenges and provides solutions to overcome them.

## Challenges

1. **Build Identity Missing Permissions**: Ensure the project admin grants the build identity the correct permissions to check in the solution to the Azure DevOps remote repository otherwise you may get "Generic Contribute" error message.

![Build identity missing contribute permissions](../images/MissingPermission_Contribute_ForBuildIdentity.png)

2. **Artefacts Not Cleaned by Default**: Add a delete file action to remove all artefacts before starting the build. This ensures that artefacts from previous builds are not left in the working directory, otherwise deleted artefacts from your solution will remain in the build directory.

![Delete files from Build working directory](../images/DeleteFilesfromBuild.png)

3. **Failure to `git push` without `git pull`**: Always perform a `git pull` before a `git push` to avoid conflicts. **Updates were rejected because a pushed branch tip is behind its remote**

![Attempt to Push Without Pull](../images/AttempttoPushWithoutPull.png)

4. **Failure to Merge**: Use `git pull --no-rebase` to fetch changes from the remote and create a merge commit, preserving your local commit history maintaining a clear, linear history. `git pull` defaults to `rebase` which will fetch any changes to the tracking branch and then rebase any local changes on top.

![failed to merge](../images/AzureDevOps-pipelines-git-checkinsolution-issuespullandpushFailedToMerge.png)

5. **Force Push Failure Due to Lack of Permission**: Ensure the user has force push permission on the branch. Use the --force option judiciously as it overwrites all changes from the remote with the local files.

**(TF401027: You need the Git 'ForcePush' permission to perform this action. Details: identity 'Build\92e00 error: failed to push some refs to 'https://dev.azure.com/contoso/test/_git/test'**

![Force Push permission missing](../images/AzureDevOps-pipelines-git-checkinsolution-issuespullandpush/ForcePushPermission_missing.png)

Amending branch security was not an option to allow force push and has to find alternate appropriate solutions. 

6. **Pushing Changes Without Using the --force with Unsuccessful Pull**: Always ensure a successful pull before attempting to push changes.

![Successful check in](../images/AzureDevOps-pipelines-git-checkinsolution-issuespullandpush/All_success.png)

## Solution

The Github issues with pull and push were resolved using the clean option in the build pipeline. For classic pipeline, you can do it from the user interface. 

![Clean options](../images/AzureDevOps-pipelines-git-checkinsolution-issuespullandpush/CleanOptions_Classic.png)

If using YAML, add a clean job GitHub action to ensure a clean working repository.

```yaml
- job: myJob
  workspace:
    clean: outputs | resources | all # what to clean up before the job runs
```

When you specify one of the clean options, they're interpreted as follows:

* outputs: Delete Build.BinariesDirectory before running a new job.
* resources: Delete Build.SourcesDirectory before running a new job.
* all: Delete the entire Pipeline.Workspace directory before running a new job.

[Read more about clean options from 'Specify jobs in your pipeline'](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/phases?view=azure-devops&tabs=yaml#workspace&wt.mc_id=MVP_308367)

## Final code for Check in Solution github action

```powerShell
# Display the current branch
git branch --show-current

# Set the user email and name for the commit
echo Commit Solution into GIT Branch $(BranchName)
git config user.email "srv_d_test@ppf.co.uk"
git config user.name "test Service Account (Dev)"

# Switch to the specified branch, creating it if it does not exist
git switch --force-create  $(BranchName) 

# Pull the latest changes from the origin without rebasing
git pull origin $(BranchName) --no-rebase

# Add all changes to the staging area
echo Adding all files to $(BranchName)
git add --all

# Commit the changes with a message
git commit -m "Checked in by Power Platform Release"

# Push the committed changes to the specified branch on the origin
echo Pushing solution components to $(BranchName)
git push origin HEAD:$(BranchName) --force
```

## References

[Specify jobs in your pipeline](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/phases?view=azure-devops&tabs=yaml#workspace&wt.mc_id=MVP_308367)

[Power Platform ALM & Pipelines w/ Matt Devaney](https://www.youtube.com/watch?v=wQe7n62RRNU)

[Converting to Modern YAML Pipeline: Application Lifecycle Management in Azure DevOps for Power Platform](https://reshmee.netlify.app/posts/powerplatform-convert-classic-pipeline-to-modern-pipeline/)
