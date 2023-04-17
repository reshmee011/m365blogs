---
title: 'Create Virtual Machine(VM) in Azure throws "Long running operation failed with status ''Failed''." through PowerShell'
date: Fri, 18 Nov 2016 17:25:22 +0000
draft: false
tags: ['Azure', 'Create Virtual Machine in Azure', 'Long running operation failed with status 'Failed', 'New-AzureRMVM', 'PowerShell', 'SharePoint 2016', 'Virtual Machine', 'VM 'adVM' has not reported status for VM agent or extensions. Please verify the VM has a running VM agent, and can establish outbound connections to Azure storage']
---

I was following the article [SharePoint Server 2016 dev/test environment in Azure](https://technet.microsoft.com/library/mt723354.aspx) to create a SharePoint Server 2016 environment in Azure. I reached the step to create the Virtual Machine that will be used as the domain controller using the cmdlet below```
New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm


```After waiting for several minutes I decided to cancel the operation. When I retried the same cmdlet, I got the following error.```
New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm -Verbose
VERBOSE: Performing the operation "New" on target "adVM".
New-AzureRMVM : Long running operation failed with status 'Failed'.
ErrorCode: VMAgentStatusCommunicationError
ErrorMessage: VM 'adVM' has not reported status for VM agent or extensions. 
Please verify the VM has a running VM agent, and can establish 
outbound connections to Azure storage.
StartTime: 18/11/2016 15:42:46
EndTime: 18/11/2016 15:42:46
OperationID: f1ecb302-9da9-4a76-9b0c-463b5e89c41c
Status: Failed
At line:1 char:1
\+ New-AzureRMVM -ResourceGroupName $rgName -Location $locName -VM $vm -Verbose
\+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 + CategoryInfo : CloseError: (:) \[New-AzureRmVM\], ComputeCloudException
 + FullyQualifiedErrorId : Microsoft.Azure.Commands.Compute.NewAzureVMCommand 

```I spent a while trying to understand what the error message meant. In that scenerio the error message was thrown because the VM requested was created successfully even though I interrupted the operation before. The error thrown was to prevent creating another VM with the same details. When I created the second VM, I waited long enough till the success message was displayed. ![vmcreatedsuccessfully](https://reshmeeauckloo.files.wordpress.com/2016/11/vmcreatedsuccessfully.png)