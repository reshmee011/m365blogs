---
title: "Limitations of Azure Storage Explorer: Filtering and Renaming Challenges"
date: 2024-04-02T12:00:00Z
tags: ["Azure Storage Explorer","Filter", "Rename files"]
featured_image: '/posts/images/Azure-Storage-Explorer-Search/Search_AzureStorageExplorer_UI.png'
draft: false
---

# Limitations of Azure Storage Explorer: Filtering and Renaming Challenges

Azure Storage Explorer simplifies managing Azure Storage data across Windows, macOS, and Linux platforms. Refer to [Get started with Storage Explorer](https://learn.microsoft.com/en-us/azure/storage/storage-explorer/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows&wt.mc_id=MVP_308367) for more info.

However, there are nuances surrounding filtering and renaming functionalities that users should be mindful of.

## Filter Functionality
 
Azure Storage Explorer lacks wildcard support, limiting search capabilities to prefixes only. This limitation can pose challenges when attempting to locate files based on substrings within the blob container.

- Search blobs by prefix
![Search_AzureStorageExplorer_UI.png](../images/Azure-Storage-Explorer-Search/Search_AzureStorageExplorer_UI.png)

- Filter by prefix
![Search_AzureStorageExplorer.png](../images/Azure-Storage-Explorer-Search/Search_AzureStorageExplorer.png)

The version used to test is 1.32.1 
![AzureStorageVersion.png](../images/Azure-Storage-Explorer-Search/AzureStorageVersion.png)

Utilizing blob tags can aid in blob storage searches. It's advisable to set blob tags accordingly, as discussed in [Use blob index tags to manage and find data with .NET](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-tags?wt.mc_id=MVP_308367)

To overcome these limitations, consider employing Azure AI Search to enable full-text search functionality on blob data. Refer to the post [Search over Azure Blob Storage content](https://learn.microsoft.com/en-us/azure/search/search-blob-storage-integration?wt.mc_id=MVP_308367) for insights on indexing files for efficient data retrieval.

## Renaming Files and Folders

Renaming files and folders within Azure Storage isn't supported directly. One workaround involves cloning the file or folder and subsequently deleting the original, although this approach may result in the loss of version history.

Missing rename functionality, with only clone or delete options available for each blob:

![RenameNotAvailable.png](../images/Azure-Storage-Explorer-Search/RenameNotAvailable.png)

An alternative solution is presented by [Cerebrata](https://cerebrata.com/blog/renaming-files-and-folders-in-azure-file-storage#:~:text=To%20rename%20a%20file%20or,popup%20window%20that%20opens%20up) offering a tool for renaming files and folders in Azure Storage. While untested, this tool provides an additional avenue for users encountering renaming challenges.

## Conclusion

Despite its utility, Azure Storage Explorer presents limitations in filtering and renaming operations. Users should explore alternative solutions, such as Azure AI Search and external tools, to address these challenges effectively.

## References

[Enable content on a site to be searchable](https://learn.microsoft.com/en-us/sharepoint/make-site-content-searchable?wt.mc_id=MVP_308367)

[Get started with Storage Explorer](https://learn.microsoft.com/en-us/azure/storage/storage-explorer/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows&wt.mc_id=MVP_308367)

[Download Azure Storage Explorer](https://azure.microsoft.com/en-gb/products/storage/storage-explorer)