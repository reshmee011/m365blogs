---
title: "Converting a HTTP Trigger into a Child Workflow in Power Automate"
date: 2023-11-20T06:19:33Z
tags: ["PowerAutomate","Child Workflow", "HTTP trigger"]
featured_image: '/posts/images/PowerAutomate_ChildWorkflow/Parent_childFlow.png'
draft: false
---

# Converting a HTTP Trigger into a Child Workflow in Power Automate

Using a HTTP trigger requires Premium licence and with the [Child Workflow](https://learn.microsoft.com/en-us/power-automate/create-child-flows) in a solution alleviates the need of premium license.

Other advantages using a solution:

- **Application Lifecycle Management**: Facilitates deployment.
- **Child Workflow**: Allows for a more modular and manageable flow structure.
- **Easier Ownership Management**: Simplifies updates to the flow owner.

I recreated the child workflow within the solution selecting the **Manually trigger a flow** trigger and updated the parent flow to use the flow as child workflow.

# Common Issues & Solutions

### 1. Integrating "Respond to a PowerApp" Action

If "Respond to a PowerApp or flow" action is missing from the child workflow, an error message will appear to add the action to resolve it. 

![MissingRespondtoAppAction](../images/PowerAutomate_ChildWorkflow/MissingRespondtoAppAction.png)

Adding the action **Respond to a PowerApp or flow** can resolve the error

![RespondtoaPowerApp](../images/PowerAutomate_ChildWorkflow/RespondtoaPowerApp.png)

### 2. Changing Connections to Use "Run Only User"

If connections are using the "Provided by run-only user" option, then the following error might appear.

![RunOnlyUser](../images/PowerAutomate_ChildWorkflow/RunOnlyUser.png)

It can be fixed by changing connections to use a fixed connection.

![RunOnlyUser](../images/PowerAutomate_ChildWorkflow/NotToUseRunOnlyUsers.png)

### 3. Handling Errors Related to Old Flow Versions

![ChildFlowError](../images/PowerAutomate_ChildWorkflow/ChildFlowError.png)

```powershell
Request to XRM API failed with error: 'Message: Flow client error returned with status code "BadRequest" and details "["error":{"code":"ChildFlowsUnsupportedForNonOpenApiConnections","message":"The workflow with id 'c18d3e7a-067e-ee11-8178-002248c6b301', name <...> child workflow cannot be used as a child workflow because it is on an old version of Flow. Please re-create it inside a solution."})". Code: 0x80060467 InnerError:'.
```
The error was misleading as I have re-created the flow inside the solution. Through trial and error removing each action, I identified the issue was the "Send an email" old action, updating it to the newer **Send an email (V2)** helped resolved the issue though it was complaining at times and removing and adding back the **Send an email v2** seems to fix the issue.


### 4. Optimizing Child Workflow Performance

The child worflow had to be called around 50-60 throughout the execution of the parent flow. It was taking between 2-10 mins to call each child workflow which meant the parent workflow was taking hours to complete. 
![ChildFlowError](../images/PowerAutomate_ChildWorkflow/Timings_WithRespondAtEnd.png)

Through trial and error, I identified it to be because of the **Respond to a PowerApp or flow** being at the end of the flow. The child workflow requires the **Respond to a PowerApp or flow** and it does not matter where it is added in the flow if it does not need to wait for an end user's input. By moving the action at the beginning of the flow sped the calling on the child flow from minutes to seconds.

![ChildFlowError](../images/PowerAutomate_ChildWorkflow/Timings_WithRespondAtBeginning.png)

## Conclusion

Transitioning from an HTTP trigger to a Child Workflow within a solution offers better governance, and avoids premium licensing concerns. 

# References

[Child Workflow](https://learn.microsoft.com/en-us/power-automate/create-child-flows)
