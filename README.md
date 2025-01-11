# Portfolio
Analytics Portfolio
# Digital Signature ETL Pipeline

## Overview

This project implements an ETL pipeline designed to process digital signature data from **DocuSign** and **OneSpan**. The goal is to extract, transform, and load the data into a structured format for generating insights via a Power BI dashboard. 

The pipeline is robust, handling data ingestion from emails, transforming it into structured datasets, and merging it for easy analysis. By integrating AWS S3 and Secrets Manager, the pipeline ensures secure storage and efficient processing.

---

## Problem Statement

The company is a third-party partner with **DocuSign** and **OneSpan**, purchasing envelopes in bulk and selling them to clients. However, there was no effective internal mechanism to track how many envelopes each client had consumed. 

The current approach required sales teams to manually access the provider domains to gather this information. This inefficiency resulted in delays and limited visibility into stock levels, affecting the ability to proactively reach out to clients when contracted volumes were running low.

### **Proposed Solution**

To address this issue, the pipeline was developed to:
1. Automate the extraction of envelope consumption data sent via emails from DocuSign and OneSpan.
2. Store and transform this data for use in internal reporting systems.
3. Provide real-time insights through a **Power BI dashboard** that allows the sales team to:
   - Monitor stock levels.
   - Take timely action to engage with clients nearing their contracted limits.
   - Improve overall operational efficiency.

By leveraging emails for data ingestion, this approach eliminated the need for direct website scraping or additional AWS email services, offering a cost-effective and scalable solution.

---

## Features

### **1. Email Processing and Data Extraction**
   - Fetches unread emails and attachments using Microsoft Graph API.
   - Filters relevant attachments (ZIP files containing CSVs) based on client-specific configurations.
   - Supports password-protected ZIP files with predefined passwords for specific clients (e.g., OneSpan).
   - Uploads extracted files to AWS S3 in a structured path.

### **2. Data Transformation**
   - Processes raw data into structured, client-specific formats:
     - **DocuSign Envelopes:** Standardizes columns, converts timezones to CEST, and calculates aggregated metrics.
     - **DocuSign SMS:** Filters irrelevant rows, enriches metadata, and dynamically calculates period start and end dates.
     - **OneSpan Envelopes:** Extracts transaction-specific metrics (e.g., completion times, statuses) and calculates average metrics dynamically.

### **3. Data Merging**
   - Combines transformed DocuSign and OneSpan envelope datasets.
   - Merges SMS and Envelope reports into a unified dataset.
   - Constructs a dynamic `date_key` column based on available dates, ensuring consistency across datasets.

### **4. AWS Integration**
   - Utilizes AWS S3 for storing raw and processed data with structured paths.
   - Manages email authentication credentials securely with AWS Secrets Manager.

### **5. Logging and Error Handling**
   - Comprehensive logging tracks all key actions, errors, and warnings in a `etl_pipeline.log` file.
   - Handles exceptions for authentication failures, invalid ZIP files, missing data, and more.

---

## ETL Workflow

The ETL pipeline is divided into three main phases:

### **1. Raw Data Extraction**
- **Script:** `Signature/raw_data_processing/extract_email_to_s3.py`
- **Purpose:** Extracts email attachments, processes ZIP files, and uploads CSV files to the raw S3 bucket.
- **Highlights:**
  - Filters emails based on subject keywords and attachment types.
  - Supports password-protected ZIPs (e.g., OneSpan ZIP files with the password "Branddocs").
  - Dynamically organizes files into S3 paths based on client, year, and month.

### **2. Data Transformation**
- **DocuSign Envelopes**
  - **Script:** `Signature/transformations/process_docusign_data.py`
  - **Purpose:** Processes envelope data into a structured format by standardizing columns, converting timezones, and enriching metadata.
- **DocuSign SMS**
  - **Script:** `Signature/transformations/process_docusign_sms_data.py`
  - **Purpose:** Processes SMS data, filters irrelevant rows, and calculates period start and end dates dynamically.
- **OneSpan Envelopes**
  - **Script:** `Signature/transformations/process_onespan_data.py`
  - **Purpose:** Processes OneSpan envelope data, calculates transaction metrics (e.g., completion times), and enriches with metadata.

### **3. Data Merging**
- **DocuSign and OneSpan Merge**
  - **Script:** `Signature/transformations/Merge_files.py`
  - **Purpose:** Combines transformed DocuSign and OneSpan envelope datasets into a unified file.
- **SMS and Envelope Merge**
  - **Script:** `Signature/transformations/sms_merge.py`
  - **Purpose:** Merges DocuSign SMS and Envelope reports into a single dataset, creating a dynamic `date_key` column.

---

## Usage

### **Command-Line Arguments**
The `main.py` script serves as the central entry point for executing the ETL pipeline.

#### Run the Complete Pipeline:
---bash
python main.py --etl_type=all

# Run only raw extraction:
---bash
python main.py --etl_type=raw

# Run Transformation Phases:
DocuSign Envelopes:
---bash
python main.py --etl_type=transformed_docusign_envelope

# DocuSign SMS:
python main.py --etl_type=transformed_docusign_sms

# OneSpan Envelopes:
---bash
python main.py --etl_type=transformed_onespan

### Run Data Merging:
# Merge DocuSign and OneSpan:
---bash
python main.py --etl_type=merge
# Merge SMS and Envelopes:
---bash
python main.py --etl_type=merge_sms_envelope








