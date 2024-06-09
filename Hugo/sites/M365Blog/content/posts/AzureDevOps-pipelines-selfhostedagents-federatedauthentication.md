---
title: "Overcoming challenges with Azure DevOps Pipelines using Self-Hosted Build Agents for Power Platform Managed Solutions"
date: 2024-06-08T10:56:16+01:00
tags: ["Azure","DevOps","self-hosted Agents","Federated authentication","PAT","git","git fetch","git switch", "git checkout","Power Platform"]
featured_image: '/posts/images/AzureDevOps-pipelines-selfhostedagents-federatedauthentication/selfhostedagent.png'
draft: false
---

# Overcoming Challenges with Azure DevOps Pipelines using Self-Hosted Build Agents for Power Platform Managed Solutions

Application Lifecycle Management (ALM) for Power Platform solutions can be effectively managed using Azure DevOps, providing a robust framework for automating deployments, version control, and continuous integration and delivery, thereby enhancing productivity and reducing manual errors. Refer to the posts for more details: [Power Platform ALM & Pipelines w/ Matt Devaney](https://www.youtube.com/watch?v=wQe7n62RRNU) and [Converting to Modern YAML Pipeline: Application Lifecycle Management in Azure DevOps for Power Platform](https://reshmee.netlify.app/posts/powerplatform-convert-classic-pipeline-to-modern-pipeline/)

ALM depends on build agents : Microsoft host agents and self-hosted agents. Self-hosted agents and Microsoft-hosted agents each have their own advantages, depending on your specific needs and circumstances. This post focuses on using self-hosted agent hence sepcifying the benefits of using self-hosted agents:

![self-hosted pool agent](../images/AzureDevOps-pipelines-selfhostedagents-federatedauthentication/selfhostedagent.png)

**Control**: With self-hosted agents, there is more control over the environment. The hardware, operating system, and installed software can be chosen and the configuration fine-tuned to meet specific needs.

**Performance**: Since the agent is dedicated to specific tasks, there might beimproved performance, especially for larger projects or more complex builds that require significant resources.

**Security**: Self-hosted agents can be set up within own network, allowing handling of sensitive code or data without exposing it to external networks. This can be crucial for projects with strict security requirements.

**Availability**: Unlike Microsoft-hosted agents, which can occasionally run out of capacity during peak times, self-hosted agents are always available for builds and releases.

**Remove the need to use PAT**: Self-hosted agents can use service principal, which can be more secure and convenient than using Personal Access Tokens (PATs). PATs are linked to individual account and expired after a certain time depending on the orginisation's policy and need to be regenerated which can be a pain.

- [Register an agent using a service principal](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/service-principal-agent-registration?view=azure-devops?wt.mc_id=MVP_308367)
- [Running Azure DevOps Self-hosted agent on AKS without using PAT](https://eggboy.medium.com/running-azure-devops-self-hosted-agent-on-aks-without-using-pat-1b90f714c147)
- [Using Managed Identity in Azure DevOps Pipeline with Federated Identity](https://eggboy.medium.com/using-managed-identity-in-azure-devops-pipeline-with-federated-identity-72813873b933)

However, transitioning to self-hosted agents with managed identity can introduce challenges not encountered with Microsoft Hosted Agents. This article discusses these challenges and provides solutions to overcome them.

## Challenges

1. **Build Identity Missing Permissions**: Ensure the project admin grants the build identity the correct permissions to check in the solution to the Azure DevOps remote repository otherwise you may get "Generic Contribute" error message.

![Build identity missing contribute permissions](../images/AzureDevOps-pipelines-selfhostedagents-federatedauthentication/MissingPermission_Contribute_ForBuildIdentity.png)

1. **Copy the GUID**: First, copy the GUID part of the identity name from the error message.
2. **Navigate to Repository Settings**: Go to your repository settings in Azure DevOps.
3. **Search for the GUID**: In the permissions section, search for the GUID copied earlier in the “Search for users and groups” field.
4. **Select the Correct User**: A user named “Project Collection Build Service (OrganizationName).” will be returned. Select this user.
5. **Set Permissions**: Set the necessary permissions for this user according to requirements. In my scenerio, the permissions highlighted were granted.

![Grant Contribute permissions](../images/AzureDevOps-pipelines-selfhostedagents-federatedauthentication/MissingPermission_Contribute_ForBuildIdentity.png)

2. **Artefacts Not Cleaned by Default**: Add a delete file action to remove all artefacts before starting the build. This ensures that artefacts from previous builds are not left in the working directory, otherwise deleted artefacts from your solution will remain in the build directory.

![Delete files from Build working directory](../images/AzureDevOps-pipelines-selfhostedagents-federatedauthentication/DeleteFilesfromBuild.png)

3. **Failure to `git push` without `git pull`**: Always perform a `git pull` before a `git push` to avoid conflicts. **Updates were rejected because a pushed branch tip is behind its remote**

![Attempt to Push Without Pull](../images/AzureDevOps-pipelines-selfhostedagents-federatedauthentication/AttempttoPushWithoutPull.png)

4. **Failure to Merge**: Use `git pull --no-rebase` to fetch changes from the remote and create a merge commit, preserving your local commit history maintaining a clear, linear history. `git pull` defaults to `rebase` which will fetch any changes to the tracking branch and then rebase any local changes on top.

![failed to merge](../images/AzureDevOps-pipelines-selfhostedagents-federatedauthentication/FailedToMerge.png)

5. **Force Push Failure Due to Lack of Permission**: Ensure the user has force push permission on the branch. Use the --force option judiciously as it overwrites all changes from the remote with the local files.

**(TF401027: You need the Git 'ForcePush' permission to perform this action. Details: identity 'Build\92e00 error: failed to push some refs to 'https://dev.azure.com/contoso/test/_git/test'**

![Force Push permission missing](../images/AzureDevOps-pipelines-selfhostedagents-federatedauthentication/ForcePushPermission_missing.png)

Amending branch security was not an option to allow force push and has to find alternate appropriate solutions. 

6. **Pushing Changes Without Using the --force with Unsuccessful Pull**: Always ensure a successful pull before attempting to push changes.

![Successful check in](../images/AzureDevOps-pipelines-selfhostedagents-federatedauthentication/All_success.png)

## Solution

The Azure DevOps issues with pull and push were resolved using the **Clean** setting in the pipeline settings UI if using self-hosted agents. 

### Classic pipeline

For classic pipeline, this can be done from the user interface. 

![Clean options](../images/AzureDevOps-pipelines-selfhostedagents-federatedauthentication/CleanOptions_Classic.png)

**You can perform different kinds of cleaning of the working directory of your private agent before the build is run.**

### YAML pipeline

If using YAML, add a clean job action to ensure a clean workspace repository.

```yaml
- job: myJob
  workspace:
    clean: outputs | resources | all # what to clean up before the job runs
```

When you specify one of the clean options, they're interpreted as follows:

* outputs: Delete Build.BinariesDirectory before running a new job.
* resources: Delete Build.SourcesDirectory before running a new job.
* all: Delete the entire Pipeline.Workspace directory before running a new job.

Jobs are always run on a new agent with Microsoft-hosted agents hence the **Clean** is helpful for self-hosted agents

[Read more about clean options from 'Specify jobs in your pipeline'](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/phases?view=azure-devops&tabs=yaml#workspace&wt.mc_id=MVP_308367)

In addition to workspace clean, the **Clean** can be set for the pipeline. From the UI, edit the pipeline and select triggers. Select YAML, Get Sources and configure **Clean** setting. 

![Get Sources](../images/AzureDevOps-pipelines-selfhostedagents-federatedauthentication/GetSources_YAML.png)

### Clean setting

When the **Clean** setting is true, which is also its default value, it's equivalent to specifying **clean: true** for every checkout step in the pipeline. **Clean: true** is equivalent to running **git clean -ffdx** followed by git **reset --hard HEAD** before **git fetching**. All the issues I was having was because **Clean** option was set to false for some reasons.  

- **git clean -ffdx**: This command is used to remove untracked files and directories from your working directory. The -f option stands for 'force', which is required to make git clean do anything in most cases. The second -f is used to force cleaning of directories even if they have a .gitignore file. The -d option tells Git to remove untracked directories as well. The -x option makes Git ignore the standard ignore rules (.gitignore files, .git/info/exclude, and core.excludesFile), thus removing all untracked files.

- **git reset --hard HEAD**: This command resets your current branch and working directory to the state of the HEAD commit. The --hard option changes all the files in your working directory to match the files in HEAD. This will discard all changes since the last commit and all uncommitted changes.

- **git fetch**: This command downloads objects and refs from another repository. It's often used to review changes that have been made to the remote repository. Unlike git pull, git fetch does not merge any changes into your current branch.

## Final code for Check in Solution action for the build pipeline

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

## Conclusion

Optimizing Azure DevOps Pipelines with self-hosted build agents for Power Platform managed solutions can significantly enhance the efficiency and control of your Application Lifecycle Management (ALM). While transitioning to self-hosted agents with Federated Authentication can introduce certain challenges, the solutions provided in this post can help overcome them effectively.

## References

[Specify jobs in your pipeline](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/phases?view=azure-devops&tabs=yaml#workspace&wt.mc_id=MVP_308367)

[Power Platform ALM & Pipelines w/ Matt Devaney](https://www.youtube.com/watch?v=wQe7n62RRNU)

[Converting to Modern YAML Pipeline: Application Lifecycle Management in Azure DevOps for Power Platform](https://reshmee.netlify.app/posts/powerplatform-convert-classic-pipeline-to-modern-pipeline/)

[What is Azure Pipelines?](https://learn.microsoft.com/en-gb/training/modules/create-a-build-pipeline/2-what-is-azure-pipelines?pivots=ms-hosted-agents)

[Connect to Azure by using an Azure Resource Manager service connection](https://learn.microsoft.com/en-gb/azure/devops/pipelines/library/connect-to-azure?view=azure-devops#create-an-azure-resource-manager-service-connection-using-workload-identity-federation)

[Deploy SPFx app using pipeline's Workload Identity federation](https://dev.to/kkazala/deploy-spfx-app-using-pipelines-workload-identity-federation-5fhi)