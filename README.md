# nycpayroll-data-integration

# NYC PAYROLL DATA INTEGRATION

## Project Overview
The City of New York is embarking on a project to integrate payroll data across all its agencies. The City of New York would like to develop a Data Analytics platform to accomplish two primary objectives:
- Financial Resource Allocation Analysis: Analyze how the City's financial resources are allocated and how much of the City's budget is being devoted to overtime.
- Transparency and Public Accessibility: Make the data available to the interested public to show how the City’s budget is being spent on salary and overtime pay for all municipal employees.

## Brief
You have been hired as a Data Engineer to create high-quality data pipelines that are dynamic, automated, scalable, and monitored for efficient operation. The project team also includes the city’s quality assurance experts who will test the pipelines to find any errors and improve overall data quality.
The source data resides in a remote folder and needs to be processed in a NYC data warehouse. The source datasets consist of CSV files with Employee master data and monthly payroll data entered by various City Agencies.

## Tools used for this project
- Azure Storage account
- Azure SQL Server
- Azure SQL Database
- Azure Data Factory

## Data Warehouse Design
Schema: Star Schema (optimized for querying and reporting) was used to desing the datawarehouse.

![dimensional model](https://github.com/user-attachments/assets/1d9fdbb0-dbf4-4135-bab7-d5fff44cbc2a)

Figure 1. Data Warehouse Model


At a high level, below is a schematic of the pipeline archecture.


![Pipeline Architecture](https://github.com/user-attachments/assets/c7c3f34b-e559-44e2-ada8-0889ccc614cc)
Figure 2. Pipleline Architecture

## Data Engineering Steps.
### Step 1 – Creating the Data Infrastructure
1. Create a Resource group which will be used for this project. The Resource group is named “nycpayrollRG”. All other resources will be created in this resource group
2. Create an Azure Data Lake Storage Gen2 (nycpayrollsa) and associated storage container resource named “nycpayroll-source-files” and upload the payroll data into it.
3. Create a sql server – nycpayroll and Azure sql database nycpayrollDB. This database used for staging and serving aggregate data for analysis.
4. Create Azure Data Factory Resource named “nycpayrollAdF. 

![Azure Reources](https://github.com/user-attachments/assets/4b76536c-74a9-4070-9897-d25455289beb)

Figure 3. Azure Resources

### Step 2: Create Linked Services in Azure Data Factory
1. Create a Linked Service (ADLS_LinkedStorage) to azure blob storage which contains all the data files
2. Create a Linked Service (AzureSqlDB_LinkedService) to SQL Database – this will be the sink for the data files.

![Screenshot 2024-08-31 211816](https://github.com/user-attachments/assets/0dbe855e-bfef-4981-a72a-6d6eea23393f)
Figure 4. Linked Services

### Step 3: Create Datasets in Azure Data Factory
1. Create Dataset to the container (nycpayroll-source-files) with file type as JSON to return the metadata of the files in the container.

![Screenshot 2024-08-31 231956](https://github.com/user-attachments/assets/89db1cf7-2247-4325-822f-929b28d85b5f)
Figure 5. Datasets

2. Create the datasets  (nycpayroll_source_DS) for the nycpayroll-source-files container which houses Payroll file on Azure Data Lake Gen2
- Select DelimitedText
-	Set the path to the nycpayroll-source-files in the Data Lake and add parameter filename which will enable dynamic access to the files name in the container

![Screenshot 2024-08-31 232411](https://github.com/user-attachments/assets/3d4b4a24-4a14-4633-a0cf-172b5d17d725)

4. Create Dataset (nycpayroll_staging_DS) which will house the data in sql database and add parameter with value table name. This is to allow for dynamic creation of table names in the staging (STG) schema in the Azure Sql Database

![Screenshot 2024-08-31 232912](https://github.com/user-attachments/assets/06f893e2-11bc-44b5-b599-90127df51348)

5. Create the datasets for transaction data table of both 2020 and 2021 data in SQL DB respectively


### Step 4: Create Data Flows in Azure Data Factory
1. Create new data flow named Aggregate_DF
-	Add first Source  nycpayroll 2020 table from SQL Database
-	Add second Source nycpayroll2021 table from SQL Database
2. Create a new Union activity in the data flow to merge both tables.
3. Add a Filter activity after Union to filter on the fiscal year. (note: The data contains years which neither 2020 nor 2021)
-	filter on  FiscalYear >= 2020
4. Add aggregate Activity and derive new aggregate columns: AggRegularGrossPaid, AggTotalOTPaid, AggTotalOtherPay and TotalPaid 
In Expression Builder, enter:
sum(RegularGrossPaid), sum(TotalOTPaid), sum(TotalOtherPay) and 
RegularGrossPaid + TotalOTPaid + TotalOtherPay for the new columns respectively as show below

![Screenshot 2024-09-01 001905](https://github.com/user-attachments/assets/1eda31ba-90c9-468a-907c-6baff1dd5261)

5. Under Group By, Select AgencyName and Fiscal Year
6. Add a Sink activity to the Data Flow by selecting the dataset (AsureSqlTable2) to load the aggregated data into the Sql DataBase in the production schema PROD with table name nycpayrollaggregate.

![Screenshot 2024-09-01 004340](https://github.com/user-attachments/assets/b2a2bb11-058d-4dcc-809f-030e160a1e1b)

### Step 5: Create Pipeline in Azure Data Factory
1. Create a pipeline named nycparoll_pipeline
1. Add a Get Metadata activity to dynamically get the filenames of data stored in nycpayroll-source-files container. This will help with dynamic creation of table names when copying data to Azure sql database. Upon completing this activity, a JSON object is returned which contains the metadata of childItems the container as shown below

![Screenshot 2024-09-01 012415](https://github.com/user-attachments/assets/c25d1953-ef23-4040-bd69-b195e03ad857)

2. Add ForEcah activity to the pipeline. In ForEach Activity, A copy activity is added to dynamically loop through the source-files and copy all the content in azure storage account to Azure sql database. 
3. Add aggregate_DF data flow to the pipeline. The structure of the entire pipeline is depicted in the picture below;

![Screenshot 2024-09-01 013253](https://github.com/user-attachments/assets/67f05253-2864-475a-acd4-91a93c009bcf)


### Step 6: Trigger and Monitor Pipeline
- Select Add trigger option from pipeline view in the toolbar
-	The schedule for the trigger can be set in the trigger dialog box
-	Choose trigger now to initiate pipeline run
- Monitor the pipeline run.

![Screenshot 2024-09-01 021022](https://github.com/user-attachments/assets/400e4f9f-2ab2-4707-9702-a5daf31ff6da)

### Step 7: Verify pipeline run and check aggreagate tables

![Screenshot 2024-09-01 025220](https://github.com/user-attachments/assets/4087cc2b-a505-469c-b1b8-ed74dc941514)

- check aggregate tables

![Screenshot 2024-09-01 030218](https://github.com/user-attachments/assets/d8a66c2b-a7fe-4ead-9048-892dc7928c17)


### Step 7: Connect Project to Github
This step connects Azure Data Factory is connected to Github
- Login to your Github account and create a new Repo in Github
- Connect Azure Data Factory to Github
- Select your Github repository in Azure Data Factory
- Publish all objects to the repository in Azure Data Factory

![Screenshot 2024-09-01 020219](https://github.com/user-attachments/assets/c7784c4e-5665-4d46-9f10-f7b8ab9895e8)
