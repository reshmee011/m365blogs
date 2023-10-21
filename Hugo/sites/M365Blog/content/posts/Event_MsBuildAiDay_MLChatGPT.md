---
title: "MS Build AI Day: Build and maintain your company Copilot with Azure ML and GPT-4 By Seth Juarez"
date: 2023-10-20T07:06:22+01:00
draft: false
Tags: [AI Concerns, Language Calculator,Azure ML]
featured_image: '/posts/images/Event_MsBuildAIDay/WordsMatchedToNumbers.jpg'
tags: [Azure OpenAi, Generative AI, AzureML, ChatGPT]
---

# Build and maintain your company Copilot with Azure ML and GPT-4 By Seth Juarez

I attended the session Build and maintain your company Copilot with Azure ML and GPT-4 at MS Build AI Day by **Seth Juarez** on the 19th Oct 2023.
Seth began the session with a "Group Therapy" followed by a delve deep into the intricacies of language models, shedding light on crucial aspects of their utilization. He referred to [Henk's presentation](https://reshmeeauckloo.com/posts/Event_GenerativeAIinAzureOpenAIservice) and [Keynote](https://reshmeeauckloo.com/posts/event_msbuildaidaykeynote) to reinforce some concepts.

## Addressing Concerns in Group Therapy

Seth encouraged attendees to voice their concerns and queries regarding language models. 

### Tackling Outdated Data

It was emphasized that while language models can be powerful tools, it's crucial to be mindful of the data they are trained on. As of 2021, this technology may not be updated with the most current information, highlighting the importance of verifying information from other sources.

### Avoiding Hallucinations

The session stressed the need to be vigilant about potential hallucinations or inaccuracies that language models might generate. Users should exercise caution and cross-verify critical information.

### Evaluating Language Models

Evaluating the performance of Language Learning Models (LLMs) remains a challenging task. Assessing the capabilities and limitations of these models is essential for making informed decisions about their use.

### Understanding Price per Token

The cost using LLMs is priced per token. This financial consideration should be factored into any deployment of these models.

### Recognizing Inherent Bias in English Development

Language models developed based on English cultures may carry inherent biases. It's essential to be aware of this potential bias and take steps to mitigate it in applications.

### Adding Data with Privacy in Mind

When incorporating additional data into prompts, it's crucial to prioritize privacy. Attendees were advised on best practices for integrating data while ensuring data security and compliance. "Add your own data" through Azure Open AI does not mean feeding the LLM with your data. Your data stays in your environment and is used to provide more contextual specific response. 

## Treating Language Models as Calculators, Not Databases

Language models should be viewed as language calculators that process and generate text, rather than databases storing static information.

## Promoting Groundedness and Coherence

The importance of ensuring grounded and coherent responses from language models was emphasized. Techniques like content filtering on both input and output can help achieve this.

## Leveraging Vector Indexing with Azure Cognitive Search

Vector-based indexing, coupled with Azure Cognitive Search, enables efficient semantic search. Chunking data into indexes proves to be a powerful method for extracting meaning from large datasets.

## Testing with ChatGPT-4

Seth covered how you can ChatGPT-4 for assessing the performance of earlier models like ChatGPT 3-5. This iterative approach aids in refining language models. 

## Supervised Testing and Declarative AI Orchestration

Supervised testing and declarative AI orchestration were highlighted as critical steps in ensuring the accuracy and reliability of language models.


For further resources and updates, attendees were directed to the links:

[Large-scale Self-supervised Pre-training Across Tasks, Languages, and Modalities](https://aka.ms/prompt)
(https://aka.ms/mptannounce)
