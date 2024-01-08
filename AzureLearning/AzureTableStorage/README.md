# Azure table storage summary

## [What is it and uses for what?](https://learn.microsoft.com/en-us/azure/storage/tables/table-storage-overview)
- is a service that stores non-relational structured data, (NoSQL data)
- storing TBs of structured data for serving web scale applicaiton
- storing dataset don't requeire complex joins.
- Querying data quickly
- Accessing data using OData protocal
## [Table storage concepts](https://learn.microsoft.com/en-us/rest/api/storageservices/Understanding-the-Table-Service-Data-Model)
- Storage account
- Tables
- Entities
- Properties
  - have up to 255 props including 3 system props:
    + PartitionKey, RowKey: developer's responsibility for inserting and updating data
    + Timestamp: system
      
![image](https://github.com/GiangHM/Documents/assets/36400582/2b366f67-9725-40ad-a6ec-1017e57048d6)
## [How to choose between Azure Commos DB for Table and Azure Table storage](https://learn.microsoft.com/en-us/azure/cosmos-db/table/support?toc=https%3A%2F%2Flearn.microsoft.com%2Fen-us%2Fazure%2Fstorage%2Ftables%2Ftoc.json&bc=https%3A%2F%2Flearn.microsoft.com%2Fen-us%2Fazure%2Fbread%2Ftoc.json)
- Latency: Commos DB: <10ms for reading and 15ms for writing
- Indexing: Commos DB automactic and complete indexing on all props
- Query: Take advantages of indexing
- Throughput: Commos DB: > 10 million operation per second (20000 per second for table storage)
- Pricing: Commos have both: Consumption and provisioned capacity modes
## [Authorization](https://learn.microsoft.com/en-us/azure/storage/common/authorize-data-access?toc=%2Fazure%2Fstorage%2Ftables%2Ftoc.json)
- Microsoft entra ID
- Shared key
- SAS
## Security and Data redundancy
- [Security](https://learn.microsoft.com/en-us/azure/storage/common/storage-service-encryption?toc=%2Fazure%2Fstorage%2Ftables%2Ftoc.json):
  - Azure storage encryption with service-side encrption
  - Encrypt data with infrastructure encryption
  - Client-side encryption
- [Data redundancy](https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy?toc=%2Fazure%2Fstorage%2Ftables%2Ftoc.json)
  - Primary region:
    + LRS: 99.999999999% (11 nines)
      
      ![image](https://github.com/GiangHM/Documents/assets/36400582/3ed0c3f1-ed24-4128-850b-cb45024c0d2a)
    + ZRS: 99.9999999999% (12 9's)
      
      ![image](https://github.com/GiangHM/Documents/assets/36400582/e5bb9e0f-96d6-47b3-a0e5-2d65c9f846ed)
  - Secondary region
    + (RA)GRS: 99.99999999999999% (16 9's)
      
      ![image](https://github.com/GiangHM/Documents/assets/36400582/1b4882ea-c106-4ee2-9a48-ade36a6525c7)
    + (RA)GZRS: 99.99999999999999% (16 9's)
      
      ![image](https://github.com/GiangHM/Documents/assets/36400582/e3fad2be-2208-41a4-89b5-9b5745bc921a)
## [Design table](https://learn.microsoft.com/en-us/azure/storage/tables/table-storage-design-guidelines)
- Partition key and Rowkey should consider carefully
- [Design for querying](https://learn.microsoft.com/en-us/azure/storage/tables/table-storage-design-for-query)
  - Choose PartitionKey and RowKey impacts your query
  - A *Point Query*: most efficient lookup, it uses index of PartitionKey and RowKey
  - *Range Query*: second best, it use PartitionKey and filter on a range of RowKey
  - *Parttion scan*: use PartitionKey and filter on another prop of entity
  - *Table scan*: don't include PartitionKey
- [Design for data modification](https://learn.microsoft.com/en-us/azure/storage/tables/table-storage-design-for-modification)
- [Design patterns](https://learn.microsoft.com/en-us/azure/storage/tables/table-storage-design-patterns)
## Examples:
- ### [Build a wrapper class](https://github.com/GiangHM/PracticalAzureSDKs/tree/main/AzureTableStorage)
- ### Using wrapper class
  - Inject the service
    ```C#
    using AzureTableStorage.Extensions;
    using AzureBlobStorage.Extensions;
    using AzureTableStorage.Services;
    using APIExamples;
    
    var builder = WebApplication.CreateBuilder(args);
    
    // Add services to the container.
    builder.Services.AddAzureTableStorage();
    builder.Services.AddTransient<ITableStorageService<WeatherForecastEntity>, TableStorageService<WeatherForecastEntity>>();

  - Use in Controller
    ```C#
    namespace APIExamples.Controllers
    {
        [ApiController]
        [Route("[controller]")]
        public class WeatherForecastController : ControllerBase
        {
            private readonly ILogger<WeatherForecastController> _logger;
            private readonly ITableStorageService<WeatherForecastEntity> _tableStorageService;
    
            public WeatherForecastController(ILogger<WeatherForecastController> logger
                , IBlobStorageService blobStorageService
                , ITableStorageService<WeatherForecastEntity> tableStorageService)
            {
                _logger = logger;
                _blobStorageService = blobStorageService;
                _tableStorageService = tableStorageService;
            }
    
    
            [HttpPost("WeatherInfo")]
            public async Task<WeatherForecastModel> AddWeatherInfo(WeatherForecastModel item)
            {
                _logger.LogInformation("If you're seeing this, we were generating the sas blob");
                var res = await _tableStorageService.InsertOrUpadteEntityAsync(new WeatherForecastEntity(item));
                return new WeatherForecastModel
                {
                    Date = res.Date,
                    TemperatureC = res.TemperatureC,
                };
            }
        }
    }




