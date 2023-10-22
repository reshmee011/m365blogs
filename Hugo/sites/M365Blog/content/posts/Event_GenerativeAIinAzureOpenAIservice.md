---
title: "MS Build AI Day: Generative AI using Azure OpenAI Service by Henk Boelman"
date: 2023-10-20T06:36:33+01:00
featured_image: '/posts/images/Event_MsBuildAIDay/HowLanguageModelWorks.jpg'
tags: [Azure OpenAi, Generative AI, Prompt, Responsible AI, Grounding, RAG, ChatGPT]
draft: false
---

# Generative AI using Azure OpenAI Service session at MS Build AI Day by Henk Boelman

I attended the session Generative AI using Azure OpenAI Service session at MS Build AI Day by **Henk Boelman** on the 19th Oct 2023. He demoed how to leverage Azure Open AI Service to "add your own data" combined with ChatGPT to retrieve information from your company owned data covering elements of generative AI and reinforcing some of the points highlighted in the [Keynote](https://reshmeeauckloo.com/posts/event_msbuildaidaykeynote) such as prompt engineering, grounding, jail breaking, etc..  

Henk covered how Language Models works with user inputs (tokens) passed to model to return results based on probability in natural language.
  
![How Language Model Works](../images/Event_MsBuildAIDay/HowLanguageModelWorks.jpg)

He demoed the playground of Azure Open AI service with the different features.

## Mastering the Art of Prompt Engineering

One of the key strengths of Azure Open AI Service lies in prompt engineering. This technique enables users to finely tune their inputs to elicit desired responses. Whether it's creating content, generating SQL queries, or even crafting lifelike photographs, the precision and flexibility afforded by prompt engineering is unparalleled.

## Understanding Determinism: The Role of Temperature

The temperature parameter passed to the model Determinism in AI refers to the predictability of outcomes based on a given input. Temperature serves as a crucial parameter in controlling this predictability. Higher temperatures introduce an element of randomness and creativity, while lower temperatures yield more precise and deterministic results.

## System Prompt vs. User Prompt: Navigating AI Interaction

Azure Open AI Service offers a dynamic interaction between system prompts and user prompts. System prompts guide the model towards a specific direction, ideal for scenarios where a controlled response is desired. For example, envision an AI assistant providing information in rhyming verse, steering the conversation in a creative yet focused manner.

![System Prompt](../images/Event_MsBuildAIDay/Henk_SystemPrompt.jpg)

## Response Grounding : Contextual information

User prompts, on the other hand, open the door for more organic interactions. When asked a factual question like "tell me about ms build," the emphasis shifts to providing accurate and informative responses. Grounding ensures that the information provided is both interesting and politely articulated, enhancing the user experience.

## Responsible AI: Avoiding Conflict and Ensuring Safety

Safety rules can be defined within the system prompt to steer the Azure Open AI Service against engaging in argumentative comments. This is especially crucial in customer service contexts where maintaining a positive and constructive conversation is paramount. The platform upholds a commitment to responsible AI, ensuring interactions are both respectful and insightful.

## Jailbreaking: Enforcing Rules

Jailbreaking, a concept within Azure Open AI Service, allows for a temporary suspension of predefined rules. However, this freedom comes with a responsibility to wield it judiciously. After requesting a set of rules, it is imperative to remember to decline the offer of suspension, ensuring that interactions remain within agreed-upon boundaries.

## Empowering AI with Your Data: Retrieval Augmented Generation (RAG)

Azure Open AI Service allows to go beyond predefined models by allowing users to integrate their own data. With techniques like Retrieval Augmented Generation (RAG), the platform harnesses the power of external knowledge bases. This facilitates a more comprehensive and accurate response generation. Henk demoed how to "add your own data" including markdown, i.e. md files of a hiking store) to provide answers based on your own data. 

## Optimizing Search with Azure Cognitive Search

The platform seamlessly integrates with Azure Cognitive Search, leveraging its capabilities to enhance information retrieval. By creating embeddings for documents with vector indexing, Azure Open AI Service ensures that the search experience is both efficient and effective.

## Vector-Based Retrieval: Unveiling Semantic Similarity

Azure Open AI Service leverages vector-based retrieval, enabling the representation of items with similar meanings. Closely positioned vectors signify a higher degree of semantic similarity, enhancing the accuracy of responses.

## TL;DR;

Henk Boelman showcased how to use Azure OpenAI Service to enhance data retrieval using ChatGPT at MS Build AI Day. The session covered prompt engineering, determinism, system vs. user prompts, responsible AI practices, and the integration of external data for more comprehensive responses. Azure OpenAI Service also leverages vector-based retrieval and Azure Cognitive Search for efficient information retrieval.

## References

[Request Access to Azure Open AI](https://aka.ms/oai/access)

[Workshop sessions link](https://github.com/Azure-Samples/PetSpotR)