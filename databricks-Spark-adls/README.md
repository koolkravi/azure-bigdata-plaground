# Access Data Lake Storage data with Databricks using Spark

## Prerequisites
### 1. Create an Azure Data Lake Storage Gen2 account
```
Ref: https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal

All services- > storage->Storage Accounts- Add
RG  = koolkravi-bdrg
Storage account name  = koolkravistorageact
Location = eastus
Account type = StorageV2(general-purpose v2)
Advance tab->Azure Data Lake Storage-> Hierarchical namespace enable
Create
container-name = flightdata
```
### 2. Assign  user account "Storage Blob Data Contributor role" 
```
Ref : https://docs.microsoft.com/en-us/azure/storage/common/storage-auth-aad-rbac-portal
All services- > storage->Storage Accounts - > (koolkravistorageact) -> 
Access control (IAM) - > Role assignment - > Add  ->  Add role assignment 
Role =  Storage Blob Data Contributor role
Assign access to =  select Azure AD user, group, or service principal.
security principal = AD User
Save
```
###  3. Install AzCopy v10.
```
Ref : https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10?toc=/azure/storage/blobs/toc.json
AzCopy is a command-line utility that you can use to copy blobs or files to or from a storage account
Download - > Windows 64-bit (zip) (azcopy_windows_amd64_10.5.1.zip)
azcopy -h
azcopy list -h
```
### 4. Create a service principal
```
Ref : https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal
```


## Step 1: Download the flight data
```
Ref: https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time
This tutorial uses flight data from the Bureau of Transportation Statistics to demonstrate how to perform an ETL operation

/resources/On_Time_Reporting_Carrier_On_Time_Performance_1987_present_2020_1.zip
```

## Step 2: Create an Azure Databricks service
```
All Services -> Analytics > Azure Databricks.
Workspace name = koolkravi_dbws
Location = eastus
Pricing Tier = Standard (Apache Spark, Secure with Azure AD)
```

## Step 3: Create a Spark cluster in Azure Databricks
```
Azure Databricks Service -> click (koolkravi_dbws) -> Launch Workspace - > new  Cluster
Cluster Name = koolkravi-dbcluster 
create cluster 
After the cluster is running, you can attach notebooks to the cluster and run Spark jobs
```

## Step 4: Ingest data

### Step 4.1. Copy source data into the storage account
```
./azcopy.exe login --tenant-id e12c9f8e-ecb3-4cbd-b504-9410ab9116ae
./azcopy cp '/d/tmp/flight_on_time.csv' 'https://koolkravistorageact.blob.core.windows.net/flightdata/folder1/On_Time.csv'
```
 
### Step 4.2.  Create a container and mount it
``` 
Azure Databricks service - >  (koolkravi_dbws) -> Launch Workspace -> select Create > Notebook
name for the notebook =
language =Python 
Spark cluster
Create
```
Copy and paste the following code block into the first cell
Replace the appId, clientSecret, tenant, and storage-account-name placeholder values
Replace the container-name placeholder value
```
configs = {"fs.azure.account.auth.type": "OAuth",
       "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
       "fs.azure.account.oauth2.client.id": "<appId>",
       "fs.azure.account.oauth2.client.secret": "<clientSecret>",
       "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/<tenant>/oauth2/token",
       "fs.azure.createRemoteFileSystemDuringInitialization": "true"}

dbutils.fs.mount(
source = "abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/folder1",
mount_point = "/mnt/flightdata",
extra_configs = configs)
```
SHIFT + ENTER keys to run the code in this block

### Step 4.3. Use Databricks Notebook to convert CSV to Parquet
add a new cell, and paste the following code into that cell
```
# Use the previously established DBFS mount point to read the data.
# create a data frame to read data.

flightDF = spark.read.format('csv').options(
    header='true', inferschema='true').load("/mnt/flightdata/*.csv")

# read the airline csv file and write the output to parquet format for easy query.
flightDF.write.mode("append").parquet("/mnt/flightdata/parquet/flights")
print("Done")
```

## Step 5: Explore data
code samples to explore the hierarchical nature of HDFS using data stored in a storage account with Data Lake Storage Gen2 enabled

In a new cell, paste the following code to get a list of CSV files uploaded via AzCopy
```
import os.path
import IPython
from pyspark.sql import SQLContext
display(dbutils.fs.ls("/mnt/flightdata"))
```
To create a new file and list files in the parquet/flights folder, run this script:
```
dbutils.fs.put("/mnt/flightdata/1.txt", "Hello, World!", True)
dbutils.fs.ls("/mnt/flightdata/parquet/flights")
```
 
