

Wildcard is not support, only prefix which makes it hard to search if you know only a substring of the file within the blob container which does not help. 

Tags can be used to search blob storage, hence set blob tags appropriately as per (Use blob index tags to manage and find data with .NET)[https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-tags]

The solution is to use Azure AI Search to add full text search to blob data as described in the post (Search over Azure Blob Storage content)[https://learn.microsoft.com/en-us/azure/search/search-blob-storage-integration] to index the files to search data.

