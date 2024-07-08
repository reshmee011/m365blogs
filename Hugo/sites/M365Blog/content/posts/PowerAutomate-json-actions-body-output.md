---
title: "JSON Data Handling in Power Automate: outputs versus body"
date: 2024-05-28T09:53:05+01:00
tags: ["Power Automate", "JSON", "Data Handling"]
featured_image: '/posts/images/PowerAutomate-json-actions-body-output/body_action.png'
omit_header_text: true
draft: false
---

# JSON Data Handling in Power Automate : outputs versus body

In Power Automate, JSON data output from various actions is key to connect each other.

## Accessing Action Outputs

Accessing the outputs of a specific action, such as the action 'Get file properties', the spcific property can be accessed via the outputs, for instance file name with extension.:

> outputs('Get_file_properties')?['body/{FilenameWithExtension}']

![outputs actions](../images/PowerAutomate-json-actions-body-output/outputs_action.png)

## Referencing Action Body

A streamlined way is to reference the body directly making it simpler.

> body('Get_file_properties')?['{FilenameWithExtension}']

![body actions](../images/PowerAutomate-json-actions-body-output/body_action.png)
