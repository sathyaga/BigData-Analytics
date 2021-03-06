# This reference implementation provides an overview of an end-to-end Enterprise BI platform with data movement from on-prem to cloud environment. The underlying architecture has Azure Blob Storage, Azure SQL DW, Azure Analysis Service to support various data management layers and PowerBI for visuakizations.

I used the NY Taxi Cab public dataset with over 100+ million records of data and tools like T-SQL and SSMS 2017. This solution would be helpful to those looking for basic guidance to onboarding their on-prem solution to a cloud (Azure) environment.


The Logical/technical steps I used to design this solution for a demo purpose is below:

# 1.	Create a data warehouse in the Azure portal: 
        --Login to Azure portal and create a blank SQL data warehouse, provision the PaaS
# 2.	Set up a server-level firewall rule in the Azure portal: 
        --Note: SQL Data Warehouse communicates over port 1433 
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

# Create External Tables
CREATE EXTERNAL TABLE [ext].[Date] 
(
    [DateID] int NOT NULL,
    [Date] datetime NULL,
    [DateBKey] char(10) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [DayOfMonth] varchar(2) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [DaySuffix] varchar(4) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [DayName] varchar(9) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [DayOfWeek] char(1) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [DayOfWeekInMonth] varchar(2) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [DayOfWeekInYear] varchar(2) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [DayOfQuarter] varchar(3) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [DayOfYear] varchar(3) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [WeekOfMonth] varchar(1) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [WeekOfQuarter] varchar(2) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [WeekOfYear] varchar(2) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [Month] varchar(2) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [MonthName] varchar(9) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [MonthOfQuarter] varchar(2) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [Quarter] char(1) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [QuarterName] varchar(9) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [Year] char(4) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [YearName] char(7) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [MonthYear] char(10) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [MMYYYY] char(6) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [FirstDayOfMonth] date NULL,
    [LastDayOfMonth] date NULL,
    [FirstDayOfQuarter] date NULL,
    [LastDayOfQuarter] date NULL,
    [FirstDayOfYear] date NULL,
    [LastDayOfYear] date NULL,
    [IsHolidayUSA] bit NULL,
    [IsWeekday] bit NULL,
    [HolidayUSA] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL
)
WITH
(
    LOCATION = 'Date',
    DATA_SOURCE = NYTPublic,
    FILE_FORMAT = uncompressedcsv,
    REJECT_TYPE = value,
    REJECT_VALUE = 0
); 
CREATE EXTERNAL TABLE [ext].[Geography]
(
    [GeographyID] int NOT NULL,
    [ZipCodeBKey] varchar(10) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL,
    [County] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [City] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [State] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [Country] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [ZipCode] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL
)
WITH
(
    LOCATION = 'Geography',
    DATA_SOURCE = NYTPublic,
    FILE_FORMAT = uncompressedcsv,
    REJECT_TYPE = value,
    REJECT_VALUE = 0 
);      
CREATE EXTERNAL TABLE [ext].[HackneyLicense]
(
    [HackneyLicenseID] int NOT NULL,
    [HackneyLicenseBKey] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL,
    [HackneyLicenseCode] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL
)
WITH
(
    LOCATION = 'HackneyLicense',
    DATA_SOURCE = NYTPublic,
    FILE_FORMAT = uncompressedcsv,
    REJECT_TYPE = value,
    REJECT_VALUE = 0
);
CREATE EXTERNAL TABLE [ext].[Medallion]
(
    [MedallionID] int NOT NULL,
    [MedallionBKey] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL,
    [MedallionCode] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL
)
WITH
(
    LOCATION = 'Medallion',
    DATA_SOURCE = NYTPublic,
    FILE_FORMAT = uncompressedcsv,
    REJECT_TYPE = value,
    REJECT_VALUE = 0
)
;  
CREATE EXTERNAL TABLE [ext].[Time]
(
    [TimeID] int NOT NULL,
    [TimeBKey] varchar(8) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL,
    [HourNumber] tinyint NOT NULL,
    [MinuteNumber] tinyint NOT NULL,
    [SecondNumber] tinyint NOT NULL,
    [TimeInSecond] int NOT NULL,
    [HourlyBucket] varchar(15) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL,
    [DayTimeBucketGroupKey] int NOT NULL,
    [DayTimeBucket] varchar(100) COLLATE SQL_Latin1_General_CP1_CI_AS NOT NULL
)
WITH
(
    LOCATION = 'Time',
    DATA_SOURCE = NYTPublic,
    FILE_FORMAT = uncompressedcsv,
    REJECT_TYPE = value,
    REJECT_VALUE = 0
);
CREATE EXTERNAL TABLE [ext].[Trip]
(
    [DateID] int NOT NULL,
    [MedallionID] int NOT NULL,
    [HackneyLicenseID] int NOT NULL,
    [PickupTimeID] int NOT NULL,
    [DropoffTimeID] int NOT NULL,
    [PickupGeographyID] int NULL,
    [DropoffGeographyID] int NULL,
    [PickupLatitude] float NULL,
    [PickupLongitude] float NULL,
    [PickupLatLong] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [DropoffLatitude] float NULL,
    [DropoffLongitude] float NULL,
    [DropoffLatLong] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [PassengerCount] int NULL,
    [TripDurationSeconds] int NULL,
    [TripDistanceMiles] float NULL,
    [PaymentType] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [FareAmount] money NULL,
    [SurchargeAmount] money NULL,
    [TaxAmount] money NULL,
    [TipAmount] money NULL,
    [TollsAmount] money NULL,
    [TotalAmount] money NULL
)
WITH
(
    LOCATION = 'Trip2013',
    DATA_SOURCE = NYTPublic,
    FILE_FORMAT = compressedcsv,
    REJECT_TYPE = value,
    REJECT_VALUE = 0
);
CREATE EXTERNAL TABLE [ext].[Weather]
(
    [DateID] int NOT NULL,
    [GeographyID] int NOT NULL,
    [PrecipitationInches] float NOT NULL,
    [AvgTemperatureFahrenheit] float NOT NULL
)
WITH
(
    LOCATION = 'Weather',
    DATA_SOURCE = NYTPublic,
    FILE_FORMAT = uncompressedcsv,
    REJECT_TYPE = value,
    REJECT_VALUE = 0
)
;

