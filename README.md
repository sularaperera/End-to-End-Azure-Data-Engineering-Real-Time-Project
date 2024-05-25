<img src="https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/Banner_new.jpeg" width=600 class="center"></img>


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

# Environment Setup

### Step by step approach taken to complete the project

Now I have these resources set up, I can start integrating them and building out data engineering pipelines and workflows.

Here's a brief overview of the resources I've created:

1.  **Resource Group**: This is like a container for holding related Azure resources. It helps you manage and organize your resources effectively.
    
2.  **Azure Data Factory**: A cloud-based data integration service that allows you to create, schedule, and manage data pipelines.
    
3.  **Azure Databricks**: An Apache Spark-based analytics platform optimized for Azure. It provides a collaborative environment for big data analytics and machine learning.
    
4.  **Key Vault**: A secure storage location for managing application secrets, keys, and certificates.
    
5.  **Storage Account (ADLS Gen2)**: Azure Data Lake Storage Gen2 provides scalable, secure storage for big data analytics. It's optimized for analytics workloads.
    
6.  **Synapse Workspace**: Azure Synapse Analytics is a cloud-based analytics service that provides both data integration and big data analytics capabilities. The Synapse Workspace is where you'll develop, orchestrate, and monitor your analytics solutions.

![enter image description here](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/resources_list.png)


## Microsoft SQL Server - Setup

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
    
    Or, you can use SSMS GUI to create a datareader role and give permission to access the database
![enter image description here](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/mssql_datareader_permission.png)

By assigning the DataReader role, you're granting read-only access to the database, which allows users to view data but not make any modifications. This ensures security and control over your database while allowing necessary access for data analysis, reporting, or other purposes through Azure Data Factory or other tools.
    
5.  **Save Credentials in Azure Key Vault**:
    
    -   Navigate to the Azure Key Vault service in the Azure portal.
    -   Create a new Key Vault or select an existing one.
    -   Add a new secret and input the username and password for the MSSQL Server login "sulara".

With these steps, you've connected your MSSQL Server database to Azure and securely stored the login credentials in Azure Key Vault. This ensures that your credentials are safely managed and accessible only to authorized applications or services. Let me know if you need further clarification on any of these steps!

## Making the connection between Onprem SQL Server and Azure Data Factory

### Intergration Runtime
A self-hosted integration runtime (SHIR) in Azure Data Factory is a runtime environment installed on a local machine or a virtual machine within your network. It enables Azure Data Factory to connect to on-premises data sources securely. By installing the SHIR and registering it with your Azure Data Factory, you establish a bridge between your on-premises resources, such as a SQL Server database, and your cloud-based data processing pipelines. This setup allows you to efficiently move data between on-premises and cloud environments, facilitating tasks like pulling data from an on-premises SQL Server database and storing it in Azure Data Lake Storage Gen2 for further processing or analysis.

![enter image description here](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/shir.png)

![enter image description here](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/2024-05-17%2010_21_22-Microsoft%20Integration%20Runtime%20Configuration%20Manager.png)


# Data Ingestion

## Copy all tables from MSSQL to ADLS using ADF

Creating an Azure Data Factory (ADF) pipeline to copy all tables from an on-premises SQL Server to Azure Data Lake Storage Gen2 (ADLS Gen2) involves a few key steps.

1.  **Set Up Linked Services**:
    
    -   **SQL Server Linked Service**: Configure a linked service for your on-premises SQL Server using the self-hosted integration runtime (SHIR).
    -   **ADLS Gen2 Linked Service**: Set up a linked service for your ADLS Gen2 account.
2.  **Create Datasets**:
    
    -   **Source Dataset**: Create a dataset for the SQL Server tables.
    -   **Sink Dataset**: Create a dataset for the ADLS Gen2 container where the data will be copied.
3.  **Create a New Pipeline**:
    
    -   In your ADF, create a new pipeline.
4.  **Add Lookup Activity**:
    
    -   Use a Lookup activity to retrieve the list of tables from your SQL Server database.
    -   Configure the Lookup activity with a query like `SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE'`.
5.  **Add ForEach Activity**:
    
    -   Add a ForEach activity to iterate over the list of tables returned by the Lookup activity.
    -   In the ForEach activity, set the items to `@activity('LookupActivityName').output.value`.
