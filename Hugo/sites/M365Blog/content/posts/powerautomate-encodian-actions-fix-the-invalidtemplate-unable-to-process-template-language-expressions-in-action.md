---
title: "Power Automate fix for InvalidTemplate. Unable to process template language expressions in action 'Convert_to_PDF'"
date: 2024-05-28T09:53:05+01:00
tags: ["Power Automate","Encodian","InvalidTemplate"]
featured_image: '/posts/images/powerautomate-encodian-actions-fix-the-invalidtemplate-unable-to-process-template-language-expressions-in-action/InvalidTemplate.png'
omit_header_text: true
draft: true
---

InvalidTemplate. Unable to process template language expressions in action 'Convert_to_PDF'

InvalidTemplate. Unable to process template language expressions in action 'Convert_to_PDF' inputs at line '0' and column '0': 'The template language expression 'json(decodeBase64(secrets('X-MS-APIM-Tokens')))['$connections']['shared_encodiandocumentmanager']['connectionId']' cannot be evaluated because property 'shared_encodiandocumentmanager' doesn't exist, available properties are 'shared_sharepointonline, shared_office365, shared_approvals, shared_teams_1'. Please see https://aka.ms/logicexpressions for usage details.'.


But note this error is not specific to the Encodian connector, it affects all connectors in Power Automate:

InvalidTemplate
This error is not caused by an issue with the action or the configuration of the action… and there are two super quick approaches to resolve:

Option 1 – remove InvalidTemplate with ‘Use new Test Data’
This issue typically occurs whilst you are building flow solutions and is typically caused by re-using previous run data. Just simply try using new data to test your flow!