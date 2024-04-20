# [What is it](https://learn.microsoft.com/en-us/azure/app-service/overview)
  - a hosting option for Web application, rest API, mobile backend
  - supported languages: .NET, .NET Core, Java, Node.js, PHP, and Python.
  - OS system: Linux, Window
  - Paas
  - How to choose a good hosting option
    
    ![image](https://github.com/GiangHM/Documents/assets/36400582/4b4fd2ca-8669-47e3-9373-722357fceb3b)

# [What is the app service plan?](https://learn.microsoft.com/en-us/azure/app-service/overview-hosting-plans)
  - Define a set of compute for running a web app (multiple apps)
  - Each App Service plan defines:
    + Operating System (Windows, Linux)
    + Region (West US, East US, and so on)
    + Number of VM instances
    + Size of VM instances (Small, Medium, Large)
    + Pricing tier (Free, Shared, Basic, Standard, Premium, PremiumV2, PremiumV3, Isolated, IsolatedV2)
  - Pricing tiers: => features you get and how much you pay
    + Shared tier: Free, Shared
    + Dedicated compute: The Basic, Standard, Premium, PremiumV2, and PremiumV3
    + Isolated: The Isolated and IsolatedV2
# How does my app scale?
  - When I create an app -> it's under an ASP => It runs on all VMs of my ASP.
  - If I have two apps in an ASP => two apps run on all VMs of my ASP and scale together
  - If I have some deployment slots for an app, all deployment slots also run on the same VM instances.
  - If I enable diagnostic logs, perform backups, or run WebJobs, they also use CPU cycles and memory on these VM instances.
    => The App Service plan is the scale unit of the App Service apps.
  - Scale up: increase CPU + memory
  - Scale-out: increase instances
  - [Automatic scaling](https://learn.microsoft.com/en-us/azure/app-service/manage-automatic-scaling?tabs=azure-portal)
    + A new scale-out option that automatically handles scaling decisions for your web apps and App Service Plans
    + Adjust the scaling setting -> avoid cold start issue
    + Always a prewarm instance -> cost
  - [Scaling with rules](https://learn.microsoft.com/en-us/azure/azure-monitor/autoscale/autoscale-get-started?toc=%2Fazure%2Fapp-service%2Ftoc.json)
    + Autoscale based on your defined rules
    + Scale out/in rules should be in pair
    + Notice the cool-down and flapping      
# Deployment best practices
  - Use [deployment slot](https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots?tabs=portal) when possible, this is a blue/green deployment technique
    + eliminate downtime
    + possibility to test before swapping to PROD slot.
    + Deployment logs: D:\Home\LogFiles\eventlong.xml
  - [Use of run from package for many benefits](https://learn.microsoft.com/en-us/azure/app-service/deploy-run-package)
# [Configuration](https://learn.microsoft.com/en-us/azure/app-service/configure-common?tabs=portal)

  ![image](https://github.com/GiangHM/Documents/assets/36400582/3a86f5fd-671a-457a-91ed-07c509b86abd)

# Logs and monitoring
  - Monitoring data: Metric, Resource logs, Activities logs
  - [Monitoring options](https://learn.microsoft.com/en-us/azure/app-service/overview-monitoring)
    + [Diagnostic setting](https://learn.microsoft.com/en-us/azure/app-service/tutorial-troubleshoot-monitor):
        ++ can be used to collect metrics for certain Azure services into Azure Monitor Logs
        ++ metrics can be stored in the Log analytic workspace or blob
    + Metrics
    + App Service Log -> [Enable App Service log with azure portal](https://learn.microsoft.com/en-us/azure/app-service/troubleshoot-diagnostic-logs#enable-application-logging-windows)
    + App Insight via azure monitor -> data send to Log analytic workspace
      ++ Monitor availability
      ++ performance
      ++ usage of my web app

      ![image](https://github.com/GiangHM/Documents/assets/36400582/3ed9bab7-07ac-4584-9d43-e2176357d425)

# [Networking](https://learn.microsoft.com/en-us/azure/app-service/networking-features)
  - Virtual network integration
  - Access restriction
  - Using a private endpoint
  - Control outbound with Azure firewall

    ![image](https://github.com/GiangHM/Documents/assets/36400582/6984094b-4c03-434e-afc5-2b636aa433de)
# [Authentication](https://learn.microsoft.com/en-us/azure/app-service/scenario-secure-app-authentication-app-service?tabs=workforce-configuration)
  - Built-in authentication
  - Adapt to many identity providers: Microsoft Entra ID, Facebook, Google, Twitter, Open ID connect, Apple sign in
