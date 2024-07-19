

Update A row action was Dataverse was not available to the flow when it was available in the same environment for same user in a different flow just not the flow we were trying to add the action.


Error using Update A row in selected environment

Principal user (Id=0e9e3c45-96c2-ed11-83fe-6045bdcfea8c, type=8, roleCount=6, privilegeCount=1491, accessMode='0 Read-Write', AADObjectId='78d6253a-35f4-404d-aca2-23b610c55204', MetadataCachePrivilegesCount=5214, businessUnitId=69535865-29b0-eb11-8236-000d3a874bd8), is missing prvWritemsdyn_flow_approval privilege (Id=2fd4f0b0-ff84-4330-93fc-6bb19834ef71) on OTC=10286 for entity 'msdyn_flow_approval' (LocalizedName='Approval'). context.Caller=0e9e3c45-96c2-ed11-83fe-6045bdcfea8c. Consider adding missed privilege to one of the principal (user/team) roles.

Solution was to recreate the flow from scratch