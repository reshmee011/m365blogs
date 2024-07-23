---
title: "Update A row action from Dataverse connector missing in Power Automate flow"
date: 2024-07-19T09:53:05+01:00
tags: ["Power Automate", "Dataverse connector", "Update A Row Action"]
featured_image: '/posts/images/powerautomate-updatearow-action-missing/Cancel_Action_Succeeded.png'
omit_header_text: true
draft: false
---

`Update A row action` from Dataverse connector was not available to an older Power Automate.The requirement was to add ability to cancel an approval task created via action 'Create Approval' within 30 days as Power Automate flows timeout after 30 days. I noticed the action `Update a Row in Selected Environment` and decided to try it instead.

![Update a Row Selected Environment](../images/powerautomate-updatearow-action-missing/UpdateARow_SelectedEnvironment.png)

Unfortunately it threw the forbidden error for no apparent reason with no obvious permissions to grant.

![Update a Row Selected Environment forbidden](../images/powerautomate-updatearow-action-missing/UpdateARow_SelectedEnvironment_forbidden.png)

The error message was 

> Principal user (Id=0e9e3c45-96c2-ed11-83fe-6045bdcfea8c, type=8, roleCount=6, privilegeCount=1491, accessMode='0 Read-Write', AADObjectId='78d6253a-35f4-404d-aca2-23b610c55204', MetadataCachePrivilegesCount=5214, businessUnitId=69535865-29b0-eb11-8236-000d3a874bd8), is missing prvWritemsdyn_flow_approval privilege (Id=2fd4f0b0-ff84-4330-93fc-6bb19834ef71) on OTC=10286 for entity 'msdyn_flow_approval' (LocalizedName='Approval'). context.Caller=0e9e3c45-96c2-ed11-83fe-6045bdcfea8c. Consider adding missed privilege to one of the principal (user/team) roles.

I tried the action `Update a Row (Legacy)` failing to get it to work. 

![Update a Row Legacy](../images/powerautomate-updatearow-action-missing/UpdateARow_Legacy.png)

The action `Update A row action` was available in the same environment for same user in a different new flow just not the flow we were trying to add the action.

![Update a row Action](../images/powerautomate-updatearow-action-missing/UpdateARow_Action.png.png)



## Solution: Recreate the flow from scratch

After recreating the Power Automate flow from scratch copying across all actions and adding the `Update a Row` action renamed to `Cancel Action`

![Cancel Action](../images/powerautomate-updatearow-action-missing/Cancel_Action.png)

Running the flow succeeded this time. For testing purposes the timeout of the delay action was set to minutes instead of 29 days.

![Cancel Action Succeeded](../images/powerautomate-updatearow-action-missing/Cancel_Action_Succeeded.png)


