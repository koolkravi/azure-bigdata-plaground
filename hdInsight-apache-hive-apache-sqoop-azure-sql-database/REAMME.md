# Tutorial: Extract, transform, and load data by using Azure HDInsight 
HDInsight-apache-hive-apache-sqoop-azure-sql-database
Extract and upload the data to an HDInsight cluster.
Transform the data by using Apache Hive.
Load the data to Azure SQL Database by using Sqoop

## Prerequisites
Ref: azure-spring-boot-azure-ad/docsauth-flows.xlsx   for azure resources names

### 1. An Azure Data Lake Storage Gen2 storage account that is configured for HDInsight

```
ref: https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2
Create a user-assigned managed identity
#Create a resource - >  user assigned ->User Assigned Managed Identity
or All Services->Identity-> Managed Identities
Create
name : us-managed-identity1
RG = koolkravi-hdirg
Location eastus
#Create a Data Lake Storage Gen2 account
All Services->Storage accounts->ADD
name=koolkdatalakestoragedev
Click Enabled next to Hierarchical namespace under Data Lake Storage Gen2.
Create
#Set up permissions for the managed identity on the Data Lake Storage Gen2 account
Assign the managed identity to the Storage Blob Data Owner role on the storage account.
storage account (koolkdatalakestoragedev) ->Access control (IAM)->Role assignments->+ Add role assignment
role= Storage Blob Data Owner
assign access to = user-assigned managed identity
select = us-managed-identity1
save

# create a cluster
cluster must be in the same Azure region as the storage account
All Services->Analytics ->HDInsight clusters
register Microsoft.HDInsight
name =koolkravihdicluster
Storage tab - > 
	Primary storage type=Azure Data Lake Storage Gen2
	 Primary Storage account =koolkdatalakestoragedev
	 Identity = user-assigned managed identity = us-managed-identity1
Create

# Access files from the cluster
Using the fully qualified name
abfs://<containername>@<accountname>.dfs.core.windows.net/<file.path>/
Using the shortened path format
abfs:///<file.path>/
Using the relative path
/<file.path>/

#Data access examples
ssh connection to the head node of the cluster
REf : https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix 
<clustername>-ssh.azurehdinsight.net				22	Primary headnode
<clustername>-ssh.azurehdinsight.net				23	Secondary headnode
<clustername>-ed-ssh.azurehdinsight.net				22	edge node (ML Services on HDInsight)
<edgenodename>.<clustername>-ssh.azurehdinsight.net	22	edge node (any other cluster type, if an edge node exists)

#A few hdfs commands
Replace CONTAINERNAME and STORAGEACCOUNT with the relevant values
##Create a file on local storage.
touch testFile.txt
##Create directories on cluster storage.
hdfs dfs -mkdir abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/sampledata1/
hdfs dfs -mkdir abfs:///sampledata2/
hdfs dfs -mkdir /sampledata3/
##Copy data from local storage to cluster storage.
hdfs dfs -copyFromLocal testFile.txt  abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/sampledata1/
hdfs dfs -copyFromLocal testFile.txt  abfs:///sampledata2/
hdfs dfs -copyFromLocal testFile.txt  /sampledata3/
##List directory contents on cluster storage.
hdfs dfs -ls abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/sampledata1/
hdfs dfs -ls abfs://sampledata2/
hdfs dfs -ls /sampledata3/
```
### 2. A Linux-based Hadoop cluster on HDInsight
```
ref: https://docs.microsoft.com/en-us/azure/hdinsight/hadoop/apache-hadoop-linux-create-cluster-get-started-portal
Quickstart: Create Apache Hadoop cluster in Azure HDInsight using Azure portal
Note: cluster alredy created in pre requisite 1 name =koolkravihdicluster
#Run Apache Hive queries
Apache Hive is the most popular component used in HDInsight. There are many ways to run Hive jobs in HDInsight. In this quickstart, you use the Ambari Hive view from the portal. For other methods for submitting Hive jobs, see Use Hive in HDInsight.

#To open Ambari, select Cluster Dashboard. You can also browse to https://ClusterName.azurehdinsight.net
e.g. https://koolkravihdicluster.azurehdinsight.net
Open Hive View
SHOW TABLES;
SELECT * FROM hivesampletable;
# After you've completed a Hive job ,you can export the results to Azure SQL Database or SQL Server database, you can also visualize the results using Excel

connect using ssh clinet :  sshuser@koolkravihdicluster-ssh.azurehdinsight.net
```

