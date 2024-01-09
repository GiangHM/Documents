# Azure function summary
## What's Azure Function
- Allow us run a piece of code/ don’t worry about infrastructure (Cloud infrastructure provide all, application scale )
- Severless application (PaaS).
- [Pay per use pricing model](https://docs.microsoft.com/en-us/azure/azure-functions/functions-scale).
  + Consumption plan
  + Premium plan
  + and Dedicated (App Service) plan
- A function is triggered by specific event type
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
## Develop 
