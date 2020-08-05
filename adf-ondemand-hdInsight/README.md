# Create on-demand Apache Hadoop clusters in HDInsight using Azure Data Factory
```
create an Apache Hadoop cluster, on demand, in Azure HDInsight using Azure Data Factory.
use data pipelines in Azure Data Factory to run Hive jobs and delete the cluster
how to operationalize a big data job run where cluster creation, job run, and cluster deletion are done on a schedule

Create an Azure storage account
Understand Azure Data Factory activity
Create a data factory using Azure portal
Create linked services
Create a pipeline
Trigger a pipeline
Monitor a pipeline
Verify the output
```
## Prerequisites
- PowerShell Az Module installed
  ```
  https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-4.4.0
  ```
- An Azure Active Directory service principal
  make sure the service principal is a member of the Contributor role of the subscription or the resource group in which the cluster is created
  ```
  SP = adf-hdicluster-sp
  RG = adf-hdicluster-rg   (east us)
  ```
	
## Steps

## Step 1: Create preliminary Azure objects
create various objects that will be used for the HDInsight cluster you create on-demand. The created storage account will 
contain the sample HiveQL script, partitionweblogs.hql, that you use to simulate a sample Apache Hive job that runs on the cluster
Azure PowerShell script to create the storage account and copy over the required files within the storage account.
The Azure PowerShell sample script in this section does the following tasks
The Azure PowerShell sample script in this section does the following tasks
Signs in to Azure.
Creates an Azure resource group.
Creates an Azure Storage account.
Creates a Blob container in the storage account
Copies the sample HiveQL script (partitionweblogs.hql) the Blob container


### 1.1 Create storage account and copy files
/resources/create-storage-account.ps
```
Azure Resource Group Name = adf-hdicluster-rg
Azure Storage Account Name = adfhdiclustersrorageact
```
out:
https://adfhdiclustersrorageact.blob.core.windows.net/
Resource group name: adf-hdicluster-rg
Storage Account Name: adfhdiclustersrorageact
Storage Account Key: qbDwLSIoivkcFWM/NG3mLrvmaAeXNZvd3ii5/AAsT5INAhIdjaJgMfbmU5YS93tvVHFA8mA4sHCbYbXgYcDNYQ==

### 1.2 Verify storage account
```
All services -> storage account- > adfhdiclustersrorageact - > container (adfgetstarted) - > folder (hivescripts) - > sample script (partitionweblogs.hql.)
https://adfhdiclustersrorageact.blob.core.windows.net/adfgetstarted/hivescripts/partitionweblogs.hql
```

## Step 2 : Understand the Azure Data Factory activity
Azure Data Factory orchestrates and automates the movement and transformation of data. Azure Data Factory can create an HDInsight Hadoop cluster 
just-in-time to process an input data slice and delete the cluster when the processing is complete.

In Azure Data Factory, a data factory can have one or more data pipelines. 
A data pipeline has one or more activities. There are two types of activities
- Data Movement Activities. You use data movement activities to move data from a source data store to a destination data store.
- Data Transformation Activities. You use data transformation activities to transform/process data. 
  HDInsight Hive Activity is one of the transformation activities supported by Data Factory. You use the Hive transformation activity in this tutorial.
- we configure the Hive activity to create an on-demand HDInsight Hadoop cluster. When the activity runs to process data, here is what happens:

```
1. An HDInsight Hadoop cluster is automatically created for you just-in-time to process the slice.

2. The input data is processed by running a HiveQL script on the cluster. In this tutorial, the HiveQL script associated with the hive activity does the following actions:

#Uses the existing table (hivesampletable) to create another table HiveSampleOut.
#Populates the HiveSampleOut table with only specific columns from the original hivesampletable.

3. The HDInsight Hadoop cluster is deleted after the processing is complete and the cluster is idle for the configured amount of time (timeToLive setting). If the next data slice is available for processing with in this timeToLive idle time, the same cluster is used to process the slice.
```

## Step 3 : Create a data factory
```
All Services > Analytics > Data Factory.
Name = adf-hdicluster-datafactory
Select Enable GIT Later
CREATE
```

```
Select Author & Monitor to launch the Azure Data Factory authoring and monitoring portal
```

## Step 4 : Create linked services
Author two linked services within data factory.

- An Azure Storage linked service 
	that links an Azure storage account to the data factory. This storage is used by the on-demand HDInsight cluster. 
	It also contains the Hive script that is run on the cluster.
- An on-demand HDInsight linked service. 
	Azure Data Factory automatically creates an HDInsight cluster and runs the Hive script. 
	It then deletes the HDInsight cluster after the cluster is idle for a preconfigured time.


### 4.1. Create an Azure Storage linked service
```
Selecty Author icon
select Manage tab --> new ->New Linked Service ->  select Azure Blob Storage ->continue
Name= HDIStorageLinkedService
storage act name  =adfhdiclustersrorageact ->  Test connection -> Create
```

### 4.2. Create an on-demand HDInsight linked service
```
+ New ->  New Linked Service -> select the Compute tab- > Select Azure HDInsight - >   select Continue.
Name = HDInsightLinkedService
Type = On-demand HDInsight.
Azure Storage linked service = HDIStorageLinkedService
Cluster type	=  hadoop
Time to live= 30 mins
Service principal ID= adf-hdicluster-sp
Service principal key =  authentication key
Cluster name prefix = adfhdi
OS type/Cluster SSH user name = sshuser
Cluster SSH password = 
Cluster user name =admin
Cluster password=
```

## Step 5: Create a pipeline

```
select author - > pipeline
Activities - > HDInsight - > drag the Hive activity to the pipeline designer surface
In the General tab
	name= createclusterrunjob
HDI Cluster tab
	HDInsight Linked Service drop-down = HDInsightLinkedService
Script tab
	Script Linked Serviceselect HDIStorageLinkedService 
	File Path, select Browse Storage = adfgetstarted/hivescripts/partitionweblogs.hql
Advanced > Parameters, select Auto-fill from script
	In the value text box, add the existing folder in the format
	#wasbs://adfgetstarted@<StorageAccount>.blob.core.windows.net/outputfolder/
	wasbs://adfgetstarted@adfhdiclustersrorageact.blob.core.windows.net/outputfolder/
Select Validate - > Publish All to publish the artifacts to Azure Data Factory
```

## Step 6: Trigger a pipeline
```
Add trigger > Trigger Now. -> OK
```

## Step 7: Monitor a pipeline
```
Monitor tab -> Pipeline Runs list  ->Status -> Refresh to refresh the status
View Activity Runs ()
```

## Step 8: Verify the output
```
storage account -> 
#You see an adfgerstarted/outputfolder that contains the output of the Hive script that was run as part of the pipeline
#You see an adfhdidatafactory-<linked-service-name>-<timestamp> container. This container is the default storage location of the HDInsight cluster that was created as part of the pipeline run.
# You see an adfjobs container that has the Azure Data Factory job logs.
```

## Step 8 clean up 
```
Delete Resource groups adf-hdicluster-rg
```






## Reference:
- 1. Create an Azure Active Directory service principal.
```
https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal
```
- 2. What is Azure Data Factory?
```
https://docs.microsoft.com/en-us/azure/data-factory/introduction
https://docs.microsoft.com/en-us/azure/data-factory/quickstart-create-data-factory-portal
```


