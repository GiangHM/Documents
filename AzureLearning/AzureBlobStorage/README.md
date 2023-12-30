# Azure blob storage summary

## [What is it?](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction)
- storage solution for cloud
- store massive amount of unstrutured data
- designed for
    + file and images
    + video and audio
    + log files
    + data for backup & restore
    + data for analysis
- [Resource](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction#blob-storage-resources)

![image](https://github.com/GiangHM/Documents/assets/36400582/198ab966-a254-4937-876a-16a4d9a7ee44)

## [Redundancy and failover](https://learn.microsoft.com/en-us/azure/storage/common/redundancy-migration?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&bc=%2Fazure%2Fstorage%2Fblobs%2Fbreadcrumb%2Ftoc.json&tabs=portal)
- Redundancy options
  + Primary
    * LRS
    * ZRS
  + Secondary
    * GRS
    * (RA)GZRS
## [Storage account type](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction#storage-accounts)
  - General purpose
  - Block blob
  - Page blob
## [Access tier and lifecycle management](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview)
- access tier
  + online tier
    * hot tier
      * accessed and modify frequenly
      * highest storage cost/ lowest acess cost
    * cool tier
      * infrequently
      * store 30 days, lowest storage cost/ highest access cost
    * cold tier
      * rarely access
      * store 90days, lowest storage cost, highest access cost
  + offline tier
    * archive tier
      * rarely access, latency
      * stored 180days
      * can't be read or modified -> need [rehydreate](https://learn.microsoft.com/en-us/azure/storage/blobs/archive-rehydrate-overview) it to online tier
- [Best pratices to optimize cost](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-overview)
  + Choose the most cost efficient
  + Move/migrate data to the most cost efficient
    * first: understand how the data used in production
    * second: use lifecycle management to move file to appropriate access tier
    * json difinition
  + [Ways to configure the lifecycle](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-policy-configure?tabs=template)
    * on azure portal
    * azure cli
    * powershell
    * azure template
  + Pack data before moving to cooler tier
## Authorization
- Microsoft entra id & RBAC -> recommendation
- Share access key -> be careful, can use with Azure Key Vault for security reason
- [Share access signature](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&bc=%2Fazure%2Fstorage%2Fblobs%2Fbreadcrumb%2Ftoc.json)
## Security
- Data encryption at rest on service side
- [Configure client side encryption](https://learn.microsoft.com/en-us/azure/storage/blobs/client-side-encryption?tabs=dotnet)
- [Configure networking security](https://learn.microsoft.com/en-us/azure/storage/common/storage-network-security?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&bc=%2Fazure%2Fstorage%2Fblobs%2Fbreadcrumb%2Ftoc.json&tabs=azure-portal)
## [Unit test Blob Storage SDK](https://learn.microsoft.com/en-us/dotnet/azure/sdk/unit-testing-mocking?tabs=csharp)
## Develop with .Net
- [Create client object](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-client-management?tabs=dotnet)
- [Working with service client, container client, blob client](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-dotnet-get-started?tabs=azure-ad)
- Use cases
  + Upload image to blob storage using SAS
  + A common .net wrapper class for reusing
    
# Code example
- ## [Wrapper class](https://github.com/GiangHM/AzureBlobStorageLib)
- ## API using wrapper class to generate SAS for uploading file, code snippets:
  + ### Inject to service:
    ```C#
    using API.Dal.Extensions;
    using API.Services.ConcreteClass;
    using API.Services.Interfaces;
    using AzureBlobStorage.Extensions;
    using Microsoft.Extensions.Logging.AzureAppServices;
    
    var builder = WebApplication.CreateBuilder(args);
           
    builder.Services.AddAzureBlobStorage();
       
    ...   
    app.Run();
    ```
  + ### Using wrapper service in controller
    ```C#
    ﻿using API.Models;
    using API.Services.Interfaces;
    using AzureBlobStorage.Services;
    using Microsoft.AspNetCore.Http;
    using Microsoft.AspNetCore.Mvc;
    
    namespace API.Controllers
    {
        [Route("api/[controller]")]
        [ApiController]
        public class DocumentController : ControllerBase
        {
            private readonly IBlobStorageService _blobStorageService;
            private readonly ILogger<DocumentController> _logger;
    
            public DocumentController(IBlobStorageService blobStorageService
                , ILogger<DocumentController> logger)
            {
                _blobStorageService = blobStorageService;
                _logger = logger;
            }
    
            [HttpGet("SasGeneration")]
            public Uri GenereateSasBlob(string fileName, Azure.Storage.Sas.BlobContainerSasPermissions permission)
            {
                _logger.LogInformation("If you're seeing this, we were generating the sas blob");
                return _blobStorageService.CreateServiceSASBlob("Documentfiles", fileName: fileName, 1, null, permission);
            }
        }
    }
    ```


      
