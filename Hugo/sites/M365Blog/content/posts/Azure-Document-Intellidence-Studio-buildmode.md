---
title: "Azure Document Intelligence Studio - Neural vs. Template Training"
date: 2024-06-07T12:02:22Z
tags: ["Azure", "Document Intelligence Studio", "Neural","Template"]
featured_image: '/posts/images/Azure-Document-Intellidence-Studio-buildmode/docIntelligenceStudio_buildmode.png'
omit_header_text: true
draft: false
---

# Azure Document Intelligence Studio - Neural vs. Template Training

Azure Document Intelligence Studio formally known as form recognizer is a powerful tool to extract data from documents at scale. Among its features, the ability to "train a new model" for data extraction from documents. This process comes with two distinct options: Neural and Template. Each has its unique advantages and use cases.

## Neural Training

The Neural option leverages deep learning algorithms to understand and process documents. This method is incredibly powerful, allowing for a high degree of accuracy in recognizing and extracting information from complex documents. However, this sophistication comes at a cost – time. Neural training requires a significant amount of data and computational power, making it a longer process compared to its counterpart.

## Template Training

On the other hand, Template training offers a quicker solution. It works by defining specific areas of a document from which to extract information. This method is ideal for documents that follow a consistent layout, such as forms or invoices. While it may not have the flexibility of neural training, its speed and efficiency make it a valuable option for projects with tight deadlines or limited resources.

## Testing the Model

After training your model, whether through Neural or Template methods, Azure Document Intelligence Studio provides a seamless testing environment. This allows you to evaluate the model's performance and make necessary adjustments before deploying it into production.

![Build Mode](../images/Azure-Document-Intellidence-Studio-buildmode/docIntelligenceStudio_buildmode.png)

## Conclusion

Azure Document Intelligence Studio's dual approach to model training offers flexibility and power in document processing tasks. Whether your priority is the depth of understanding with Neural training or the speed and simplicity of Template training, this tool provides the capabilities to meet diverse needs.

## References

(Document Intelligence Studio)[https://documentintelligence.ai.azure.com/studio/custommodel/projects/1f622679-d25e-43ff-b2d4-bfc30449cab7/label] formerly known as form recognizer