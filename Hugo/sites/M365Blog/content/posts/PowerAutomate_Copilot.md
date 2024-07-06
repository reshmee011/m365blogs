---
title: "Exploring some Limitations of Copilot in Power Automate"
date: 2023-11-20T06:19:42Z
tags: ["Power Automate","Copilot"]
featured_image: '/posts/images/PowerAutomate_Copilot/NewFlowExperience_withPrompting.png'
omit_header_text: true
draft: false
---

# Exploring some Limitations of Copilot in Power Automate

I was thrilled when Copilot became available in the UK region within Power Automate around mid-November 2023. To understand its functionalities and limitations, I started exploring. Please refer to [Understanding the Cloud Flows Designer](https://learn.microsoft.com/en-us/power-automate/flows-designer), which outlines some limitations and described the Flows Designer experience.

## Creating New Flows

While creating a new flow using the prompt "Start an approval process and update the item in a SharePoint list when a new item is created," I noticed a functional flow generated. However, it seemed suboptimal due to excessive foreach actions, prompting a need for post-creation optimization.

![Flow Created Using Prompt](../images/PowerAutomate_Copilot/NewFlowExperience_withPrompting.png)

## Limitations Encountered in Existing Flows

During the editing of existing flows, I encountered several messages that prevented their opening in the flow designer experience.

1. **Old Format:** Some flows generated the message:
    ![Old Format Message](../images/PowerAutomate_Copilot/OldFormat.png)
    These flows seem incompatible with AI-powered editing due to their older format, although created just last week.

2. **Unsupported Trigger:** Another message encountered was:
    ![Unsupported Trigger Message](../images/PowerAutomate_Copilot/UnSupportedTrigger.png)
    Flows containing triggers like "For a selected item" or "For a selected file" are currently unsupported for the designer/Copilot experience.

3. **Excessive Actions:** I encountered a message regarding too many actions:
    ![Too Many Actions Message](../images/PowerAutomate_Copilot/PowerAutomate_TooManyActions.png)
    However, it wasn't specified how many actions qualify as "too many."

4. **Shared Connections:** Some flows displayed the message:
    ![Shared Connection Message](../images/PowerAutomate_Copilot/PowerAutomate_TooManyActionsAndOthers.png)
    This message indicated the presence of another user's shared connection, yet its cause remains unidentified.

5. **Unnecessary loop** 
    Example provided above with unnecessary for each action in the flow generated

6. **Lack of HTML formatting** in **Send Email** action
     ![HTML formatting lacking](../images/PowerAutomate_Copilot/HTMLEmailFormat_Missing.png)
    Ability to add HTML seems missing 
   
## Switch to Classic Designer

The AI powered editing experience might not work for some limitations and to change to classic designer

### Change Parameter 'v3' from URL 

Notice the parameter v3 set to true
```dotnetcli
https://make.powerautomate.com/environments/Default-d872ec63-6bea-4678-9429-078f4fa93560/flows/c2d7e687-b42b-4cad-8cba-5150ab9f18e7?v3survey=true&v3=true
```

Amend the v3 parameter to false

```dotnetcli
https://make.powerautomate.com/environments/Default-d872ec63-6bea-4678-9429-078f4fa93560/flows/c2d7e687-b42b-4cad-8cba-5150ab9f18e7?v3survey=true&v3=false
```
### Switch from UI to classic designer

On the top right , click on the ellipsis to access the option to change the editing experience to classic designer

![Classic Designer](../images/PowerAutomate_Copilot/SwitchToClassicDesigner.png)


### Feedback to switch to Classic Designer

A feedback for the AI powered editing experience pop up appears, please fill in with the reasons for switching to classic designer to help Microsoft to improve the product further. The improvements are around these areas

- Lack of HTML formatting view
- Invalid connection
- Too many clicks
- Cannot open multiple action panes
- Slow designer
- Slow copilot
- No copy/paste actions
- Data loss from flow after saving (Dynamic content, parameter)
- Dropdown items/dynamic contentmissing
- Copilot understanding is limited
- Missing connector
- Unnecessary loop

![Feedback to switch to Classic Designer](../images/PowerAutomate_Copilot/ReasonsNotToSUseAIEditingExperience.png)

## Conclusion

Copilot serves as a beneficial aid in creating and editing flows within the designer experience, saving time. However, it exhibits limitations and remains in its early stages, with potential for significant growth. It's crucial to validate Copilot's responses and recommendations.

## References

- [Understanding the Cloud Flows Designer](https://learn.microsoft.com/en-us/power-automate/flows-designer)
- [Get Started with Copilot in Cloud Flows](https://learn.microsoft.com/en-us/power-automate/get-started-with-copilot)