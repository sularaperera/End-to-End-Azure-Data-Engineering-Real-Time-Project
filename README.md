## Project Architecture

![enter image description here](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/Architecture.jpg)

1.  **Data Ingestion (ADF Pipeline)**:
    
    -   Set up Azure Data Factory (ADF) to create pipelines for ingesting data from the on-premises SQL Server Database.
    -   Configure the ADF pipeline to push data to the Bronze container in ADLS Gen2.
    
2.  **Data Transformation (Databricks)**:
    
    -   Utilize Azure Databricks for data transformation tasks.
    -   Create Databricks notebooks or Spark jobs to clean and transform data from the Bronze container.
    -   Implement data quality checks and error handling mechanisms during the transformation process.
    -   Load the cleaned data into the Silver container in ADLS Gen2.
3.  **Further Cleaning and Processing (Databricks)**:
    
    -   Implement additional cleaning and processing steps in Databricks to refine the data further.
    -   Apply transformations such as deduplication, normalization, or feature engineering as required.
    -   Load the processed data into the Gold container in ADLS Gen2.
4.  **Data Warehousing (Azure Synapse Analytics)**:
    
    -   Utilize Azure Synapse Analytics to create a dedicated SQL database for analytics purposes.
    -   Establish a connection between Azure Synapse Analytics and ADLS Gen2 to access data stored in the Gold container.
    -   Design and implement schemas within the SQL database to accommodate the structured data from the Gold container.
5.  **Reporting and Visualization (Power BI)**:
    
    -   Connect Power BI to Azure Synapse Analytics SQL database to access the clean data for reporting.
    -   Develop interactive dashboards and reports in Power BI to visualize key metrics and insights.
    -   Implement data refresh schedules in Power BI to ensure that reports reflect the most recent data from Azure Synapse Analytics.

By planning your project architecture in detail, you'll be able to effectively implement each component and ensure a smooth flow of data from ingestion to visualization. Let me know if you need further assistance with any specific aspect of this architecture!


## Step by step approach taken to complete the project

Now I have these resources set up, I can start integrating them and building out data engineering pipelines and workflows.

Here's a brief overview of the resources I've created:

1.  **Resource Group**: This is like a container for holding related Azure resources. It helps you manage and organize your resources effectively.
    
2.  **Azure Data Factory**: A cloud-based data integration service that allows you to create, schedule, and manage data pipelines.
    
3.  **Azure Databricks**: An Apache Spark-based analytics platform optimized for Azure. It provides a collaborative environment for big data analytics and machine learning.
    
4.  **Key Vault**: A secure storage location for managing application secrets, keys, and certificates.
    
5.  **Storage Account (ADLS Gen2)**: Azure Data Lake Storage Gen2 provides scalable, secure storage for big data analytics. It's optimized for analytics workloads.
    
6.  **Synapse Workspace**: Azure Synapse Analytics is a cloud-based analytics service that provides both data integration and big data analytics capabilities. The Synapse Workspace is where you'll develop, orchestrate, and monitor your analytics solutions.


Connecting MSSQL Server to Azure involves setting up the necessary authentication and permissions, as well as securely storing credentials in Azure Key Vault. Here's how you can accomplish this:

1.  **Restore Database "AdventureWorksLT2017"**:
    
    -   Connect to your MSSQL Server instance using SQL Server Management Studio (SSMS) or another SQL client.
    -   Restore the "AdventureWorksLT2017" database from a backup file or detach/attach method as per your requirement.
2.  **Create Login and User**:

    USE AdventureWorksLT2017
    
    CREATE LOGIN sulara WITH PASSWORD = 'lksd090923jskdjkj@jsdlk'
    CREATE USER sulara FOR LOGIN sulara

    
3.  **Grant Permissions**:

    USE AdventureWorksLT2017
    
    ALTER ROLE db_datareader ADD MEMBER sulara

![enter image description here](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/mssql_datareader_permission.png)

By assigning the DataReader role, you're granting read-only access to the database, which allows users to view data but not make any modifications. This ensures security and control over your database while allowing necessary access for data analysis, reporting, or other purposes through Azure Data Factory or other tools.
    
5.  **Save Credentials in Azure Key Vault**:
    
    -   Navigate to the Azure Key Vault service in the Azure portal.
    -   Create a new Key Vault or select an existing one.
    -   Add a new secret and input the username and password for the MSSQL Server login "sulara".

With these steps, you've connected your MSSQL Server database to Azure and securely stored the login credentials in Azure Key Vault. This ensures that your credentials are safely managed and accessible only to authorized applications or services. Let me know if you need further clarification on any of these steps!


### Intergration Runtime
A self-hosted integration runtime (SHIR) in Azure Data Factory is a runtime environment installed on a local machine or a virtual machine within your network. It enables Azure Data Factory to connect to on-premises data sources securely. By installing the SHIR and registering it with your Azure Data Factory, you establish a bridge between your on-premises resources, such as a SQL Server database, and your cloud-based data processing pipelines. This setup allows you to efficiently move data between on-premises and cloud environments, facilitating tasks like pulling data from an on-premises SQL Server database and storing it in Azure Data Lake Storage Gen2 for further processing or analysis.

[End-to-End-Azure-Data-Engineering-Real-Time-Project/Images/shir.png at main · sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project (github.com)](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/shir.png)
