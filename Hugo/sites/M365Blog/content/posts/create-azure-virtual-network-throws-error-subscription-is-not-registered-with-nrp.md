---
title: 'Create Azure Virtual Network throws  Error "Subscription is not registered with NRP"'
date: Fri, 18 Nov 2016 15:48:19 +0000
draft: false
tags: ['Azure', 'Create Azure Virtual Network', 'PowerShell', 'Subscription is not registered with NRP', 'Virtual Network']
---

I was following the article [SharePoint Server 2016 dev/test environment in Azure](https://technet.microsoft.com/library/mt723354.aspx) to create a SharePoint Server 2016 environment in Azure. I reached the step to create the SP2016Vnet Azure Virtual Network that will host the SP2016Subnet subnet. I tried  the cmdlets below```
$spSubnet=New-AzureRMVirtualNetworkSubnetConfig -Name SP2016Subnet 
-AddressPrefix 10.0.0.0/24

 New-AzureRMVirtualNetwork -Name SP2016Vnet -ResourceGroupName $rgName 
-Location $locName -AddressPrefix 10.0.0.0/16 -Subnet $spSubnet 
-DNSServer 10.0.0.4
```Unfortunately I got the error message```
WARNING: The output object type of this cmdlet will be modified in a future release.
New-AzureRMVirtualNetwork : Subscription <Guid> is not registered with NRP.
StatusCode: 409
ReasonPhrase: Conflict
OperationID : '0937045e-3d71-411f-ba11-785e5fcff586'
At line:1 char:1
\+ New-AzureRMVirtualNetwork -Name SP2016Vnet -ResourceGroupName $rgName -Location ...
\+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 + CategoryInfo : CloseError: (:) \[New-AzureRmVirtualNetwork\], NetworkCloudException
 + FullyQualifiedErrorId : Microsoft.Azure.Commands.Network.NewAzureVirtualNetworkCommand
```![errorwithnew_azurermvirtualnetwork_notregisteredwithnrp](https://reshmeeauckloo.files.wordpress.com/2016/11/errorwithnew_azurermvirtualnetwork_notregisteredwithnrp.png) After spending a while googling through solutions , I gave up and tried to modify the script until I get it working. The fix was to enclosed the parameter "Name" value between double quotes when calling method New-AzureRMVirtualNetworkSubnetConfig.```
$spSubnet=New-AzureRMVirtualNetworkSubnetConfig 
-Name "SP2016Subnet" -AddressPrefix 10.0.0.0/24

New-AzureRMVirtualNetwork -Name SP2016Vnet 
-ResourceGroupName $rgName -Location $locName 
-AddressPrefix 10.0.0.0/16 -Subnet $spSubnet -DNSServer 10.0.0.4
```The cmdlets work after the fix. ![new_azurermvirtualnetwork_withouterrors](https://reshmeeauckloo.files.wordpress.com/2016/11/new_azurermvirtualnetwork_withouterrors.png)