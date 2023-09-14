---
title: "UseOfChatGPTInPowerShell_TestData"
date: 2023-09-14T07:03:40+01:00
draft: true
---

```PowerShell

function Ensure-SharePointList {
    param(
        [Parameter(Mandatory = $true)]
        [string]$siteUrl
)
connect-pnponline -Url $siteUrl -interactive

If(!(Get-PnPList -Identity "Wellbeing" -ErrorAction SilentlyContinue)){

  $list = New-PnPList -Title "Wellbeing" -Template GenericList
# Add columns to the list
  Add-PnPField -List $list -DisplayName "Description" -InternalName "Description" -Type Note | out-null
  Add-PnPField -List $list -DisplayName "Category" -InternalName "Category" -Type MultiChoice -Choices "Social", "Exercise", "Diet","Leisure", "Mindfulness","Creativity", "Personal Development" | out-null
  Add-PnPField -List $list -DisplayName "Level" -InternalName "Level" -Type MultiChoice -Choices "All", "Beginner", "Expert" | out-null
  Add-PnPField -List $list -DisplayName "Duration" -InternalName "Duration" -Type Text | out-null
  #-Choices "<1hr", "n/A", "<1day", "<1week"
 }
}
function Get-ListData {
    param(
        [Parameter(Mandatory = $true)]
        [string]$numofData,
        [Parameter(Mandatory = $true)]
        [string]$openai_api_key
    )

    $openai_api_endpoint = "https://api.openai.com/v1/chat/completions";

    $data = @{}
    $data["model"] = "gpt-3.5-turbo";
    $data["messages"] = @(@{});
    $data["messages"][0]["role"] = "user";
    $messageContent = "create list of ${numofData} unique wellbeing activities in the CSV format with properties 'Title', 'Description','Category', 'Level' 'Duration' with delimiter '|'. Category can be multi valued, e.g.'Social, Exercise, Diet,Leisure, Mindfulness,Creativity, Personal Development'. ";
    $data["messages"][0]["content"] = $messageContent;
    
    $headers = @{
        "Content-Type"  = "application/json"
        "Authorization" = "Bearer " + $openai_api_key
    }

    Write-Host "Calling OpenAI API to get list of wllbeing activities.";

    $response = Invoke-WebRequest -Method Post -Uri $openai_api_endpoint -Headers $headers -Body ($data | ConvertTo-Json);

    if ($response -and $response.StatusCode -eq 200) {
        $responseObject = $response.Content | ConvertFrom-Json
        $data =$responseObject.choices[0].message.content | convertFrom-CSV -Delimiter '|'
        } else {
        $data = $null;
    }
    return $data;
}
# Configure variables
$siteUrl = "https://contoso.sharepoint.com/sites/Wellbeing" # change to your domain

$numofData = 100 # change to integer value
#Go to https://platform.openai.com/account/api-keys to generate key

 $openai_api_key = "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";

$data = Get-ListData -numofData $numofData -openai_api_key $openai_api_key

Ensure-SharePointList -siteUrl $siteUrl

# list of data to create in the format of "Title|Description|Category|Level|Duration"
connect-pnponline -Url $siteUrl -interactive

# Loop through the list of data and insert into SharePoint list
foreach ($record in $data)
{ 
 $itemValue = @{}
 $itemValue.add("Title",$record.Title)
 $itemValue.add("Description",$record.Description)
 $itemValue.add("Category",$record.Category)
 $itemValue.add("Level",$record.Level)
 $itemValue.add("Duration",$record.Duration)

  Add-PnPListItem -List "Wellbeing" -Values $itemValue | out-null
}



# function to get user names from a list of users in format "FirstName LastName"

``````

This code appears to be a PowerShell script that interacts with Microsoft SharePoint and the OpenAI API to create a list of wellbeing activities and store them in a SharePoint list.

Here's a breakdown of the main sections:

Ensure-SharePointList Function:

This function checks if a SharePoint list named "Wellbeing" exists. If not, it creates the list and adds specific columns (Description, Category, Level, Duration).
Get-ListData Function:

This function is responsible for generating a list of wellbeing activities using the OpenAI API. It sends a request to the API with a predefined message to generate the data. It then converts the API response into a CSV format.
Configuration Variables:

siteUrl: Specifies the URL of the SharePoint site where the list will be created.
numofData: Indicates the number of unique wellbeing activities to be generated (set to 100).
openai_api_key: Contains the OpenAI API key. You should replace "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" with your actual API key.
Retrieve Data from OpenAI and Ensure SharePoint List:

Calls Get-ListData to get the list of wellbeing activities from OpenAI.
Calls Ensure-SharePointList to create the SharePoint list if it doesn't already exist.
Insert Data into SharePoint List:

Loops through the generated data and inserts each record into the "Wellbeing" list on SharePoint.
Overall, this script automates the process of generating wellbeing activities using the OpenAI API and storing them in a SharePoint list for further use. It's designed to be run in a PowerShell environment with the necessary permissions to interact with both OpenAI and SharePoint.