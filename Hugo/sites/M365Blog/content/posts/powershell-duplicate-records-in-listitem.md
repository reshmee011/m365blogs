   ---
title: "Update Choice values of List Items in SharePoint List"
date: 2024-07-13T06:39:48Z
tags: ["PowerShell","Choice","SharePoint","List"]
featured_image: '/posts/images/powershell-update-choice-value-from-old-to-new/csv.png'
omit_header_text: true
draft: true
---

#Parameters
$SiteURL = "https://contoso.sharepoint.com/teams/app-ar"
$ListName = "Remittances"

$dateTime = (Get-Date).toString("dd-MM-yyyy")
$invocation = (Get-Variable MyInvocation).Value
$directorypath = Split-Path $invocation.MyCommand.Path
$fileName = "\RemittanceDuplicateReport-" + $dateTime + ".csv"
$OutPutView = $directorypath + $fileName

Connect-PnPOnline -url $SiteURL -Interactive


$ListItems = Get-PnPListItem -List $ListName  -PageSize 500 | Where {$_.FieldValues.Month -eq '202402' }
#Array for Results Data
$DataCollection = @()
ForEach($Item in $ListItems)
{
    #Collect data       
    $Data = New-Object PSObject -Property @{
            SourceRef  = $Item["DocumentSourceReference"]
            GroupingRef  = $Item["GroupingReference"]
            Provider  = $Item["Provider"].LookupValue
            PaymentDate  = $Item["PaymentDate"]
            FilterKey  = $Item["FilterKey"]
            Fund  = $Item["MemberType"]
            Month = $Item["Month"]
            Policy = $Item["PolicyNumber"]
            Amount = $Item["Amount"]
            CreatedDate = $Item["Created"]
            RemittanceMatchedtoReceipt = $Item["IsPaymentMatched"]
            MatchReceiptId = $Item["PaymentRecord"]
            ID = $Item.Id
        }
    $DataCollection += $Data
}
#Get Duplicate Rows based on Column: SourceRef, Month. Policy and Amount 
# remove group-object SourceRef to identify duplicates independent of files 
$duplicates = $DataCollection | Sort-Object -Property SourceRef  | Group-Object -Property Month, Policy,Amount, FilterKey,PaymentDate | Where-Object {$_.Count -gt 1} | Select-Object -ExpandProperty Group
$duplicates | Export-CSV $OutPutView -Force -NoTypeInformation
#Leave the 1st Item as Original and Select the Second Item in the Group as Duplicate
ForEach($Duplicate in $Duplicates) 
{
    $sourceRef = $Duplicate.SourceRef
    $groupingReference = $Duplicate.GroupingRef

    #TODO Delete the Duplicate Row
    $Duplicate.Group | Select-Object -Skip 1 | ForEach-Object {
    # TO DO logic to delete duplicate
    } 
}

