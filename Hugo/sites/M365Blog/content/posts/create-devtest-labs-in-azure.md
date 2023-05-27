---
title: 'Create DevTest Labs in Azure'
date: Wed, 11 Jan 2017 08:28:58 +0000
draft: false
tags: ['Azure', 'Create DevTest Lab', 'DevTest Lab', 'PowerShell', 'Provisioning DevTest Lab using PowerShell']
---

[Azure DevTest Labs is available in UK South and UK West](https://azure.microsoft.com/en-us/updates/azure-devtest-labs-now-available-in-uk-south-and-uk-west/) as from December 2016, in addition to the other 21 regions it has supported. The steps to create the DevTest lab are

*   Login to Azure portal as administrator
*   Click the green + New menu

![createdevtestlab](https://reshmeeauckloo.files.wordpress.com/2017/01/createdevtestlab1.png)

*   Type DevTest Labs into the search box
*   Select DevTestLabs from the results page
*   Click on Create from the Description page.

The advantages using DevTest Labs as mentioned from the Description page are _DevTest Labs helps developers and testers to quickly create virtual machines in Azure to deploy and test their applications. You can easily provision Windows and Linux machines using reusable templates while minimizing waste and controlling cost._

*   _Quickly provision development and test virtual machines_
*   _Minimize waste with quotas and policies_
*   _Set automated shutdowns to minimize costs_
*   _Create a VM in a few clicks with reusable templates_
*   _Get going quickly using VMs from pre-created pools_
*   _Build Windows and Linux virtual machines_

 

*   Enter the lab name, select the subscription, select location North Europe, tick the Pin to Dashboard tick box and alternatively update the Auto-shutdown schedule.

![createdevtestlab_details](https://reshmeeauckloo.files.wordpress.com/2017/01/createdevtestlab_details.png)

*   Click on Create.
*   The dashboard is displayed with a new tile showing that the DevTest Lab is being deployed.![deployingdevtest-labs_inprogress](https://reshmeeauckloo.files.wordpress.com/2017/01/deployingdevtest-labs_inprogress.png)
*   The DevTest Lab page is displayed once deployment of the DevTest Lab is completed.

![devtest-labs_completed](https://reshmeeauckloo.files.wordpress.com/2017/01/devtest-labs_completed.png)   Instead of using the Portal, PowerShell can be used to create Azure DevTest Lab. The GitHub repository https://github.com/Azure/azure-devtestlab/tree/master/Samples/ProvisionDemoLab provides an example how it can be achieved. The repository has a readme file, a deployment template with a corresponding parameters file and a PowerShell script to execute the deployment. The Readme file provides a description of the resources created. _About the resources created in the Demo Lab:_ _The ARM template creates a demo lab with the following things:_ _\* It sets up all the policies and a private artifact repo._ _\* It creates 3 custom VM images/templates._ _\* It creates 4 VMs, and 3 of them are created with the new custom VM images/templates._ To run the PowerShell script the subscriptionId is required. This can be obtained from the cmdlet Login-AzureRmAccount. ![login-azurermaccount](https://reshmeeauckloo.files.wordpress.com/2017/01/login-azurermaccount.png) The PowerShell is run as below```
.\\ProvisionDemoLab.ps1 -SubscriptionId 41111111-1111-1111-1111-111111111111 
-ResourceGroupLocation northeurope -ResourceGroupName RTestLab
```![provisiondemolab](https://reshmeeauckloo.files.wordpress.com/2017/01/provisiondemolab.png) The script produces the following results. ![ProvisionDemoLab_SuccededMessage.PNG](https://reshmeeauckloo.files.wordpress.com/2017/01/provisiondemolab_succededmessage.png) From the portal , the result shows the 4 vms.![provisiondemolab_portal](https://reshmeeauckloo.files.wordpress.com/2017/01/provisiondemolab_portal.png) The repositories have been created as well.![provisiondemolab_repositories](https://reshmeeauckloo.files.wordpress.com/2017/01/provisiondemolab_repositories.png) Custom images of the running machines have been created as well. ![ProvisionDemoLab_CustomImages.PNG](https://reshmeeauckloo.files.wordpress.com/2017/01/provisiondemolab_customimages.png) There are artifacts ready to be used though none are applied yet to the virtual machines. ![ProvisionDemoLab_Artifacts.PNG](https://reshmeeauckloo.files.wordpress.com/2017/01/provisiondemolab_artifacts.png) You can create your own templates/parameters files in the Portal by creating a new resource and exporting  instead of executing the configuration in the GitHub repository.