---
title: 'Create Dev/Test SharePoint 2013 environment in Azure'
date: Tue, 06 Dec 2016 11:49:22 +0000
draft: false
tags: ['Azure', 'Create Dev/Test SharePoint 2013 environment in Azure', 'Create SharePoint 2013 VM', 'Create SQL Server 2012 VM', 'Get-AzureRmVMImageOffer', 'Get-AzureRmVMImageSKU', 'PowerShell', 'SharePoint 2013', 'SQL Server', 'Virtual Machine', 'Virtual Network']
---

Azure has a trial image to build either [SharePoint 2013 HA farm](https://azure.microsoft.com/en-gb/marketplace/partners/sharepoint2013/sharepoint2013farmsharepoint2013-ha/) or [SharePoint 2013 Non-HA farm](https://azure.microsoft.com/en-gb/marketplace/partners/sharepoint2013/sharepoint2013farmsharepoint2013-nonha/).

When trying to create SharePoint 2013 Non-HA farm, I was stuck at step "Choose storage account type" with the message "Loading pricing...".

![hangingselectstorageaccounttypefromazure](https://reshmeeauckloo.files.wordpress.com/2016/11/hangingselectstorageaccounttypefromazure.png)

Following [SharePoint Server 2016 dev/test environment in Azure](https://technet.microsoft.com/library/mt723354.aspx), I managed to created a SharePoint 2013 environment in Azure running PowerShell commands.

There are three major phases to setting up this dev/test environment:

1.  Set up the virtual network and domain controller (ad2013VM).I followed all steps described in [Phase 1: Deploy the virtual network and a domain controller](https://technet.microsoft.com/library/mt723354.aspx#Phase 1: Deploy the virtual network and a domain controller) to set up the virtual network and domain controller
2.  Configure the SQL Server computer (sql2012VM).I followed all steps from [Phase 2: Add and configure a SQL Server 2014 virtual machine](https://technet.microsoft.com/library/mt723354.aspx#Phase 2: Add and configure a SQL Server 2014 virtual machine) to create the SQL server computer with few changes to the PowerShell script to create a SQL2012R2 machine.
3.  Configure the SharePoint server (sp2013VM).                                                                               I followed all steps from [Phase 3: Add and configure a SharePoint Server 2016 virtual machine](https://technet.microsoft.com/library/mt723354.aspx#Phase 3: Add and configure a SharePoint Server 2016 virtual machine) with few changes to the script to create a SharePoint 2013 virtual machine.

Configure the SQL Server computer (sql2012VM).
----------------------------------------------

I needed to get the name of SQL 2012 SP2 Azure image offer. I can list all SQL Azure image offers using the cmdlet Get-AzureRMImageOffer.

```
Get-AzureRmVMImageOffer -Location "westeurope" 
-PublisherName "MicrosoftSQlServer"
```

![get-azurermimageoffersql](https://reshmeeauckloo.files.wordpress.com/2016/11/get-azurermimageoffersql.png)

The following SQL Image Offers are available

```
Offer
-----
SQL2008R2SP3-WS2008R2SP1
SQL2008R2SP3-WS2012
SQL2012SP2-WS2012
SQL2012SP2-WS2012R2
SQL2012SP3-WS2012R2
SQL2012SP3-WS2012R2-BYOL
SQL2014-WS2012R2
SQL2014SP1-WS2012R2
SQL2014SP1-WS2012R2-BYOL
SQL2014SP2-WS2012R2
SQL2014SP2-WS2012R2-BYOL
SQL2016-WS2012R2
SQL2016-WS2012R2-BYOL
SQL2016-WS2016
SQL2016-WS2016-BYOL
SQL2016CTP3-WS2012R2
SQL2016CTP3.1-WS2012R2
SQL2016CTP3.2-WS2012R2
SQL2016RC3-WS2012R2v2
SQL2016SP1-WS2016
SQL2016SP1-WS2016-BYOL
SQLvNextRHEL
```

I was interested in SQL 2012 SP2 Standard version. Fortunately the Azure Image Offer Names are intuitive, e.g. Name SQL2012SP2-WS2012R2 means windows server 2012 R2 virtual machine with SQL Server 2012 SP2 installed.

I also needed the SKU value of the SQL 2012 SP2 using the cmdlet Get-AzureRmVMImageSKU

```
 Get-AzureRmVMImageSKU -Location "westeurope" -PublisherName "MicrosoftSQlServer" 
-Offer SQL2012SP2-WS2012R2|format-table Skus
```

The following SKUs for SQL2012SP2-WS2012R2 are available

```
Skus
----
Enterprise
Enterprise-Optimized-for-DW
Enterprise-Optimized-for-OLTP
Standard
Web
```https://gist.github.com/reshmee011/bdea1a3f1ffd20601d9e5aa3c709fe17

The changes from the original script are on the following lines

*   line 21: "sql2012VM" stored in variable $vmName
*   line 23: $vnet=Get-AzureRMVirtualNetwork -Name "SP2013Vnet" -ResourceGroupName $rgName
*   line 40 : $vm=Set-AzureRMVMSourceImage -VM $vm -PublisherName MicrosoftSQLServer -Offer SQL2012SP2-WS2012R2 -Skus Standard -Version "latest"

Configure the SharePoint server (sp2013VM).
-------------------------------------------

Similarly to creating the SQL virtual machine, I needed the Azure Image Offer Name for SharePoint 2013.

The available SharePoint Azure Image offers for Microsoft SharePoint can be retrieved using the cmdlet below.

```
Get-AzureRmVMImageOffer -Location "westeurope" 
-PublisherName "MicrosoftSharePoint"
```

Only one result "MicrosoftSharePointServer" is returned.

To get the available SKUs for "MicrosoftSharePointServer", the cmdlet below can be run.

```
 Get-AzureRmVMImageSKU -Location "westeurope" -PublisherName "MicrosoftSharePointServer" 
|format-table Skus
```

![get-azurermimageoffersharepoint](https://reshmeeauckloo.files.wordpress.com/2016/11/get-azurermimageoffersharepoint.png)

Two results are returned : "2013" and "2016". I am interested in the "2013" value which refers to the Microsoft SharePoint Server 2013 version.

https://gist.github.com/reshmee011/b876fd65bf0c83d89d52cf29ce262465

The changes from the original script are on the following lines

*   line 18: $vmName="sp2013VM"
*   line 26:$vnet=Get-AzureRMVirtualNetwork -Name "SP2013Vnet" -ResourceGroupName $rgName
*   line 34: $skuName="2013"

The end result of the PowerShell scripts is a resource group with the virtual machines (adVm, sp2013Vm and sql2012VM), network interfaces, availability sets, storage account and public IP addresses to enable SharePoint 2013 to run in Azure VMs.   ![sharepoint2013resourcegroup](https://reshmeeauckloo.files.wordpress.com/2016/12/sharepoint2013resourcegroup.png)