6.  **Configure Copy Activity Inside ForEach**:
    
    -   Inside the ForEach activity, add a Copy activity.
    -   Set the source of the Copy activity to the SQL Server dataset and parameterize the table name.
    -   Set the sink of the Copy activity to the ADLS Gen2 dataset, specifying the destination folder and file name (which can be parameterized to use the table name).
7.  **Parameterize the Datasets**:
    
    -   **Source Dataset**: Parameterize the table name in the SQL Server dataset.
    -   **Sink Dataset**: Parameterize the folder path and file name in the ADLS Gen2 dataset.
8.  **Run the Pipeline**:
    
    -   Validate the pipeline configuration and run it.
    -   Monitor the pipeline execution to ensure that data is being copied from each table in the SQL Server to the corresponding location in ADLS Gen2.

![enter image description here](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/2024-05-18%2020_01_26-.png)

![enter image description here](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/2024-05-18%2020_15_03-.png)


![enter image description here](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/2024-05-19%2020_53_35-bronze%20-%20Microsoft%20Azure%20and%206%20more%20pages%20-%20Sulara%20Perera%20-%20Microsoft%E2%80%8B%20Edge.png)

This configuration will iterate through all tables in your SQL Server, copying each one to a specified location in your ADLS Gen2 bronze container. Adjust the dataset and linked service details as per your specific setup.

# Data Transformation

## Transforming Data using Azure Databricks

Great! Now that you have your data loaded into the ADLS Gen2 bronze container and a Databricks cluster set up, you can start performing data transformations using Databricks. Here’s a step-by-step guide to help you get started with your data transformation tasks:

### Mount ADLS Gen2 to DBFS using Credential Passthrough


![enter image description here](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/12.png)

    configs = { "fs.azure.account.auth.type": "CustomAccessToken", "fs.azure.account.custom.token.provider.class": spark.conf.get("spark.databricks.passthrough.adls.gen2.tokenProviderClassName") }
    
    dbutils.fs.mount (
    	source = "abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/", mount_point = "/mnt/<mount-name>",
    	extra_configs = configs)

      Here you can replace <container-name>, <storage-account-name> and <mount-name> with azure resources names

### Or, you could use Service Principle to connect with ADLS

using a Service Principal to connect with Azure Data Lake Storage (ADLS) Gen2 is a common and secure way to manage data access and use a Service Principal to connect Databricks to ADLS Gen2:

#### Step 1: Set Up the Service Principal
Create a Service Principal:

#### Go to the Azure portal.
Navigate to "Azure Active Directory" > "App registrations" > "New registration".
Provide a name, select the supported account type, and register the application.
Note the "Application (client) ID" and "Directory (tenant) ID".

#### Create a Client Secret:

