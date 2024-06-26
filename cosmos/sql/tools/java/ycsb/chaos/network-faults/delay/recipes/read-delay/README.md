# Simulate network delay for a read workload

This recipe encapsulates a read workload that executes for 20-25 minutes. Around 5 minutes into the execution a network latency is simulated by introducing the specified network delay on all the packets bound to the specified fault region. 

## Recipe definition 

|  Config   |  Value   |
| --- | --- |
| Database | Cosmos SQL API |
| Benchmarking tool | YCSB |
| Workload | Read |
| Duration | 20-25 minutes |
| Fault Type | Network Delay |
| Fault Start | 5 Minutes after the workload starts |
| Fault duration | 5 minutes |
| Number of documents in DB | 30 |
| Document Size | ≈1 KB(YCSB default) |

## Execute
1. Create a Geo-Redundancy [Cosmos DB SQL API account and container](https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/quickstart-portal)

   |  Setting   |  value  | 
   | --- | --- |
   | Geo-Redundancy | enable |
   | Database Name | ycsb | 
   | Container Name | usertable | 
   | Partition Key  | /id |
   | Container throughput | 2000 RU/s |
   


3. Create a [storage account](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal) 
4. Create a [resource group](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal) in the same region as the Cosmos DB account 
5. Click the deploy to Azure button and fill in the following missing parameter values:

   |  Parameter   |  Value  |
   | --- | --- |
   | Resource group | name of the resource group from step 3 |
   | Region | Make sure the region is the same as the Cosmos DB account primary region |
   | Results Storage Connection String | connection string of the storage account from step 2 |
   | Cosmos URI  | URI of the Cosmos DB account from step 1 |
   | Cosmos Key  | Primary key of the Cosmos DB account from step 1 |
   | Admin Password | Admin account password for the VM |
   | Preferred Region List | Comma separated preferred regions list. Ex: South Central US,East US.  [More about SDKs Failover configuration](https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/troubleshoot-sdk-availability)|
   | delayInMs | Network delay in milliseconds |
   | faultRegion | Region which should experience the fault. Ex: South Central US |
   | waitForFaultToStartInSec | Time in seconds to wait before starting the fault |
   | durationOfFaultInSec| Duration of the fault in seconds |


 [More details about the parameters](../../#basic-configuration)

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-db-benchmarking%2Fmain%2Fcosmos%2Fsql%2Ftools%2Fjava%2Fycsb%2Fchaos%2Fnetwork-faults%2Fdelay%2Frecipes%2Fread-delay%2Fazuredeploy.json)


## Output
You can visualize the total request count by region by creating [Azure Monitor metrics chart](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/metrics-getting-started), for Azure Cosmos DB. You will see the total request count drop during the fault. If the specified delay is large, all the requests will be routed to the next available region in the "Preferred Regions List" assuming that the fault is active in the first region and the account is configured to have more than one region. 

The job status and results will be available in the following locations in the storage account provided
| Type | Location |
| --- | --- |
| Status  | ycsbbenchmarkingMetadata (Table) |
| Results | ycsbbenchmarking-{Date} (Container) |

 [More details about job status and results](../../#monitoring)
