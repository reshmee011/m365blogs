---
title: "Using Template for ALM for Power Platform Deployment"
date: 2024-06-15T16:54:24+01:00
tags: ["Power Platform","solution","managed solution","Template","Azure DevOps","ALM"]
omit_header_text: true
featured_image: '/posts/images/AzureDevOps-PowerPlatform-deployment-ALM-SettingsFile/CreateDeploymentSettings.png'
draft: true
---

Template to build
```yaml
parameters:
  - name: 'solution_folder'
    type: string
  - name: 'solution_name'
    type: string
  - name: 'branch_name'
    type: string

jobs:
- job: Export
  pool:
    name: M365

  steps:
  - task: DeleteFiles@1
    displayName: 'Delete files from $(Build.SourcesDirectory)/${{ parameters.solution_folder }}'
    inputs:
      SourceFolder: '$(Build.SourcesDirectory)/${{ parameters.solution_folder }}'
      Contents: '**/*'

  - task: microsoft-IsvExpTools.PowerPlatform-BuildTools.tool-installer.PowerPlatformToolInstaller@2
    displayName: 'Power Platform Tool Installer'

  - task: microsoft-IsvExpTools.PowerPlatform-BuildTools.export-solution.PowerPlatformExportSolution@2
    displayName: 'Power Platform Export Unmanaged Solution'
    inputs:
      authenticationType: PowerPlatformSPN
      PowerPlatformSPN: 'powerplatform-d-connection'
      Environment: 'https://contoso-dev.crm11.dynamics.com'
      SolutionName: ${{ parameters.solution_name }}
      SolutionOutputFile: '$(Build.ArtifactStagingDirectory)/${{ parameters.solution_folder }}/${{ parameters.solution_name }}.zip'

  - task: microsoft-IsvExpTools.PowerPlatform-BuildTools.unpack-solution.PowerPlatformUnpackSolution@2
    displayName: 'Power Platform Unpack Unmanaged Solution '
    inputs:
      SolutionInputFile: '$(Build.ArtifactStagingDirectory)/${{ parameters.solution_folder }}/${{ parameters.solution_name }}.zip'
      SolutionTargetFolder: '$(Build.SourcesDirectory)/${{ parameters.solution_folder }}'

  - script: |
      echo Commit Solution into GIT Branch ${{ parameters.branch_name }}
      git config user.email pipeline.automation@contoso.co.uk
      git config user.name "Pipeline Automation"
      
      git switch --force-create ${{ parameters.branch_name }}
      
      git -c http.extraheader="AUTHORIZATION: bearer $(System.AccessToken)" pull origin ${{ parameters.branch_name }} --no-rebase
      
      echo Adding all files to ${{ parameters.branch_name }}
      git add --all
      git commit -m "Checked in by Power Platform Release"
      
      echo Pushing solution components to ${{ parameters.branch_name }}
      git -c http.extraheader="AUTHORIZATION: bearer $(System.AccessToken)" push origin HEAD:${{ parameters.branch_name }} --force
    displayName: 'Check In Solution'
```
Template to deploy

```yaml
parameters:
- name: job_name
  type: string
- name: agent_pool
  type: string
- name: service_connection
  type: string
- name: environment_url
  type: string
- name: solution_folder
  type: string
- name: settings_file
  type: string  
  
jobs:
  - deployment: ${{ parameters.job_name }}
    pool: 
      name: ${{ parameters.agent_pool }}
    steps:
    - task: PowerPlatformToolInstaller@2
      displayName: 'Power Platform Tool Installer'
      inputs:
        PowerAppsAdminVersion: 2.0.150
        XrmToolingPackageDeploymentVersion: 2.0.150
        MicrosoftPowerAppsCheckerVersion: 2.0.150
        CrmSdkCoreToolsVersion: 2.0.150
        XrmOnlineManagementApiVersion: 2.0.150
    - task: PowerPlatformPackSolution@2
      displayName: 'Power Platform Pack Solution '
      inputs:
        SolutionSourceFolder: ${{ parameters.solution_folder }}
        SolutionOutputFile: ${{ parameters.solution_folder }}/managed.zip
        SolutionType: Managed
    - task: PowerPlatformImportSolution@2
      displayName: 'Power Platform Import Solution'
      inputs:
        authenticationType: PowerPlatformSPN
        PowerPlatformSPN: ${{ parameters.service_connection }}
        Environment: ${{ parameters.environment_url }}
        SolutionInputFile: ${{ parameters.solution_folder }}/managed.zip'
        UseDeploymentSettingsFile: true
        DeploymentSettingsFile: ${{ parameters.solution_folder }}/${{ parameters.settings_file }}
        ConvertToManaged: true
        PublishCustomizationChanges: true```
```

Using the template to export

```yaml
stages:
- stage: export
  displayName: Export Supplier Validation Solution
  jobs:
    - template: ./power-platform-solution-export.yml
      parameters:
        solution_folder: SupplierDataValidation
        solution_name: PPFFinanceSupplierValidation
        branch_name: supplier
```

Using the template to deploy

```yaml
name: $(TeamProject)_$(BuildDefinitionName)_$(SourceBranchName)_$(Date:yyyyMMdd)$(Rev:.r)

trigger:
- none

resources:
  pipelines:
  - pipeline: buildpipeline
    source: FinanceSupplierDataValidation-Export

stages:
- stage: deployprod
  displayName: Deploy to Prod
  jobs:
    - template: ./power-platform-solution-deploy.yml
      parameters:
        job_name: deploy_solution_prod
        service_connection: 'powerplatform-p-connection'
        environment_url: 'https://org.crm11.dynamics.com'
        solution_folder: '$(System.DefaultWorkingDirectory)/SupplierDataValidation'
        settings_file: 'supplier-data-validation-settings-prod.json'
```