---
title: "Azure Storage Explorer Filter and rename limitations"
date: 2024-04-02T12:00:00Z
tags: ["Azure Storage Explorer","Filter", "Rename files"]
featured_image: '/posts/images/Azure-Storage-Explorer-Search/Search_AzureStorageExplorer_UI.png'
draft: false
---

# Azure Storage Explorer Filter and rename limitations

Azure Storage Explorer  makes it easy to work with Azure Storage data on Windows, macOS, and Linux. Refer to [Get started with Storage Explorer](https://learn.microsoft.com/en-us/azure/storage/storage-explorer/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows&wt.mc_id=MVP_308367) for more info.

However there are gotchas relating to Filter and rename that we need to be aware about.

## Filter
Wildcard is not support, only prefix which makes it hard to search if you know only a substring of the file within the blob container which does not help. 

- Search blobs by prefix
![Search_AzureStorageExplorer_UI.png](../images/Azure-Storage-Explorer-Search/Search_AzureStorageExplorer_UI.png)

- Filter by prefix
![Search_AzureStorageExplorer.png](../images/Azure-Storage-Explorer-Search/Search_AzureStorageExplorer.png)



The version used to test is 1.32.1 
![AzureStorageVersion.png](../images/Azure-Storage-Explorer-Search/AzureStorageVersion.png)

Tags can be used to search blob storage, hence set blob tags appropriately as per (Use blob index tags to manage and find data with .NET)[https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-tags]

The solution is to use Azure AI Search to add full text search to blob data as described in the post (Search over Azure Blob Storage content)[https://learn.microsoft.com/en-us/azure/search/search-blob-storage-integration] to index the files to search data.

## Rename

Renaming files and folders in azure storage is not possible, one option is to clone the file and folder and delete the original folder. It is not ideal as you may lose version history. 

- Rename functionality missing, only clone or delete option available on each blob
![RenameNotAvailable.png](../images/Azure-Storage-Explorer-Search/RenameNotAvailable.png)

Another solution is to use https://cerebrata.com/blog/renaming-files-and-folders-in-azure-file-storage#:~:text=To%20rename%20a%20file%20or,popup%20window%20that%20opens%20up. I have not tried it but good to know there is a tool which will allow you to rename.



## Conclusion 


## References 
[Enable content on a site to be searchable](https://learn.microsoft.com/en-us/sharepoint/make-site-content-searchable?wt.mc_id=MVP_308367)

[Get started with Storage Explorer](https://learn.microsoft.com/en-us/azure/storage/storage-explorer/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows&wt.mc_id=MVP_308367)

[Download Azure Storage Explorer](https://azure.microsoft.com/en-gb/products/storage/storage-explorer)