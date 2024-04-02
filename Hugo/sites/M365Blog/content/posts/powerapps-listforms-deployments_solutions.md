---
title: "Powerapps Listforms Deployments solutions"
date: 2024-03-25T12:02:22Z
tags: ["PowerApps", "CI/CD", "Deployments","Solutions"]
featured_image: '/posts/images/powerapps-azuredevops-pipeline-failed-duetolicense-expired/Sample.png'
draft: false
---

# Power Apps list forms deployment with help of solution

Power Apps List forms are be default hidden from the list of apps from https://make.powerapps.com/. Deployment of list forms remains a challenge. 

This is my brief attempt to try to streamline the process using a solution. A solution within Power Platform makes it dataverse friendly.

Let's say you already have a list form tied to a list. In the background it is a hidden list. The list app can be added to a solution using cmdlet set-PowerAppAsSolutionAware. 

```
Install-Module Microsoft.PowerApps.Administration.PowerShell
```

```PowerShell
Set-PowerAppAsSolutionAware -EnvironmentName "26763ce5-b5a6-e556-8560-d37e8bf80aaa" -AppName "a510fe11-dbf1-4842-89ba-ce3a18d0c470" -SolutionId "ff9cb948-3359-ed11-9562-00224843838a" 
```

By default when the list form is created, it is created in the default environment. To change the list form setting default environment use the cmdlet
Change the default environment of List customization using powerapps to another environment

```PowerShell
Set-AdminPowerAppSharepointFormEnviroment -EnvironmentName <guid>
```

```dotnetcli
Get-AdminPowerApp
```

To create settings for the solution 

```powershell
pac solution create-settings --solution-zip "C:\Users\auckloor\Downloads\PPF_PowerAppForms_1_0_0_2_managed.zip" --settings-file DeploymentSettingsDev.json
```


```PowerShell
$common = @{
    dev = @{
        siteUrl  = 'https://contoso.sharepoint.com/teams/team-test'
        listGuid = '5717c826-bac4-47d6-8d78-ef5ceb00627c'
        listUrl = 'https://contoso.sharepoint.com/teams/team-test/Lists/TestPowerAppForm'
    }
    test = @{
        siteUrl  = 'https://contoso.sharepoint.com/teams/team-test1'
        listGuid = 'f3f68a11-0290-4562-b92f-4f3cbafdc0d6'
        listUrl = 'https://contoso.sharepoint.com/teams/team-test1/Lists/T_TestPowerAppForm'
    }
}

function ReplaceValuesInFile($file, $environment) {    
    $json = Get-Content -Path $file
    
    $common.dev.Keys | ForEach-Object {
        $json = $json.Replace($common.dev.($_), $common.($environment).($_))
    }

    Set-Content -Path $file -Value $json
}
#Set-PowerAppAsSolutionAware -EnvironmentName "26763ce5-b5a6-e556-8560-d37e8bf80aaa" -AppName "a510fe11-dbf1-4842-89ba-ce3a18d0c470" -SolutionId "ff9cb948-3359-ed11-9562-00224843838a" 
pac auth select --index 3
$folderPath = "E:\Scripts\PowerApps\TestPowerAppForm_"
#pac solution online-version --solution-name "TestPowerAppForm"  --solution-version "1.0.0.4"
#TODO - increment version whenever export
pac solution export --path "E:\Scripts\PowerApps\" --managed --name "TestPowerAppForm"

pac solution unpack --zipfile "$($folderPath)managed.zip" --folder "$($folderPath)managed" --packagetype managed -pca 

$canvasAppRelativeUrl = "CanvasApps"

$environment= "Test"
Get-ChildItem -Path ( "$($folderPath)managed\$($canvasAppRelativeUrl)") -Include '*.json' -Recurse | ForEach-Object {
    ReplaceValuesInFile $_ $environment
}
Get-ChildItem -Path ( "$($folderPath)managed\$($canvasAppRelativeUrl)") -Include '*.xml' -Recurse | ForEach-Object {
    ReplaceValuesInFile $_ $environment
}

pac solution pack --zipfile "$($folderPath)managed.zip" --folder "$($folderPath)managed" --packagetype managed -pca 
#connect to ST environment
pac auth select --index 2
#TODO delete any form attached to the list to allow the importing the update power app form 
pac solution import --path "$($folderPath)managed.zip"  --settings-file "E:\Scripts\DeploymentSettingsTest.json" 
#TODO - Publish powerapp
```

#publish-powerapp -AppName

#publish-adminpowerapp

## References
https://learn.microsoft.com/en-us/power-platform/admin/powerapps-powershell#designate-sharepoint-custom-form-environment