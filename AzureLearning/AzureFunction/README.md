# Azure function summary
## What's Azure Function
- Allow us run a piece of code/ don’t worry about infrastructure (Cloud infrastructure provide all, application scale )
- Severless application (PaaS).
- [Pay per use pricing model](https://docs.microsoft.com/en-us/azure/azure-functions/functions-scale).
  + Consumption plan
  + Premium plan
  + and Dedicated (App Service) plan
- A function is triggered by specific event type
## [Function scenarios](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scenarios?pivots=programming-language-csharp)
- Process file upload
- Run schedule tasks
- Build a scalable web API
- Create reliable message system (integrate with Azure service bus, Kafka)
## [Trigger and binding](https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings)
![image](https://github.com/GiangHM/Documents/assets/36400582/407db352-07d5-4ef6-9927-99afe81a87f7)

- Trigger
- Binding
  + Input binding
  + Output binding
- [Binding example](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-service-bus)
## [Denpendency Injection](https://learn.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection)
- Create a method to configure and add components to IFunctionsHostBuilder in order to register services to DI
- Service life time
  + Transient
  + Scoped
  + Singleton
## Authentication and Authorization
## [Best practices](https://learn.microsoft.com/en-us/azure/azure-functions/functions-best-practices?tabs=csharp)
- Choose the correct hosting plan, because:
  + Determine how app is scaled and how instances allocation is managed
  + available resource for each func app
  + support advance functionality
- Configure storage correctly: where we store app content and azure use storage to manage the trigger.
- Organize your functions:
  + Functions are deployed together.
  + Functions are scaled together
- [Write robust code](https://learn.microsoft.com/en-us/azure/azure-functions/performance-reliability)
  + Avoid long running
  + Plan cross functions communication
  + Write stateless functions.
  + Write defensive functions
- [Manage connections](https://learn.microsoft.com/en-us/azure/azure-functions/manage-connections?tabs=csharp)
  + HTTP client
  + Azure client
  + SQL client
- [Error handling and retry](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-error-pages?tabs=fixed-delay%2Cisolated-process&pivots=programming-language-csharp)
  - Retries:
    + Built-in retry behaivors of individual trigger extensions
    + Retry policies provided by the Function runtime
  - Retry strategies:
    + Fixed delay: a specific amount of time is allowed to elapse between each retry.
    + Exponential backoff: Exponential back-off adds some small randomization to delays to stagger retries in high-throughput scenarios.
## Develop with .Net
- [In process model](https://learn.microsoft.com/en-us/azure/azure-functions/functions-dotnet-class-library?tabs=v4%2Ccmd)
- [Isolated worker process](https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide?tabs=windows)
- Dependency Injection
  - [In process model](https://learn.microsoft.com/en-us/azure/azure-functions/functions-dotnet-dependency-injection#register-services): using FunctionsStartup 
  - [Isolated worker process](https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide?tabs=windows#dependency-injection): use the .Net stardand way of call ConfigureService method
## [Deployment slot](https://learn.microsoft.com/en-us/azure/azure-functions/functions-deployment-slots?tabs=azure-portal)
# Examples
## [C# Isolated worker process model](https://github.com/GiangHM/PracticalAzureSDKs/tree/main/IsolatedWorkerFunctions)