### 3. Azure SQL Database: 
You use Azure SQL Database as a destination data store. If you don't have a database in SQL Database
Quickstart: Create an Azure SQL Database single database
ref: https://docs.microsoft.com/en-us/azure/azure-sql/database/single-database-create-quickstart?tabs=azure-portal
```
All Services -> Database - > Azure SQL -> Add
QL databases - >  Single database
Database name = koolkDataBase 
Server name = koolksqlserver
Server admin login = azureuser
Password = 
location =eastUS
Compute + storage  - > Configure database
Compute tier= Serverless
Networking tab - > 
	Connectivity method = Public endpoint.
	Firewall rules ->  Add current client IP address = Yes
 Additional settings ->
	Data source - > Use existing data > sample

Review and create

fully qualified server name =  koolksqlserver.database.windows.net

#Query the database
SQL Database page - > select Query editor (preview) 
login = koolksqlserver
paswword = 

SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
FROM SalesLT.ProductCategory pc
JOIN SalesLT.Product p
ON pc.productcategoryid = p.productcategoryid;

```

### 4. Install Azure CLI:
```
ref: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest
```

### 5. A Secure Shell (SSH) client
```
ref: https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix
Connect to HDInsight (Apache Hadoop) using SSH
ssh sshuser@koolkravihdicluster-ssh.azurehdinsight.net
```

## Step 1: Download the flight data
```
https://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time
/resources/547151747_T_ONTIME_REPORTING.zip
```
 
## Step 2: Extract and upload the data

### 2.1 upload the .zip file to the HDInsight cluster head node
```
#scp <file-name>.zip <ssh-user-name>@<cluster-name>-ssh.azurehdinsight.net:<file-name.zip>

scp 547151747_T_ONTIME_REPORTING.zip sshuser@koolkravihdicluster2-ssh.azurehdinsight.net:547151747_T_ONTIME_REPORTING.zip


note: if using public key 
scp -i ~/.ssh/id_rsa <file_name>.zip <user-name>@<cluster-name>-ssh.azurehdinsight.net:.
```

### 2.2 connect to the cluster by using SSH
```
#ssh <ssh-user-name>@<cluster-name>-ssh.azurehdinsight.net
ssh sshuser@koolkravihdicluster2-ssh.azurehdinsight.net
```

### 2.3 unzip the .zip file
```
#unzip <file-name>.zip
unzip 547151747_T_ONTIME_REPORTING.zip
```

### 2.4 create the Data Lake Storage Gen2 container.
Replace the <container-name> placeholder with the name that you want to give your container.
Replace the <storage-account-name> placeholder with the name of your storage account
```
#hadoop fs -D "fs.azure.createRemoteFileSystemDuringInitialization=true" -ls abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/
hadoop fs -D "fs.azure.createRemoteFileSystemDuringInitialization=true" -ls abfs://koolkcontainer@koolkdatalakestoragedev.dfs.core.windows.net/
```

### 2.5 create a directory 

```
#hdfs dfs -mkdir -p abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/data
hdfs dfs -mkdir -p abfs://koolkcontainer@koolkdatalakestoragedev.dfs.core.windows.net/tutorials/flightdelays/data

```

### 2.5 copy the .csv file to the directory
Use quotes around the file name if the file name contains spaces or special characters

```
#hdfs dfs -put "<file-name>.csv" abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/data/
hdfs dfs -put "547151747_T_ONTIME_REPORTING.csv" abfs://koolkcontainer@koolkdatalakestoragedev.dfs.core.windows.net/tutorials/flightdelays/data/
```

## Step 3: Transform the data
use Beeline to run an Apache Hive job
As part of the Apache Hive job, import the data from the .csv file into an Apache Hive table named delays.

### Step 3-1: create and edit a new file named flightdelays.hql
```
nano flightdelays.hql
```

### Step 3-2: HiveQL
Replace the <container-name> and <storage-account-name> placeholders with your container(koolkcontainer) and storage account name (koolkdatalakestoragedev)

