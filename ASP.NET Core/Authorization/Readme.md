# Authorization + Authorization Requirement Data

- Authorization refers to the process that determines what a user is able to do
- Authorization types: Role and Policy based model
For futher infomation, we can check the microsoft document [here](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/introduction?view=aspnetcore-9.0)

Some scenarios require us to build a custom authorization policy, for an intance (my real world project)
A large range of policies, so it doesn't make sense to add each individual authorization policy with an AuthorizationOptions.AddPolicy call

# Prior to ASP.NET Core 8:
We use IAuthorizationPolicyProvider to implement a custom Authorization policy. It's cosiderred as a old version. For a detailed examination, we can visit [Custom Authorization Policy Providers using IAuthorizationPolicyProvider in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/iauthorizationpolicyprovider?view=aspnetcore-9.0)

# ASP.NET Core 8 introduce new approach by using IAuthorizationRequirementData
For a detailed explanation and example from the official document, please check out [here](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/iard?view=aspnetcore-9.0)
### In my case, I implement a UserFeatureAuthorizeAttribute
```C#
﻿using Microsoft.AspNetCore.Authorization;
using System;
using System.Collections.Generic;

namespace PracticalAPI.AuthorizationRequirementData
{
    public enum FeatureOperator
    {
        And = 1,
        Or = 2
    }
    public class UserFeatureAuthorizeAttribute : AuthorizeAttribute
        , IAuthorizationRequirement
        , IAuthorizationRequirementData
    {
        internal const string PolicyPrefix = "UserFeature_";
        public FeatureOperator Operator { get; set; }
        public string[] Features { get; set; }
        public UserFeatureAuthorizeAttribute(FeatureOperator featureOperator, params string[] features)
        { 
            Operator = featureOperator;
            Features = features;
        }
        public UserFeatureAuthorizeAttribute(string feature)
        {
            Operator = FeatureOperator.And;
            Features = new string[] { feature };
        }
        public IEnumerable<IAuthorizationRequirement> GetRequirements()
        {
            yield return this;
        }
    }
}
```
### And a requirement is:
```C#
using Microsoft.AspNetCore.Authorization;
using System;

namespace PracticalAPI.AuthorizationRequirementData
{
    public class UserFeatureRequirement : IAuthorizationRequirement
    {
        public FeatureOperator Operator { get; private set; }
        public string[] Features { get; private set; }

        public UserFeatureRequirement(FeatureOperator featureOperator, string[] features)
        {
            if(features == null || features.Length == 0)
                throw new ArgumentException("At least one user feature is required", nameof(features));

            Operator = featureOperator;
            Features = features;
        }
    }
}
```

### Then, I implement the AuthorizationHandler
```C#
using Microsoft.AspNetCore.Authorization;
using Microsoft.Extensions.Logging;
using System.Globalization;
using System.Threading.Tasks;
using System;

namespace PracticalAPI.AuthorizationRequirementData
{
    public class UserFeatureAuthorizationHandler : AuthorizationHandler<UserFeatureAuthorizeAttribute>
    {
        private readonly ILogger<UserFeatureAuthorizationHandler> _logger;
        private static string featureType = "feature";

        public UserFeatureAuthorizationHandler(ILogger<UserFeatureAuthorizationHandler> logger)
        {
            _logger = logger;
        }

        protected override Task HandleRequirementAsync(AuthorizationHandlerContext context,
                                                   UserFeatureAuthorizeAttribute requirement)
        {
            if (requirement.Operator == FeatureOperator.And)
            {
                foreach (var feature in requirement.Features)
                {
                    _logger.LogWarning("Evaluating authorization requirement for feature: {Feature}", feature);
                    if (!context.User.HasClaim(featureType, feature))
                    {
                        context.Fail();
                        return Task.CompletedTask;
                    }
                }

                context.Succeed(requirement);
                return Task.CompletedTask;
            }

            foreach (var feature in requirement.Features)
            {
                _logger.LogWarning("Evaluating authorization requirement for feature: {Feature}", feature);
                if (context.User.HasClaim(featureType, feature))
                {
                    context.Succeed(requirement);
                    return Task.CompletedTask;
                }
            }

            context.Fail();
            return Task.CompletedTask;
        }
    }
}
```

### Add the Authorization Handler to Program
```C#
  using AuthRequirementsData.Authorization;
  using Microsoft.AspNetCore.Authorization;
  
  var builder = WebApplication.CreateBuilder();
  ....
  // Use Custom Authorization
  // builder.Services.AddSingleton<IAuthorizationHandler, UserFeatureAuthorizationHandler>()
  
  var app = builder.Build();
  
  app.MapControllers();
  
  app.Run();
```

### And use our custom authorization attribute in controller
```C#
    [UserFeatureAuthorize("OTMP")]
    public async Task<ActionResult<bool>> MyAction([FromBody] RequestModel model)
    {
       // Logic code go here
    }
```

[My completed example](https://github.com/GiangHM/PracticalASPNet/tree/main/PracticalAPI/AuthorizationRequirementData)
