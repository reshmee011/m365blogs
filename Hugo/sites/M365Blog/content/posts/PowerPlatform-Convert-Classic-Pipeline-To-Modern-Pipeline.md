---
title: "Add or Update Image Column in SharePoint with Power Automate: Download and Use Images from the Web"
date: 2023-05-28T06:56:03+01:00
author: "Reshmee Auckloo"
githubname: reshmee011
categories: ["Post"]
images:
- images:Update-Image-Column-PowerAutomate/images/PowerAutomate.png
tags: ["Image, PowerAutomate","SharePoint List"]
type: "regular"
draft: false
---


# Convert Classic Pipeline to Modern Pipeline using YAML for Application Lifecycle Management in Azure DevOps

There are loads of posts explaining Application Lifecycle Management for the Power Platform using Azure DevOps all using the graphical classical pipeline.
The latest post I read on this is [Application Lifecycle Management for the Power Platform using Azure DevOps](https://www.jondoesflow.com/post/application-lifecycle-management-for-the-power-platform-using-azure-devops) which is brilliant. 

An interesting article covering difference between [YAML and Classic UI](https://medium.com/@wywywywy/azure-devops-pipeline-choosing-between-yaml-and-classic-ui-b5612c3e211a) 

The current post will focus on converting the classic pipeline into YAML separated into two categories : build and release pipeline

## Build pipeline


[View YAML]
![View YAML](../images/PowerPlatform-Convert-Classic-Pipeline-To-Modern-Pipeline/Build_ViewYAML.png)

![CopyToClipboard](../images/PowerPlatform-Convert-Classic-Pipeline-To-Modern-Pipeline/Build_CopyToClipboard.png)


1. Create a yaml file with extension .yml in your chosen repository 
2. Paste the copied yaml code and commit to your repository
    Edit the YAML
      Replace "microsoft-IsvExpTools.PowerPlatform-BuildTools." with "". 
  
3. Click on Pipelines from the left navigation and click on "New Pipeline"
![Click on Pipeline](../images/PowerPlatform-Convert-Classic-Pipeline-To-Modern-Pipeline/Build_Pipelines.png)

4. Clik on new Pipeline and select "Azure Repos Git" from option "Where is your code?"  

![Where is your code?](../images/PowerPlatform-Convert-Classic-Pipeline-To-Modern-Pipeline/CodeLocation.png)

5. Select the repository where the YAML is

6. Pick the option Existing Azure Pipelines YAML file, pick the branch and Path where the build yaml file is 

![Configure](../images/PowerPlatform-Convert-Classic-Pipeline-To-Modern-Pipeline/Configure_AzurePipelines.png)

7. Modify the yaml file as appropriate adding any trigger actions ensuring right formatting and indentation is used 

```yml
trigger:
  branches:
    include:
    - main
    - develop
pool:
  name: Azure Pipelines
#Your build pipeline references a secret variable named ‘TEST_PAT’. Create or edit the build pipeline for this YAML file, define the variable on the Variables tab, and then select the option to make it secret. See https://go.microsoft.com/fwlink/?linkid=865972
variables:
- name: SolutionName
  value: TEST
- name: SolutionConnectorName
  value: domainnameBasicDisplay
steps:
- task: PowerPlatformToolInstaller@2
  displayName: 'Power Platform Tool Installer '
- task: PowerPlatformExportSolution@2
  displayName: 'Power Platform Export TEST Unmanaged Solution'
  inputs:
    authenticationType: PowerPlatformSPN
    PowerPlatformSPN: 'powerplatform-d-connection'
    Environment: 'https://domainname-dev.crm11.dynamics.com'
    SolutionName: $(SolutionName)
    SolutionOutputFile: '$(Build.ArtifactStagingDirectory)/PowerPlatformSolution/$(SolutionName).zip'
- task: PowerPlatformExportSolution@2
  displayName: 'Power Platform Export CustomConnector Unmanaged Solution'
  inputs:
    authenticationType: PowerPlatformSPN
    PowerPlatformSPN: 'powerplatform-d-connection'
    Environment: 'https://domainname-dev.crm11.dynamics.com'
    SolutionName: $(SolutionConnectorName)
    SolutionOutputFile: '$(Build.ArtifactStagingDirectory)/PowerPlatformSolution/$(SolutionConnectorName).zip'
- task: PowerPlatformUnpackSolution@2
  displayName: 'Power Platform Unpack TEST Unmanaged Solution '
  inputs:
    SolutionInputFile: '$(Build.ArtifactStagingDirectory)/PowerPlatformSolution/$(SolutionName).zip'
    SolutionTargetFolder: '$(Build.SourcesDirectory)/PowerPlatformSolution/$(SolutionName)'
- task: PowerPlatformUnpackSolution@2
  displayName: 'Power Platform Unpack CustomConnector Unmanaged Solution '
  inputs:
    SolutionInputFile: '$(Build.ArtifactStagingDirectory)/PowerPlatformSolution/$(SolutionConnectorName).zip'
    SolutionTargetFolder: '$(Build.SourcesDirectory)/PowerPlatformSolution/$(SolutionConnectorName)'
- task: PowerPlatformExportSolution@2
  displayName: 'Power Platform Export TEST Managed Solution'
  inputs:
    authenticationType: PowerPlatformSPN
    PowerPlatformSPN: 'powerplatform-d-connection'
    Environment: 'https://domainname-dev.crm11.dynamics.com'
    SolutionName: $(SolutionName)
    SolutionOutputFile: '$(Build.ArtifactStagingDirectory)/PowerPlatformSolution/$(SolutionName).zip'
    Managed: true
- task: PowerPlatformExportSolution@2
  displayName: 'Power Platform Export CustomConnector Managed Solution'
  inputs:
    authenticationType: PowerPlatformSPN
    PowerPlatformSPN: 'powerplatform-d-connection'
    Environment: 'https://domainname-dev.crm11.dynamics.com'
    SolutionName: $(SolutionConnectorName)
    SolutionOutputFile: '$(Build.ArtifactStagingDirectory)/PowerPlatformSolution/$(SolutionConnectorName).zip'
    Managed: true
- task: PowerPlatformUnpackSolution@2
  displayName: 'Power Platform Unpack TEST Managed Solution '
  inputs:
    SolutionInputFile: '$(Build.ArtifactStagingDirectory)/PowerPlatformSolution/$(SolutionName).zip'
    SolutionTargetFolder: '$(Build.SourcesDirectory)/PowerPlatformSolution/$(SolutionName)_Managed'
    SolutionType: Managed
- task: PowerPlatformUnpackSolution@2
  displayName: 'Power Platform Unpack CustomConnector Managed Solution '
  inputs:
    SolutionInputFile: '$(Build.ArtifactStagingDirectory)/PowerPlatformSolution/$(SolutionConnectorName).zip'
    SolutionTargetFolder: '$(Build.SourcesDirectory)/PowerPlatformSolution/$(SolutionConnectorName)_Managed'
    SolutionType: Managed
- script: |
   echo Commit Solution
   git config user.email srv_d_TEST@domainname.co.uk
   git config user.name "Service Account (Dev)"
   git switch -c feature/-listview
   git pull 
   git add --all
   git commit -m"Checked in by Power Platform Release"
   echo Push Solution to Repo
   git push --all https://$(TEST_PAT)@dev.azure.com/domainname-it/M365/_git/$(SolutionName)
  displayName: 'Check in files'
```

Let's break down the different sections and their meanings:

branches: This section specifies the branches that will trigger the build pipeline. In this case, the build will be triggered when changes are made to the main and develop branches.

pool: The pool section defines the agent pool to use for the build. In this case, the build will use the "Azure Pipelines" agent pool.

variables: This section defines the variables used in the pipeline. Two variables are defined: SolutionName with a value of "TEST" and SolutionConnectorName with a value of "domainnameBasicDisplay". These variables can be referenced later in the pipeline using the syntax $(VariableName).

steps: The steps section contains the sequence of tasks to be executed in the pipeline.

    1.The first task is PowerPlatformToolInstaller@2, which installs the Power Platform tools.

    2. The next task is PowerPlatformExportSolution@2, which exports an unmanaged solution from the Power Platform. It exports a solution named $(SolutionName) using the specified authenticationType which is PowerPlatformSPN, and Environment. In the above example, there are two tasks for PowerPlatformExportSolution@2 for two different solution

    3. PowerPlatformUnpackSolution@2 tasks, which unpack the previously exported solutions. They extract the solution files to specific target folders based on the solution names.

    4. There are two more PowerPlatformExportSolution@2 tasks, similar to the previous ones, but these export managed solutions instead of unmanaged solutions.

    5. There are two tasks are PowerPlatformUnpackSolution@2 tasks that unpack the managed solutions to target folders with a "_Managed" suffix in their names.
    
    6. The final task is a script task that performs a series of Git commands. It configures the user email and name, creates a new branch, pulls the latest changes, adds all the files, commits the changes with a specific message, and pushes the changes to a remote repository. The repository URL is constructed using the TEST_PAT secret variable definied as a variable. The PAT is the Azure DevOps personal access token which can be generated from your account settings.
    

It's important to note that the YAML trigger only describes the build pipeline configuration. To use this trigger, you would need to create or edit the build pipeline in Azure DevOps, define the necessary secret variable, and configure the required resources and connections accordingly.

![PAT](../images/PowerPlatform-Convert-Classic-Pipeline-To-Modern-Pipeline/Build_PAT.png)

8. Create a variable for PAT
![PAT](../images/PowerPlatform-Convert-Classic-Pipeline-To-Modern-Pipeline/Build_PATVariable.png)


1. Review Pipeline and click on Run 

![Run Pipeline](../images/PowerPlatform-Convert-Classic-Pipeline-To-Modern-Pipeline/Build_Run.png)

Grant permission if prompted
![Grant Permission](../images/PowerPlatform-Convert-Classic-Pipeline-To-Modern-Pipeline/GrantPermission.png)

If all successful 
![Build Sucess](../images/PowerPlatform-Convert-Classic-Pipeline-To-Modern-Pipeline/Build_RunSuccess.png)

## Release Pipeline

Classic pipeline, no option to view YAML

With 4 tasks



 

 

Unfortunately the view YAML is only available on each task and not on the entire release pipeline

 



 

You can copy each YAML for each task onto a new release pipeline , editing as appropriately

 

name: $(TeamProject)_$(BuildDefinitionName)_$(SourceBranchName)_$(Date:yyyyMMdd)$(Rev:.r)

 

trigger:

- none

pool:

  name: Azure Pipelines

variables:

- name: SolutionName

  value: TEST

stages:

- stage: deploytest

  displayName: Deploy to Test

  jobs:

  - deployment: deploy_to_Test 

    environment: ST Intranet

    strategy:

      runOnce:

        deploy:

          steps:

          - checkout: self

          - task: PowerPlatformToolInstaller@2

            inputs:

              DefaultVersion: true

          - task: PowerPlatformPackSolution@2

            inputs:

              SolutionSourceFolder: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/$(SolutionName)_Managed'

              SolutionOutputFile: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/$(SolutionName)_Managed.zip'

              SolutionType: Managed

          - task: PowerPlatformImportSolution@2

            inputs:

              authenticationType: PowerPlatformSPN

              PowerPlatformSPN: 'powerplatform-t-connection'

              Environment: 'https://domainname-st.crm11.dynamics.com'

              SolutionInputFile: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/$(SolutionName)_Managed.zip'

              UseDeploymentSettingsFile: true

              DeploymentSettingsFile: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/settings-test.json'

              ConvertToManaged: true

              PublishCustomizationChanges: true

              MaxAsyncWaitTime: "60"

          - task: PowerPlatformPublishCustomizations@2

            inputs:

              authenticationType: 'PowerPlatformSPN'

              PowerPlatformSPN: 'powerplatform-t-connection'

- stage: deployUAT

  displayName: Deploy to UAT

  jobs:

   - deployment: deploy_to_Uat 

     environment: UAT Intranet

     strategy:

      runOnce:

       deploy:

        steps:

        - checkout: self

        - task: PowerPlatformToolInstaller@2

          inputs:

            DefaultVersion: true

        - task: PowerPlatformPackSolution@2

          inputs:

            SolutionSourceFolder: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/$(SolutionName)_Managed'

            SolutionOutputFile: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/$(SolutionName)_Managed.zip'

            SolutionType: Managed

        - task: PowerPlatformImportSolution@2

          inputs:

            authenticationType: PowerPlatformSPN

            PowerPlatformSPN: 'powerplatform-u-connection'

            Environment: 'https://domainname-uat.crm11.dynamics.com'

            SolutionInputFile: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/$(SolutionName)_Managed.zip'

            UseDeploymentSettingsFile: true

            DeploymentSettingsFile: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/settings-uat.json'

            ConvertToManaged: true

            PublishCustomizationChanges: true

            MaxAsyncWaitTime: "60"

        - task: PowerPlatformPublishCustomizations@2

          inputs:

            authenticationType: 'PowerPlatformSPN'

            PowerPlatformSPN: 'powerplatform-u-connection'

- stage: deployPROD

  displayName: Deploy to PROD

  jobs:

   - deployment: deploy_to_Prod 

     environment: Intranet

     strategy:

      runOnce:

       deploy:

        steps:

        - checkout: self

        - task: PowerPlatformToolInstaller@2

          inputs:

            DefaultVersion: true

        - task: PowerPlatformPackSolution@2

          inputs:

            SolutionSourceFolder: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/$(SolutionName)_Managed'

            SolutionOutputFile: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/$(SolutionName)_Managed.zip'

            SolutionType: Managed

        - task: PowerPlatformImportSolution@2

          inputs:

            authenticationType: PowerPlatformSPN

            PowerPlatformSPN: 'powerplatform-p-connection'

            Environment: 'https://org7b3c6721.crm11.dynamics.com'

            SolutionInputFile: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/$(SolutionName)_Managed.zip'

            UseDeploymentSettingsFile: true

            DeploymentSettingsFile: '$(System.DefaultWorkingDirectory)/PowerPlatformSolution/settings-prod.json'

            ConvertToManaged: true

            PublishCustomizationChanges: true

            MaxAsyncWaitTime: "60"

        - task: PowerPlatformPublishCustomizations@2

          inputs:

            authenticationType: 'PowerPlatformSPN'

            PowerPlatformSPN: 'powerplatform-p-connection'

 

Define environment as appropriate

 



Approvals on each environment

 

 



 

 