# Azure Databricks

# Azure Data Lake Storage Gen2, Azure Databricks & Spark

- Create a Databricks cluster
- Ingest unstructured data into a storage account
- Run analytics on your data in Blob storage

## Prerequisites

### Create an Azure Data Lake Storage Gen2 account
```
https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal
```
```
RG=myrg
Storage account = mystrgact
location = east US

for  Azure Data Lake Storage
Advanced - > Hierarchical namespace = 	Enabled
```

### Make sure that your user account has the Storage Blob Data Contributor role assigned to it
```
https://docs.microsoft.com/en-us/azure/storage/common/storage-auth-aad-rbac-portal
```
```
All Services- >storage account -> mystrgact ->overview 
Access control (IAM) ->Role assignments ->Add ->Add role assignment
select Role =Storage Blob Data Contributor role
Select user 

SAVE
```

### Install AzCopy v10. 
```
https://docs.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10?toc=/azure/storage/blobs/toc.json
```
```
azcopy -h
```

### Create a service principal
```
How to: Use the portal to create an Azure AD application and service principal that can access resources.
https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal
```

 Register an application with Azure AD and create a service principal
```
All Services->Identity->Azure Active Directory	
App registrations ->New registration
Name = mydbapp 

Register
```

Assign a role to the application
- Storage Blob Data Contributor role to the service principal
- Make sure to assign the role in the scope of the Data Lake Storage Gen2 storage account

```
All Services - > Storage account - > mystrgact 
Access control (IAM) - > add -> Role assignments
Role = Storage Blob Data Contributor
Adssign access to : Azure AD User, group, or Service Principle
User = mydbapp (Service principle)

SAVE
```

Create a new application secret
```
Azure Active Directory-> App registrations -> Certificates & secrets
Client secrets -> New client secret.
client secret : <<>>
```

Get tenant and app ID values for signing in
```
All Services->Identity->Azure Active Directory
Azure Active Directory -> App registrations -> mydbapp
tenant ID : <<>>
app ID : <<>>

```

## Download the flight data
```
https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time
Select the Prezipped File check box to select all data fields.

On_Time_Reporting_Carrier_On_Time_Performance_1987_present_2020_1.zip
```

## Create an Azure Databricks service
```
 All Services > Analytics > Azure Databricks
 Azure Databricks Service
 Workspace name = myws
 Resource group =  myws
 Location =east US
 Pricing Tier	Select Standard
 
 CREATE
```

## Create a Spark cluster in Azure Databricks
```
 All Services > Analytics > Azure Databricks 
 myws -> Launch Workspace ->  New cluster 
 
 name= my-spark-cluster
 Terminate after 120 minutes of inactivity checkbox
 accept the default values
 
 Create cluster 
```

## Ingest data
Copy source data into the storage account
Use AzCopy to copy data from your .csv file into your Data Lake Storage Gen2 account
```
./azcopy login  --tenant-id=e12c9f8e-ecb3-4cbd-b504-9410ab9116ae

azcopy cp "D:\data.csv" https://mystrgact.dfs.core.windows.net/mydata/folder1/On_Time.csv

./azcopy cp "/d/data.csv" https://mystrgact.dfs.core.windows.net/mydata/folder1/On_Time.csv

```

## Create a container and mount it



???TODO???



## Clean up resources

Delete RG








# References
## Tutorial: Azure Data Lake Storage Gen2, Azure Databricks & Spark
```
https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-use-databricks-spark
```