```
DROP TABLE delays_raw;
-- Creates an external table over the csv file
CREATE EXTERNAL TABLE delays_raw (
    YEAR string,
    FL_DATE string,
    UNIQUE_CARRIER string,
    CARRIER string,
    FL_NUM string,
    ORIGIN_AIRPORT_ID string,
    ORIGIN string,
    ORIGIN_CITY_NAME string,
    ORIGIN_CITY_NAME_TEMP string,
    ORIGIN_STATE_ABR string,
    DEST_AIRPORT_ID string,
    DEST string,
    DEST_CITY_NAME string,
    DEST_CITY_NAME_TEMP string,
    DEST_STATE_ABR string,
    DEP_DELAY_NEW float,
    ARR_DELAY_NEW float,
    CARRIER_DELAY float,
    WEATHER_DELAY float,
    NAS_DELAY float,
    SECURITY_DELAY float,
    LATE_AIRCRAFT_DELAY float)
-- The following lines describe the format and location of the file
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
STORED AS TEXTFILE
LOCATION 'abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/data';

-- Drop the delays table if it exists
DROP TABLE delays;
-- Create the delays table and populate it with data
-- pulled in from the CSV file (via the external table defined previously)
CREATE TABLE delays
LOCATION 'abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/processed'
AS
SELECT YEAR AS year,
    FL_DATE AS flight_date,
    substring(UNIQUE_CARRIER, 2, length(UNIQUE_CARRIER) -1) AS unique_carrier,
    substring(CARRIER, 2, length(CARRIER) -1) AS carrier,
    substring(FL_NUM, 2, length(FL_NUM) -1) AS flight_num,
    ORIGIN_AIRPORT_ID AS origin_airport_id,
    substring(ORIGIN, 2, length(ORIGIN) -1) AS origin_airport_code,
    substring(ORIGIN_CITY_NAME, 2) AS origin_city_name,
    substring(ORIGIN_STATE_ABR, 2, length(ORIGIN_STATE_ABR) -1)  AS origin_state_abr,
    DEST_AIRPORT_ID AS dest_airport_id,
    substring(DEST, 2, length(DEST) -1) AS dest_airport_code,
    substring(DEST_CITY_NAME,2) AS dest_city_name,
    substring(DEST_STATE_ABR, 2, length(DEST_STATE_ABR) -1) AS dest_state_abr,
    DEP_DELAY_NEW AS dep_delay_new,
    ARR_DELAY_NEW AS arr_delay_new,
    CARRIER_DELAY AS carrier_delay,
    WEATHER_DELAY AS weather_delay,
    NAS_DELAY AS nas_delay,
    SECURITY_DELAY AS security_delay,
    LATE_AIRCRAFT_DELAY AS late_aircraft_delay
FROM delays_raw;
```

### Step 3-3: start Hive and run the flightdelays.hql file
```
beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -f flightdelays.hql
```
### Step 3-4: After the flightdelays.hql script finishes running, use the following command to open an interactive Beeline session
```
beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http'
```

### Step 3-5: use the following query to retrieve data from the imported flight delay data
This query retrieves a list of cities that experienced weather delays, along with the average delay time, and saves it to abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/output. Later, Sqoop reads the data from this location and exports it to Azure SQL Database.
```
INSERT OVERWRITE DIRECTORY '/tutorials/flightdelays/output'
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
SELECT regexp_replace(origin_city_name, '''', ''),
    avg(weather_delay)
FROM delays
WHERE weather_delay IS NOT NULL
GROUP BY origin_city_name;
```

## Step 4: Create a SQL database table
SQL Databases 
There are many ways to connect to SQL Database and create a table. The following steps use FreeTDS from the HDInsight cluster
## Step 4.1: To install FreeTDS, use the following command from an SSH connection to the cluster
```
sudo apt-get --assume-yes install freetds-dev freetds-bin
```
## Step 4.2: use the following command to connect to SQL Database.
Replace the <server-name> placeholder with the logical SQL server name.
Replace the <admin-login> placeholder with the admin login for SQL Database.
Replace the <database-name> placeholder with the database name
```
#TDSVER=8.0 tsql -H '<server-name>.database.windows.net' -U '<admin-login>' -p 1433 -D '<database-name>'

TDSVER=8.0 tsql -H 'koolksqlserver.database.windows.net' -U 'azureuser' -p 1433 -D 'koolkDataBase'
```

