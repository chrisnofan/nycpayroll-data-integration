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
