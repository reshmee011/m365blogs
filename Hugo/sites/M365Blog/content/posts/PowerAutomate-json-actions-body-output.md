---
title: "Power Automate JSON refer to outputs/body of actions"
date: 2024-05-28T09:53:05+01:00
tags: ["Power Automate","File","Major Version"]
featured_image: '/posts/images/PowerAutomate-json-actions-body-output/changesetting.png'
omit_header_text: true
draft: true
---

# Title

outputs('Get_file_properties')?['body/{FilenameWithExtension}']
![outputs actions](../images/PowerAutomate-json-actions-body-output/outputs_action.png)
body('Get_file_properties')?['{FilenameWithExtension}']
![body actions](../images/PowerAutomate-json-actions-body-output/body_action.png)



addToTime(body('Wait_for_an_approval_2')?['completionDate'], int(body('Get_file_properties')?['ReviewFrequency']?['Value']), 'Month', 'dd/MM/yyyy HH:mm')


formatDateTime(outputs('Wait_for_an_approval_2')?['body/completionDate'], 'dd/MM/yyyy HH:mm')
to
formatDateTime(body('Wait_for_an_approval_2')?['completionDate'], 'dd/MM/yyyy HH:mm')

outputs('Wait_for_an_approval_2')?['body/responses']?[0]?['responder/email']

to 
body('Wait_for_an_approval_2')?['responses']?[0]?['responder/email']

![Approval details](../images/PowerAutomate-json-actions-body-output/get_approval_details.png)