## Step 4.3: enter the following statements
The query creates a table named delays, which has a clustered index.
```
CREATE TABLE [dbo].[delays](
[OriginCityName] [nvarchar](50) NOT NULL,
[WeatherDelay] float,
CONSTRAINT [PK_delays] PRIMARY KEY CLUSTERED
([OriginCityName] ASC))
GO
```
## Step 4.4: Use the following query to verify that the table is created
```
SELECT * FROM information_schema.tables
GO
```
Enter exit at the 1> prompt to exit the tsql utility.


## Step 5: Export and load the data 
Load the data to Azure SQL Database by using Sqoop
In the previous sections, you copied the transformed data at the location abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/output. 

In this section, you use Sqoop to export the data from abfs://<container-name>@<storage-account-name>.dfs.core.windows.net/tutorials/flightdelays/output to the table you created in the Azure SQL Database.

### Step 5.1: Use the following command to verify that Sqoop can see your SQL database:
The command returns a list of databases, including the database in which you created the delays table.
```
#sqoop list-databases --connect jdbc:sqlserver://<SERVER_NAME>.database.windows.net:1433 --username <ADMIN_LOGIN> --password <ADMIN_PASSWORD>
sqoop list-databases --connect jdbc:sqlserver://koolksqlserver.database.windows.net:1433 --username azureuser --password Gd33$5512FG

```
### Step 5.2: Use the following command to export data from the hivesampletable table to the delays table
Sqoop connects to the database that contains the delays table, and exports data from the /tutorials/flightdelays/output directory to the delays table.

```
sqoop export --connect 'jdbc:sqlserver://<SERVER_NAME>.database.windows.net:1433;database=<DATABASE_NAME>' --username <ADMIN_LOGIN> --password <ADMIN_PASSWORD> --table 'delays' --export-dir 'abfs://<container-name>@.dfs.core.windows.net/tutorials/flightdelays/output' --fields-terminated-by '\t' -m 1
```

### Step 5.3: use the tsql utility to connect to the database:
```
TDSVER=8.0 tsql -H <SERVER_NAME>.database.windows.net -U <ADMIN_LOGIN> -P <ADMIN_PASSWORD> -p 1433 -D <DATABASE_NAME>
```
### Step 5.4: Use the following statements to verify that the data was exported to the delays table
You should see a listing of data in the table. The table includes the city name and the average flight delay time for that city.
```
SELECT * FROM delays
GO
```

Enter exit to exit the tsql utility.
 
## Step 6 : Clean up resources




# Ref:
- 1. Tutorial: Extract, transform, and load data by using Azure HDInsight 
```
https://docs.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-tutorial-extract-transform-load-hive
```
- 2.Managed identities in Azure HDInsight
```
https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-managed-identities
A managed identity is an identity registered in Azure Active Directory (Azure AD) whose credentials are managed by Azure. With managed identities, you don't need to register service principals in Azure AD. Or maintain credentials such as certificates
There are two types of managed identities: user-assigned and system-assigned. Azure HDInsight supports only user-assigned managed identities. HDInsight doesn't support system-assigned managed identities
```
- 3. Azure HDInsight virtual network architecture
```
https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-virtual-network-architecture
Resource types in Azure HDInsight clusters
Azure HDInsight clusters have different types of virtual machines, or nodes. Each node type plays a role in the operation of the system
Head node 2
ZooKeeper node 3
Worker nod

```
- 4. Use SSH tunneling to access Apache Ambari web UI, JobHistory, NameNode, Apache Oozie, and other UIs
```
https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-linux-ambari-ssh-tunnel
```

- 5. What is Apache Hive and HiveQL on Azure HDInsight? How to use Hive?
```
https://docs.microsoft.com/en-us/azure/hdinsight/hadoop/hdinsight-use-hive
```

- 6. Use Apache Sqoop to import and export data between Apache Hadoop on HDInsight and Azure SQL Database
  export the results to Azure SQL Database or SQL Server database
```
https://docs.microsoft.com/en-us/azure/hdinsight/hadoop/apache-hadoop-use-sqoop-mac-linux
```
- 7. Connect Excel to Apache Hadoop by using Power Query
  visualize the results using Excel.
```
https://docs.microsoft.com/en-us/azure/hdinsight/hadoop/apache-hadoop-connect-excel-power-query
```

- 8. Create a server-level firewall rule using the Azure portal
```
https://docs.microsoft.com/en-us/azure/azure-sql/database/firewall-create-server-level-portal-quickstart
```

