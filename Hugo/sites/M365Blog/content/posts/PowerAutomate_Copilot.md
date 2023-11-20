---
title: "Exploring some Limitations of Copilot in Power Automate"
date: 2023-11-20T06:19:42Z
tags: ["Power Automate","Copilot"]
featured_image: '/posts/images/PowerAutomate_Copilot/NewFlowExperience_withPrompting.png'
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
    ![Too Many Actions Message](../images/PowerAutomate_Copilot/TooManyActions.png)
    However, it wasn't specified how many actions qualify as "too many."

4. **Shared Connections:** Some flows displayed the message:
    ![Shared Connection Message](../images/PowerAutomate_Copilot/PowerAutomate_TooManyActionsAndOthers.png)
    This message indicated the presence of another user's shared connection, yet its cause remains unidentified.

## Conclusion

Copilot serves as a beneficial aid in creating and editing flows within the designer experience, saving time. However, it exhibits limitations and remains in its early stages, with potential for significant growth. It's crucial to validate Copilot's responses and recommendations.

## References

- [Understanding the Cloud Flows Designer](https://learn.microsoft.com/en-us/power-automate/flows-designer)
- [Get Started with Copilot in Cloud Flows](https://learn.microsoft.com/en-us/power-automate/get-started-with-copilot)