## Step 6: Query the data
 
Enter each of the following code blocks into Cmd 1 and press Cmd + Enter to run the Python script.
To create data frames for your data sources, run the following script:
Replace the <csv-folder-path> placeholder value with the path to the .csv file.

```
# Copy this into a Cmd cell in your notebook.
acDF = spark.read.format('csv').options(
    header='true', inferschema='true').load("/mnt/flightdata/On_Time.csv")
acDF.write.parquet('/mnt/flightdata/parquet/airlinecodes')

# read the existing parquet file for the flights database that was created earlier
flightDF = spark.read.format('parquet').options(
    header='true', inferschema='true').load("/mnt/flightdata/parquet/flights")

# print the schema of the dataframes
acDF.printSchema()
flightDF.printSchema()

# print the flight database size
print("Number of flights in the database: ", flightDF.count())

# show the first 20 rows (20 is the default)
# to show the first n rows, run: df.show(n)
acDF.show(100, False)
flightDF.show(20, False)

# Display to run visualizations
# preferably run this in a separate cmd cell
display(flightDF)
```

Enter this script to run some basic analysis queries against the data.
```
# Run each of these queries, preferably in a separate cmd cell for separate analysis
# create a temporary sql view for querying flight information
FlightTable = spark.read.parquet('/mnt/flightdata/parquet/flights')
FlightTable.createOrReplaceTempView('FlightTable')

# create a temporary sql view for querying airline code information
AirlineCodes = spark.read.parquet('/mnt/flightdata/parquet/airlinecodes')
AirlineCodes.createOrReplaceTempView('AirlineCodes')

# using spark sql, query the parquet file to return total flights in January and February 2016
out1 = spark.sql("SELECT * FROM FlightTable WHERE Month=1 and Year= 2016")
NumJan2016Flights = out1.count()
out2 = spark.sql("SELECT * FROM FlightTable WHERE Month=2 and Year= 2016")
NumFeb2016Flights = out2.count()
print("Jan 2016: ", NumJan2016Flights, " Feb 2016: ", NumFeb2016Flights)
Total = NumJan2016Flights+NumFeb2016Flights
print("Total flights combined: ", Total)

# List out all the airports in Texas
out = spark.sql(
    "SELECT distinct(OriginCityName) FROM FlightTable where OriginStateName = 'Texas'")
print('Airports in Texas: ', out.show(100))

# find all airlines that fly from Texas
out1 = spark.sql(
    "SELECT distinct(Reporting_Airline) FROM FlightTable WHERE OriginStateName='Texas'")
print('Airlines that fly to/from Texas: ', out1.show(100, False))
```

## Clean up resources
```
delete the resource group
```
 
 
 
 
 
# Issues
1.databrick cluster creation issue
```
Cloud Provider Launch Failure: A cloud provider error was encountered while setting up the cluster. See the Databricks guide for more information.
Azure error code: OperationNotAllowed
Azure error message: Operation could not be completed as it results in exceeding approved Total Regional Cores quota. 
Additional details - 
Deployment Model: Resource Manager, Location: eastus, 
Current Limit: 4, Current Usage: 4, 
Additional Required: 4, (Minimum) New Limit Required: 8. 
Submit a request for Quota increase at https://aka.ms/ProdportalCRP/?#create/Microsoft.Support/Parameters/%7B%22subId%22:%22c05ab958-1b38-44ac-81a1-2b8825fe1aab%22,%22pesId%22:%2206bfd9d3-516b-d5c6-5802-169c800dec89%22,%22supportTopicId%22:%22e12e3d1d-7fa0-af33-c6d0-3c50df9658a3%22%7D by specifying parameters listed in the ‘Details’ section for deployment to succeed. 
Please read more about quota limits at https://docs.microsoft.com/en-us/azure/azure-supportability/regional-quota-requests.

Ans: REF: https://docs.microsoft.com/en-us/answers/questions/1544/azure-databricks.html
atleast 8 core is needed but we have only 4 core allowed in free trail subscription 
```
 

# Reference

## 1. Tutorial: Azure Data Lake Storage Gen2, Azure Databricks & Spark
```
- https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-use-databricks-spark
```
## .  Azcopy commands
```
#mk dir
azcopy make 'https://<storage-account-name>.<blob or dfs>.core.windows.net/<container-name>'
example
./azcopy make 'https://koolkravistorageact.dfs.core.windows.net/mycontainer1'

#download
azcopy copy 'https://mystorageaccount.dfs.core.windows.net/mycontainer/myTextFile.txt' 'C:\myDirectory\myTextFile.txt'
```
