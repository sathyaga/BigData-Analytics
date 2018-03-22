# BigData-Analytics - This reference implementation is desinged to provide an overview for building an end-to-end Enterprise BI platform using Azure Blob Storage, Azure SQL DW, Azure Analysis Service and PowerBI. I used a NY Taxi Cab public dataset with over 100 million records of data. This solution would be helpful to those looking for onboarding their on-prem solution to a cloud (Azure) environment.


I used the following logical steps to design this solution for a demo purpose:

# 1.	Create a data warehouse in the Azure portal: Login to Azure portal and create a blank SQL data warehouse (PaaS)
# 2.	Set up a server-level firewall rule in the Azure portal: 
    SQL Data Warehouse communicates over port 1433. If you are trying to connect from within a corporate network, outbound traffic over         port 1433 might not be allowed by your network's firewall. If so, you cannot connect to your Azure SQL Database server unless your IT       department opens port 1433.
# 3.	Connect to the data warehouse with SSMS
# 4.	Create a user designated for loading data:
    --CREATE LOGIN LoaderRC20 WITH PASSWORD = 'a123STRONGpassword!';
    --CREATE USER LoaderRC20 FOR LOGIN LoaderRC20;
# grants the new user CONTROL permissions on the new data warehouse
    --CREATE USER LoaderRC20 FOR LOGIN LoaderRC20;
    --GRANT CONTROL ON DATABASE::[mySampleDataWarehouse] to LoaderRC20;
    --EXEC sp_addrolemember 'staticrc20', 'LoaderRC20';

# 5. Create external tables for data in Azure blob storage
    CREATE MASTER KEY;
    CREATE EXTERNAL DATA SOURCE NYTPublic
    WITH
    (
    TYPE = Hadoop,
    LOCATION = 'wasbs://2013@nytaxiblob.blob.core.windows.net/'
    );

  # T-SQL statement to specify formatting characteristics and options for the external data file. The external file is compressed with Gzip

  CREATE EXTERNAL FILE FORMAT uncompressedcsv
  WITH (
    FORMAT_TYPE = DELIMITEDTEXT,
    FORMAT_OPTIONS ( 
        FIELD_TERMINATOR = ',',
        STRING_DELIMITER = '',
        DATE_FORMAT = '',
        USE_TYPE_DEFAULT = False
    )
);
CREATE EXTERNAL FILE FORMAT compressedcsv
WITH ( 
    FORMAT_TYPE = DELIMITEDTEXT,
    FORMAT_OPTIONS ( FIELD_TERMINATOR = '|',
        STRING_DELIMITER = '',
    DATE_FORMAT = '',
        USE_TYPE_DEFAULT = False
    ),
    DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec'
);

########