In the registered application, navigate to "Certificates & secrets".
Create a new client secret and note its value (you'll need this for configuration).
Assign Role to the Service Principal:

Navigate to your ADLS Gen2 storage account.
Go to "Access Control (IAM)" > "Add role assignment".
Assign the "Storage Blob Data Contributor" role to the Service Principal.

#### Step 2: Configure Databricks to Use the Service Principal
Set Up Configuration in Databricks:

Open your Databricks workspace and create a new notebook or open an existing one.
Use the following code to set up the configuration for ADLS Gen2 with the Service Principal:

    container_name =  "bronze"
    storage_account_name =  "adlscleverstudiesmrk"
    client_id =  "b82f9382-7e2b-4837-8652-51e3175eb23a"
    tenant_id =  "7effba51-b521-4654-913a-44f334bd092c"
    client_secret =  "sC58Q~MNHqurroe5_u0l.5gM-vFgk.UEyMK0waak"
    
    configs = {"fs.azure.account.auth.type": "OAuth",
    "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
    "fs.azure.account.oauth2.client.id": f"{client_id}",
    "fs.azure.account.oauth2.client.secret": f"{client_secret}",
    "fs.azure.account.oauth2.client.endpoint": f"https://login.microsoftonline.com/{tenant_id}/oauth2/token"}
    
    dbutils.fs.mount(
    source=f"abfss://{container_name}@{storage_account_name}.dfs.core.windows.net/",
    mount_point=  f"/mnt/{storage_account_name}/{container_name}",
    extra_configs= configs)


## Lets tranform the data (bronze to silver)

#### 1. Listing Files in the Directory

    # List files in the SalesLT directory
    dbutils.fs.ls("dbfs:/mnt/adlscleverstudiesmrk/bronze/SalesLT/")

#### 2. Extracting Table Names

    table_names = []
    for table in dbutils.fs.ls("dbfs:/mnt/adlscleverstudiesmrk/bronze/SalesLT/"):
        table_names.append(table.path.split('/')[-1])

-   This loop extracts the table names from the paths listed in the previous step.
-   **`table.path.split('/')[-1]`**: Splits the file path by '/' and takes the last part, which is the table name.


#### Importing Required Libraries

    from pyspark.sql.functions import from_utc_timestamp, date_format 
    from pyspark.sql.types import TimestampType

#### Processing Each Table

    for table in table_names:
        path = 'dbfs:/mnt/adlscleverstudiesmrk/bronze/SalesLT/' + table
        print(path)
        df = spark.read.format('parquet').load(path)
        columns = df.columns
-   Iterates over each table name, constructs the full path, and reads the data into a Spark DataFrame.
-   **`spark.read.format('parquet').load(path)`**: Reads the data from the specified path in Parquet format into a DataFrame.
-   **`df.columns`**: Retrieves the list of columns in the DataFrame.


#### Transforming Date Columns

    for col in columns:
        if "Date" in col or "date" in col:
            df = df.withColumn(col, date_format(from_utc_timestamp(df[col].cast(TimestampType()),"UTC"),"yyyy-MM-dd"))

-   Transforms columns containing dates to a specific date format.
-   **`if "Date" in col or "date" in col`**: Checks if the column name contains "Date" or "date".
-   **`df.withColumn`**: Creates a new column or replaces an existing column in the DataFrame.
-   **`df[col].cast(TimestampType())`**: Casts the column to a `TimestampType`.
-   **`from_utc_timestamp(df[col], "UTC")`**: Converts the timestamp to UTC.
-   **`date_format(..., "yyyy-MM-dd")`**: Formats the timestamp to the "yyyy-MM-dd" date format.

#### Writing Transformed Data

    output_path = 'dbfs:/mnt/adlscleverstudiesmrk/silver/SalesLT/' + table
    df.write.format('delta').mode("overwrite").save(output_path)


-   Writes the transformed DataFrame to the silver container in Delta format.
-   **`output_path`**: Constructs the output path for the transformed data.
-   **`df.write.format('delta').mode("overwrite").save(output_path)`**: Writes the DataFrame in Delta format to the specified path, overwriting any existing data.

This code effectively reads raw data from the bronze layer, applies necessary date transformations, and writes the cleaned and transformed data to the silver layer for further analysis or processing.


## Lets tranform the data (silver to gold)

    dbutils.fs.ls("dbfs:/mnt/adlscleverstudiesmrk/silver/SalesLT/")
    
    
    table_names = []
    for table in dbutils.fs.ls("dbfs:/mnt/adlscleverstudiesmrk/silver/SalesLT/"):
    table_names.append(table.name.split('/')[0])
    
    
    for table in table_names:
    path =  'dbfs:/mnt/adlscleverstudiesmrk/silver/SalesLT/'  + table
    print(path)
    df = spark.read.format('delta').load(path)
    column_names = df.columns
    
      
    for old_col_name in column_names:
    # convert column name from ColumnName to Column_Name format
    new_col_name = "".join(["_" + char if char.isupper() and not old_col_name[i -1].isupper() else char for i, char in enumerate(old_col_name)]).lstrip("_")
    df = df.withColumnRenamed(old_col_name, new_col_name)
    
      
    output_path =  'dbfs:/mnt/adlscleverstudiesmrk/gold/SalesLT/'  + table
    df.write.format('delta').mode("overwrite").save(output_path)

![enter image description here](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/2024-05-21%2021_38_32-getschema.sql%20-%20not%20connected_%20-%20Microsoft%20SQL%20Server%20Management%20Studio.png)

![enter image description here](https://github.com/sularaperera/End-to-End-Azure-Data-Engineering-Real-Time-Project/blob/main/Images/2024-05-21%2021_32_41-gold%20-%20Microsoft%20Azure%20and%205%20more%20pages%20-%20Sulara%20Perera%20-%20Microsoft%E2%80%8B%20Edge.png)


# Data Loading



# Data Reporting
