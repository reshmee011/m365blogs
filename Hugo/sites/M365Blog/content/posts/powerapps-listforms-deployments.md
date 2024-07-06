---
title: "Powerapps Listforms Deployments"
date: 2024-03-25T12:02:05Z
omit_header_text: true
draft: true
---
## steps
1. export the list form app
2. save it to packages/dev folder relative to the script 
3. run the first script ExtractApps to extract the files 
Generate package for targeted environment
4. update the json properties: siteurl, list1guid and name
5. Run the script under "Generate targeted environment packages"
6. Import package into power apps selected environment
7. open app form 
8. Publish the form

## Unpacks solution to save into source repository

   
```
# Create the Packages/Dev folder locally before running the script

function ExtractApps($apps) {
    $apps | ForEach-Object {
        Write-Host "Extracting $($_.Name)" -ForegroundColor Blue
        $folderName = ($_.Name -split '_')[0]
        $outputPath = "src\$folderName"

        # Delete current folder and extract zip file
        If (Test-Path -Path $outputPath) { Remove-Item -Path $outputPath -Recurse -Force }
        New-Item -ItemType Directory -Path $outputPath
        Expand-Archive -Path "packages\dev\$($_.Name)" -DestinationPath $outputPath -Force
       
        # Unpack msapp files
        $msapp = Get-ChildItem -Path $outputPath -Include '*.msapp' -Recurse
        $msappPath = Convert-Path $msapp.PSPath
        $msappExtractPath = New-Item -Path ($msappPath + '_') -ItemType Directory
        pac canvas unpack --msapp $msapp --sources $msappExtractPath

        Remove-Item -Path $msappPath -Force
        Rename-Item -Path ($msappPath + '_') -NewName $msappPath
    }
}

cd $PSScriptRoot

$ErrorActionPreference = "Stop"
$files = Get-ChildItem -Path "packages/dev/*" -Include *.zip
ExtractApps $files
```

## Generate targeted environment packages
```
Param(
    [Parameter(Mandatory=$true)]
    [ValidateSet("ST", "UAT", "PROD")]
    [string] $env
)

$tokens = @{
    dev = @{
        siteUrl = 'https://contoso.sharepoint.com/teams/d-app-ar'
        list1Guid = '6009a7be-5602-4340-ba46-2ed38307afbb' 
        name = 'Dev Form'
    }
ST = @{
        siteUrl = 'https://contoso.sharepoint.com/teams/app-ar'
        list1Guid = '668886a6-e9fc-4f1d-845e-16368114d94f'
        name = 'ST Form'
    }
UAT = @{
        siteUrl = 'https://contoso.sharepoint.com/teams/app-ar'
        list1Guid = '668886a6-e9fc-4f1d-845e-16368114d94f'
        name = 'UAT Form'
    }
     prod = @{
        siteUrl = 'https://contoso.sharepoint.com/teams/app-ar'
        list1Guid = '668886a6-e9fc-4f1d-845e-16368114d94f'
        name = 'Prod Form'
    }
}

function ReplaceValuesInFile($file, $environment) {    
    Write-Host "`tReplacing tokens in $file" -ForegroundColor Blue
    $json = Get-Content -Path $file
    
    $tokens.dev.Keys | ForEach-Object {
        $json = $json.Replace($tokens.dev.($_), $tokens.($environment).($_))
    }

    Set-Content -Path $file -Value $json
}

function CreateApps($apps) {
    $apps | ForEach-Object {
        Write-Host "Creating $($_.Name) for $env" -ForegroundColor Blue
        
        $srcPath = "src\$($_.Name)"
        $envPath = "packages/$env"
        $outputPath = "$envPath/$($_.Name)"
        If (!(Test-Path -Path $envPath)) { New-Item -ItemType Directory -Path $envPath }
        If (Test-Path -Path $outputPath) { Remove-Item -Path $outputPath -Recurse -Force }
        Copy-Item -Path $srcPath -Destination $envPath -Recurse

        # Replace tokens
        $jsonFiles = Get-ChildItem -Path $outputPath -Include '*.json' -Recurse
        $jsonFiles | ForEach-Object {
            ReplaceValuesInFile $_ $env
        }

        $yamlFiles = Get-ChildItem -Path $outputPath -Include '*.yaml' -Recurse
        $yamlFiles | ForEach-Object {
            ReplaceValuesInFile $_ $env
        }
        
        # Create msapp archive
        $msapp = Get-ChildItem -Path $outputPath -Include '*.msapp' -Recurse
        $msappPath = Convert-Path $msapp.PSPath
        Rename-Item -Path $msappPath -NewName ($msappPath + '_')
        pac canvas pack --msapp $msappPath --sources ($msappPath + '_')
        Remove-Item -Path ($msappPath + '_') -Recurse -Force
        
        # Create main archive
        Compress-Archive -Path ($outputPath + '\*') -DestinationPath "packages\$env\$($_.Name)" -Force
        Remove-Item -Path $outputPath -Recurse -Force
    }
}

cd $PSScriptRoot

$ErrorActionPreference = "Stop"
$apps = Get-ChildItem -Path "src/*"
CreateApps $apps

```

Even if the powerapps form is deleted from the form settings, the form might have to be deleted from the respective environments.

```
Remove-AdminPowerApp -AppName d9a43f7a-c26c-4aec-b34b-5fe3f235966b -EnvironmentName  default-6d0f5fa7-7e4f-417d-a8fe-07682da182e8
```
