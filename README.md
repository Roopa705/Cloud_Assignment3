# AWS Data Processing Pipeline

This project demonstrates an end-to-end serverless data processing pipeline on AWS. The process involves ingesting raw data, using a serverless function to process it, cataloging the data for querying, and finally, visualizing the results on a dynamic webpage.

# Architecture Components

## 1. Data Staging in Amazon S3 🪣

The foundation of the pipeline is Amazon S3 (Simple Storage Service). A specific folder structure is used to create a logical workflow for the data as it moves through the pipeline:

- **`raw/`**: This folder acts as the ingestion point or "data lake." All raw, unmodified data files (like `Orders.csv`) are uploaded here first.

- **`processed/`**: This folder stores the output from our processing step. After the raw data is cleaned, filtered, or transformed, the resulting "clean" data is saved here. This separates the original data from the processed, query-ready data.

- **`enriched/`**: This folder is a designated output location for results from Amazon Athena queries. When we run analytical queries (e.g., "total sales per month"), the resulting summary tables or CSV files are stored here.

## 2. Secure Service-to-Service Communication (IAM Roles) 🔐

This pipeline involves multiple AWS services that need to interact (S3, Lambda, Glue, EC2). Instead of hardcoding access keys (a major security risk), we use IAM (Identity and Access Management) Roles. A role is a set of permissions that a service can "assume" to securely access other services.

- **Lambda Execution Role**: This role grants the Lambda function permission to perform its job: reading files from the `raw/` S3 folder and writing its processed output to the `processed/` S3 folder.

- **Glue Service Role**: This role allows the AWS Glue Crawler to access and scan the files within the `processed/` S3 folder. This is necessary for it to discover the data's schema.

- **EC2 Instance Profile**: This role is attached to our web server. It gives the application running on the server (the Flask app) permission to interact with other AWS services, specifically to run queries on Amazon Athena and access S3 for query results.

## 3. Serverless Data Processing (Lambda Function) ⚙️

The core processing logic of this pipeline lives in an AWS Lambda function. Lambda is a "serverless" compute service, meaning it runs code in response to events without requiring us to manage a server.

In this project, the **FilterAndProcessOrders** function's job is to:

1. Receive an input event (which we'll set up in the next step).
2. Read the raw data file (e.g., `Orders.csv`) that triggered the event.
3. Process the data in memory. This could involve cleaning messy data, filtering out incomplete orders, or standardizing date formats.
4. Write the new, clean data to a file in the `processed/` S3 folder.

## 4. Event-Driven Automation (S3 Trigger) ⚡

This step makes the pipeline automated and event-driven. We configure an **S3 Event Notification** that acts as a "trigger."

This trigger constantly watches the `raw/` folder in our S3 bucket. When it detects a new object (specifically, a `.csv` file) being created, it automatically invokes our **FilterAndProcessOrders** Lambda function, passing in the details of the file that just arrived. This setup means the moment new data is uploaded, the processing begins immediately without any manual intervention.

## 5. Data Discovery and Cataloging (Glue Crawler) 🕸️

At this point, we have clean data sitting in the `processed/` S3 folder, but it's just a file. To query it using SQL, we need a formal "table" definition, or schema.

An **AWS Glue Crawler** automates this. We point the crawler at our `processed/` folder. It scans the data, automatically infers the schema (e.g., "column 1 is a string, column 2 is an integer"), and creates a table definition in the **AWS Glue Data Catalog**. This catalog acts as a central metadata repository. The data itself doesn't move; the catalog just stores the metadata that points to the data's location in S3.

## 6. Serverless SQL Querying (Amazon Athena) 🔍

With our data cataloged, we can now use **Amazon Athena** to query it. Athena is a serverless, interactive query service that allows you to run standard SQL queries directly on data stored in S3.

It uses the Glue Data Catalog to understand the schema and location of our processed table. This allows us to perform powerful business intelligence (BI) and analytical queries, such as:

- Calculating total sales by customer
- Aggregating revenue by month
- Summarizing order statuses
- Finding the average order value

## 7. The Web Dashboard (EC2 and Flask) 🖥️

This final component is the user-facing dashboard that visualizes our query results. It consists of a few parts:

- **EC2 Instance**: A virtual server (Amazon Elastic Compute Cloud) that hosts our web application. We configure its security group (a virtual firewall) to allow public traffic on port 5000 (for the web app) and 22 (for us to log in securely via SSH).

- **Flask Web Application**: A lightweight web server written in Python (`app.py`).

- **Boto3**: The AWS SDK (Software Development Kit) for Python. This library is used within our Flask app to programmatically execute the Athena queries.

When a user accesses the web page, the Flask application:

1. Receives the request
2. Uses Boto3 to send the SQL queries (from Section 6) to Amazon Athena
3. Receives the query results back from Athena
4. Renders those results into an HTML page and sends it to the user's browser

This provides a dynamic dashboard where the data is fetched in near real-time from our data pipeline.

## Data Flow Summary

```
1. Raw data uploaded to S3 (raw/)
   ↓
2. S3 trigger detects new file
   ↓
3. Lambda function processes data
   ↓
4. Clean data saved to S3 (processed/)
   ↓
5. Glue Crawler catalogs the data
   ↓
6. Athena queries the cataloged data
   ↓
7. Flask app displays results on web dashboard
```

## Key AWS Services Used

- **Amazon S3**: Object storage for data files
- **AWS Lambda**: Serverless compute for data processing
- **AWS Glue**: Data catalog and schema discovery
- **Amazon Athena**: Serverless SQL query engine
- **Amazon EC2**: Virtual server for web application
- **IAM**: Identity and access management for secure service communication

## Benefits of This Architecture

- **Serverless**: No infrastructure to manage for Lambda and Athena
- **Event-driven**: Automatic processing when new data arrives
- **Scalable**: Can handle varying data volumes
- **Cost-effective**: Pay only for resources used
- **Secure**: IAM roles ensure proper access control
- **Maintainable**: Clear separation of concerns between components
# Cloud_Assignment3
