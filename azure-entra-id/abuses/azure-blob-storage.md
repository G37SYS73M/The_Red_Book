# Azure Blob Storage

Blob storage is used to store unstructured data (like files, videos, audio etc.)

* There three types of resources in blob storage:
  * Storage account - Unique namespace across Azure. Can be accessed over HTTP or HTTPS.
  * Container in the storage account - 'Folders' in the storage account
  * Blob in a container - Stores data. Three types of blobs - Block, Append and Page blobs.
* A storage account has globally unique endpoints.
* Very useful in enumeration too by guessing the storage account names!

| Storage Service              | Endpoint                                          |
| ---------------------------- | ------------------------------------------------- |
| Blob storage                 | https://\<storage-account>.blob.core.windows.net  |
| Azure Data Lake Storage Gen2 | https://\<storage-account>.dfs.core.windows.net   |
| Azure Files                  | https://\<storage-account>.file.core.windows.net  |
| Queue storage                | https://\<storage-account>.queue.core.windows.net |
| Table storage                | https://\<storage-account>.table.core.windows.net |

* There are multiple ways to control access to a storage account
  * Use Entra ID credentials - Authorize user, group or other identities based on Entra ID authentication. RBAC roles supported!
  * Share Key - Use access keys of the storage account. This provides full access to the storage account
  * Shared Access Signature (SAS) - Time limited and specific permissions!
* By default, anonymous access is not allowed for storage accounts.
* If 'Allow Blob public access' is allowed on the storage account, it is possible to configure anonymous/public read access to :
  * Only the blobs inside containers. Listing of container content not allowed.
  * Contents of container and blobs

**Storage Explorer**

* Storage explorer is a standalone desktop app to work with Azure storage accounts.
* It is possible to connect using access keys, SAS URLs etc.

**Attack Path**

* The knowledge that Storage accounts have globally unique endpoints and can allow public read access comes handy!
* Let's try to find out insecure storage blobs in the defcorphq tenant.
* We can add permutations like common, backup, code to the 'permutations.txt' in C:\AzAD\Tools\Microburst\Misc to tune it for `defcorphq`.
* We can then use the below command from MicroBurst:

```
Invoke-EnumerateAzureBlobs -Base defcorp
```

