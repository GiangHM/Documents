# Physical infrastructure
  ## Region
  - Regions are made up of one or more data centers nearby.
  - Provide flexibility and scale to reduce customer latency.
  - Preserve data residency with a comprehensive compliance offering.
  - All resources must choose a region when creating
      
  ## Availability zone => to achieve resiliency and reliability
  - Belongs to a region
  - Protect against downtime due to data center failure
  - Physically separate data centers within the same region
  - Each data center is equipped with independent power, cooling, and networking
  - Connected through private fiber-optic networks

    ![image](https://github.com/GiangHM/Documents/assets/36400582/b32dc5dd-b416-460f-8cae-c8238fc25507)
  
  ## Region pair
  - At least 300 miles of separation between region pairs.
  - Automatic replication for some services.
  - Prioritized region recovery in the event of an outage.
  - Updates are rolled out sequentially to minimize downtime.
  - Ex: South East Asia <> East Asia
# Logical infrastructure
  ## Resources:
   - are components like storage, virtual machines, and networks that are available to build cloud solutions.
  ## Resource Group
   - is a container to manage and aggregate resources in a single unit. 
  ## Azure subscription
   - Billing boundary: generate separate billing reports and invoices for each subscription.
   - Access control boundary: manage and control access to the resources that users can provision with specific subscriptions

    ![image](https://github.com/GiangHM/Documents/assets/36400582/c1868eb2-29f7-4e88-91ad-d46e4a49feae)

  ## Management Group

  ![image](https://github.com/GiangHM/Documents/assets/36400582/d4e65081-0b6d-4c17-ad62-fe66d244b1c3)



