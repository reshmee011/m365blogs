

The 'inputs.parameters' of workflow operation 'Create_item' of type 'OpenApiConnection' is not valid. Error details: Input parameter 'item/StartDate' is required to be of type 'String/date'. The runtime value '"44378"' to be converted doesn't have the expected format 'String/date'.



OpenApiOperationParameterTypeConversionFailed
The 'inputs.parameters' of workflow operation 'Create_item' of type 'OpenApiConnection' is not valid. Error details: Input parameter 'item/ActualTotalCost' is required to be of type 'Number/double'. The runtime value '""' to be converted doesn't have the expected format 'Number/double'.

```powershell
if(empty(items('Foreach')?['Actual Total Cost']), null, items('Foreach')?['Actual Total Cost'])
```