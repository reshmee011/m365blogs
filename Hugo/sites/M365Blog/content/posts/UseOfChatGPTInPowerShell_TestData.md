---
title: "Leveraging ChatGPT to Generate Test Data for SharePoint Lists Using PnP PowerShell"
date: 2023-09-14T07:03:40+01:00
tags: ["PnPBatch", "list","SharePoint", "SystemUpdate"]
featured_image: '../images/UseOfChatGPTInPowerShell_TestData/TestDataSuccessfullyGenerated.png'
draft: false
---

# Leveraging ChatGPT to Generate Test Data for SharePoint Lists Using PnP PowerShell

Utilizing ChatGPT to generate test data for various applications is a powerful capability. After stumbling upon the sample script
[Create test users using MS Graph API from list or ask ChatGPT to generate test users](https://pnp.github.io/script-samples/graph-create-test-users-with-chat-gpt/README.html?tabs=graphps) by [Valeras Narbutas](https://github.com/ValerasNarbutas) on creating test users with MS Graph API, I decided to explore using ChatGPT to generate a list of data. This script harnesses OpenAI's ChatGPT model to effortlessly generate a list of wellneing activities. My aim was to save this data into a SharePoint list, intending to use it as a datasource in a forthcoming application.

Before integrating ChatGPT into PowerShell, you'll need an OpenAI API key. You can obtain one by visiting [OpenAI's API key page](https://platform.openai.com/account/api-keys) and creating a new secret key. Keep in mind that if you're using a free account, you might encounter a quota error ("You exceeded your current quota, please check your plan and billing details").

![Exceeded current quota](../images/UseOfChatGPTInPowerShell_TestData/ExceededYourCurrentQuota.png)

Bear in mind that the [OpenAI's pricing](https://openai.com/pricing) is per token. Generating extensive test data can potentially be costly. I recommend using the browser to experiment with different prompts to determine which one yields the best data. I initially attempted to generate the data in JSON format. However, I found that the maximum records generated ranged from 0 to 10 with varying responses.

When I used the prompt: "create list of unique 200 wellbeing activities in the json format with properties 'Title', 'Description','Category', 'Level' 'Duration'. Category can be multi valued, e.g.'Social, Exercise, Diet,Leisure, Mindfulness,Creativity, Personal Development'", I didn't get very far.

![Too much data requested](../images/UseOfChatGPTInPowerShell_TestData/TooMuchForOpenAI.png)

However, by making a slight modification to the prompt and requesting the data in CSV format, I obtained the desired result:

"Create list of unique wellbeing activities in the CSV format with properties 'Title', 'Description','Category', 'Level' 'Duration' with delimiter '|'. Category can be multi valued, e.g.'Social, Exercise, Diet,Leisure, Mindfulness,Creativity, Personal Development'.Number or records to create: 100"

![Data in CSV format](../images/UseOfChatGPTInPowerShell_TestData/RequestDatainSCV.png)

With a functional prompt in hand, I proceeded to create the script below using PnP PowerShell and a REST API call to ChatGPT in order to generate a list of wellbeing activities and store them in a SharePoint list.

## Prerequisites
[PnP PowerShell](https://pnp.github.io/powershell/)
[Open AI Key](https://platform.openai.com/account/api-keys)

```PowerShell
function Ensure-SharePointList {
    param(
        [Parameter(Mandatory = $true)]
        [string]$siteUrl
)
connect-pnponline -Url $siteUrl -interactive

If(!(Get-PnPList -Identity "Wellbeing" -ErrorAction SilentlyContinue)){

  $list = New-PnPList -Title "Wellbeing" -Template GenericList | out-null
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
```
## Summary of the script

This script automates the process of generating wellbeing activities using the OpenAI API and storing them in a SharePoint list for further use. It's designed to be run in a PowerShell environment with the necessary permissions to interact with both OpenAI and SharePoint.

### Configuration Variables

Update the values of the following variables before use

**siteUrl**: Specifies the URL of the SharePoint site where the list will be created.
**numofData**: Indicates the number of unique wellbeing activities to be generated (set to 100).
**openai_api_key**: Contains the OpenAI API key. You should replace "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" with your actual API key.


[Wellbeing activities saved into SharePoint List](../images/UseOfChatGPTInPowerShell_TestData/TestDataSuccessfullyGenerated.png)