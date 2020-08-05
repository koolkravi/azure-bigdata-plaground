# Azure Stream Analytics
- Azure Stream Analytics is a PaaS service, it's fully managed and highly reliable
- Built-in integration with various sources and destinations

# A. Work with data streams by using Azure Stream Analytics

A typical event processing pipeline
Event Producer-> Event ingesion System-> Stream Analytics engine -> Event Consumer

## Event Producer
- sensor, 
- Systems, 
- Application

## Event ingesion System
- Azure Event Hubs, 
- Azure IoT Hub, or 
- Azure Blob storage

## Stream Analytics engine
Compute is run over the incoming streams of data and insights are extracted. 
Azure Stream Analytics exposes the Stream Analytics query language (SAQL)

## Event consumer
- Azure Data Lake, 
- Azure Cosmos DB, 
- Azure SQL Database
- zure Blob storage
- dashboards powered by Power BI.


# B. Transform data by using Azure Stream Analytics
 
Create Azure Stream Analytics jobs to process input data, transform it with a query, and return results.
In Azure Stream Analytics, jobs are the foundation of real-time analytics
Streaming Analytics Workflow 
input (Source) >  transformation query -> output (Sink)

## Step 1: Create an Azure Stream Analytics job
```
All services -> Analytics -> Stream Analytics jobs -> click Add
job name = SimpleTransformer
RG = koolkravi-streamanalytics
LOCATION =  East US
Hosting environment = cloud
Streaming units=1

Select Create to create the new job
```

## Setp 2: Configure the Azure Stream Analytics job input
An Azure Stream Analytics job supports three input types 
- Azure Event Hubs
- Azure IoT Hub	
- Azure Blob storage	

Create the input source -Azure Blob store as the input 
Azure Blob storage has three components(Azure Storage account, container, blob)

### 2.1 : Creating an Azure Blob storage account
```
All services -> Storage -> Storage accounts -> click Add
RG = koolkravi-streamanalytics
LOCATION =  East US
Storage account name = streamsrckoolkravi
Performance =  	Standard  
Account Kind = Storage V2
Replication = RA-GRS  (Read Access - Geo Redundant Storage)

Select Review + create
```
### 2.2 : Connect the input source to the Stream Analytics job

```
All services -> Analytics -> Stream Analytics jobs
Job topology- >  Inputs -> Add stream input - > Blob storage
Input alias  = streaminput
Select Blob storage from your subscriptions
storage account= streamsrckoolkravi
Container -> Create new koolkravi-learn-container
Path pattern= input/
date format - YYYY/MM/DD
others = keep defaults

Save to associate the input
```

## Step 3 Configure the Azure Stream Analytics job output
Stream Analytics jobs support various output sinks 
- Azure Blob storage, 
- SQL Database, and 
- Event Hubs

### 3.1 : Creating a second Blob storage account
```
All services -> Storage -> Storage accounts -> click Add
RG = koolkravi-streamanalytics
LOCATION =  East US
Storage account name = streamsrckoolkravi2
Performance =  	Standard  
Account Kind = Storage V2
Replication = RA-GRS  (Read Access - Geo Redundant Storage)

Select Review + create
```

### 3.2 : Connect an output sink to a Stream Analytics job
```
All services -> Analytics -> Stream Analytics jobs
Job topology- >  Outputs -> Add  - > Blob storage
Output alias  = streamoutput
Select Blob storage from your subscriptions
storage account= streamsrckoolkravi2
Container -> Create new ->koolkravi-learn-container
Path pattern= output/
others = keep defaults

Save to associate the input
```

## Step 4 : Write an Azure Stream Analytics transformation query
An Azure Stream Analytics query transforms an input data stream and produces an output. 
Queries are written in a language like SQL that's a subset of the Transact-SQL (T-SQL) language

- input: census data  JSON file
- Run a transformation query to pull coordinates of each city out of the data, and then 
- write the results to a new file in your Blob storage

### 4.1 :  Create the sample input file

input.json
```
{
    "City" : "Reykjavik",
    "Coordinates" :
    {
        "Latitude": 64,
        "Longitude": 21
    },
    "Census" :
    {
        "Population" : 125000,
        "Municipality" : "Reykjavík",
        "Region" : "Höfuðborgarsvæðið"
    }
}
```
### 4.2 : Upload the input file
```
All services -> Storage -> Storage accounts - > streamsrckoolkravi
Blob service -> Containers -> koolkravi-learn-container
Upload -> select the JSON file  input.json
Expand the Advanced option - >
Upload to folder = input/[YYYY-MM-DD]    input/2020-07-14

leave defaults
save
```
### 4.3 : Set up the output Blob storage container

```
GO to destination blob storage
All services -> Storage -> Storage accounts - > streamsrckoolkravi2
select Storage Explorer (preview) -> 
BlOB CONTAINERS -> Select container  koolkravi-learn-container
New Folder-> folder name =output 

Ok
```

### 4.4. : Write the transformation query
pull the coordinates from the input data and write them to the output

```
All services -> Analytics -> Stream Analytics jobs ->SimpleTransfomer
Job topology- > Query
```
```
SELECT City,
    Coordinates.Latitude,
    Coordinates.Longitude
INTO streamoutput
FROM streaminput
```

```
Save Query
```

### 4.5 Test the query

```
All services -> Analytics -> Stream Analytics jobs ->SimpleTransfomer
Job topology- > Query - > Test.
```

## Step 5 :  Start an Azure Stream Analytics job

run job  to produce an output
```
All services -> Analytics -> Stream Analytics jobs ->SimpleTransfomer
click start -> Now > Start
```
```
status  
Starting state -> Running state ->Completed
```

## Step 6 : View the results from an Azure Stream Analytics job
```
Go to  output storage account
All services -> Storage -> Storage accounts - > streamsrckoolkravi2
Storage Explorer (preview)- > BLOB CONTAINERS ->   koolkravi-learn-container
output -> Download
```

```
out
{
    "city" : "Reykjavik",
    "latitude" : 64,
    "longitude" : 21
}
```

## Setp 6 : Monitor and troubleshoot Azure Stream Analytics jobs
Monitoring is a key part of any mission-critical workload. 
It helps to proactively detect and prevent issues that might otherwise cause application or service downtime
### Monitor Azure Stream Analytics jobs by using several tools
```
All services -> Analytics -> Stream Analytics jobs ->SimpleTransfomer 
```
- Activity logs
- Dashboards (  Monitoring -> select Metrics)
- Alerts (Monitoring ->  select Alert rules > New alert rule)
- Diagnostic logs (Monitoring-> select Diagnostic logs)

## Setp 7: Clean Up
```
delete koolkravi-streamanalytics
```


# References
- https://docs.microsoft.com/en-in/learn/modules/introduction-to-data-streaming/
- https://docs.microsoft.com/en-in/learn/modules/transform-data-with-azure-stream-analytics/

