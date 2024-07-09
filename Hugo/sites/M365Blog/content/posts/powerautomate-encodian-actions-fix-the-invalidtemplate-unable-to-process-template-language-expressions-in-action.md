---
title: "Power Automate fix for InvalidTemplate: Unable to process template language expressions"
date: 2024-07-07T09:53:05+01:00
tags: ["Power Automate","Encodian","InvalidTemplate"]
featured_image: '/posts/images/powerautomate-encodian-actions-fix-the-invalidtemplate-unable-to-process-template-language-expressions-in-action/InvalidTemplate.png'
omit_header_text: true
draft: false
---

# Power Automate fix for InvalidTemplate: Unable to process template language expressions

**InvalidTemplate. Unable to process template language expressions in action** can happen with actions within PowerAutomate. In my scenerio I added the encodian action 'Convert_to_PDF' and was resubmitting a flow for testing and kept getting the error message.

> InvalidTemplate. Unable to process template language expressions in action 'Convert_to_PDF' inputs at line '0' and column '0': 'The template language expression 'json(decodeBase64(secrets('X-MS-APIM-Tokens')))['$connections']['shared_encodiandocumentmanager']['connectionId']' cannot be evaluated because property 'shared_encodiandocumentmanager' doesn't exist, available properties are 'shared_sharepointonline, shared_office365, shared_approvals, shared_teams_1'. Please see https://aka.ms/logicexpressions for usage details.'.

This error is not specific to the Encodian connector, it can affect other connectors in Power Automate:

![Invalid Template](../images/powerautomate-encodian-actions-fix-the-invalidtemplate-unable-to-process-template-language-expressions-in-action/InvalidTemplate.png)

The way I resolve the issue was instead resubmitting a previous flow run to start for it to be successful.

![Successful flow](../images/powerautomate-encodian-actions-fix-the-invalidtemplate-unable-to-process-template-language-expressions-in-action/SuccessfulAction.png.png)