# Load the data into the Data Warehouse
CREATE TABLE [dbo].[Date]
WITH
( 
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
)
AS SELECT * FROM [ext].[Date]
OPTION (LABEL = 'CTAS : Load [dbo].[Date]')
;
CREATE TABLE [dbo].[Geography]
WITH
( 
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
)
AS
SELECT * FROM [ext].[Geography]
OPTION (LABEL = 'CTAS : Load [dbo].[Geography]')
;
CREATE TABLE [dbo].[HackneyLicense]
WITH
( 
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
)
AS SELECT * FROM [ext].[HackneyLicense]
OPTION (LABEL = 'CTAS : Load [dbo].[HackneyLicense]')
;
CREATE TABLE [dbo].[Medallion]
WITH
(
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
)
AS SELECT * FROM [ext].[Medallion]
OPTION (LABEL = 'CTAS : Load [dbo].[Medallion]')
;
CREATE TABLE [dbo].[Time]
WITH
(
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
)
AS SELECT * FROM [ext].[Time]
OPTION (LABEL = 'CTAS : Load [dbo].[Time]')
;
CREATE TABLE [dbo].[Weather]
WITH
( 
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
)
AS SELECT * FROM [ext].[Weather]
OPTION (LABEL = 'CTAS : Load [dbo].[Weather]')
;
CREATE TABLE [dbo].[Trip]
WITH
(
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
)
AS SELECT * FROM [ext].[Trip]
OPTION (LABEL = 'CTAS : Load [dbo].[Trip]')
;

# Create Statistics on newly loaded data
```sql
CREATE STATISTICS [dbo.Date DateID stats] ON dbo.Date (DateID);
CREATE STATISTICS [dbo.Trip DateID stats] ON dbo.Trip (DateID);
```
