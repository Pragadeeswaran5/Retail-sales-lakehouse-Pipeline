# 🛒 Retail Sales Lakehouse Pipeline

## 📌 Overview

This project implements an end-to-end **Modern Data Lakehouse Pipeline on Azure** using the **Medallion Architecture (Bronze → Silver → Gold)**.

The pipeline ingests raw retail sales data from **Azure Data Lake Storage (ADLS)** using **Databricks Auto Loader**, applies data cleansing and deduplication in **Delta Lake**, builds a dimensional **Star Schema with SCD Type-2 customer history tracking**, and orchestrates execution with **Azure Data Factory (ADF)**.

---

## 🏗️ Architecture

### High-Level Data Flow

ADLS → Databricks Auto Loader → Bronze Layer →  
Silver Layer (Data Cleaning & Deduplication) →  
Gold Layer (Dimensions + SCD Type-2 + Fact Table) 

### Architecture Diagram

![Architecture](https://github.com/user-attachments/assets/f153c024-c336-4486-901a-bdc09f4d9b5a)

---

## 🛠️ Technology Stack

### ☁ Cloud & Storage  
- Azure Data Lake Storage Gen2 (ADLS)

### ⚙ Processing  
- Azure Databricks  
- PySpark  
- Delta Lake

### 📥 Ingestion  
- Databricks Auto Loader  
- Structured Streaming

### 🔁 Orchestration  
- Azure Data Factory (ADF)

### 📐 Data Modeling  
- Dimensional Modeling (Star Schema)  
- Fact & Dimension tables  
- Slowly Changing Dimensions – SCD Type-2

---

## 🥉 Bronze Layer – Raw Ingestion

### Purpose
Capture raw CSV data from ADLS into **Delta tables without transformation** while ensuring schema tracking and ingestion metadata capture.

### Responsibilities

- Streaming ingestion using **Databricks Auto Loader**
- Automatic schema inference and tracking
- Handling arriving files incrementally
- Appending ingestion timestamps

### Result

A raw, append-only Delta table that preserves original data fidelity for auditing and replay capability.

---

## 🥈 Silver Layer – Data Cleansing & Standardization

### Purpose
Transform raw data into analytics-ready datasets with standardized, validated, and de-duplicated records.

### Key Operations

- Data type casting and standardization
- Timestamp normalization
- Phone number normalization to **E.164 international format**
- Country enrichment and normalization using **ISO codes**
- Null handling and address cleansing
- Removal of unnecessary technical columns
- Business-key based deduplication using **window functions**
- Creation of processing timestamp for traceability

### Deduplication Strategy

Records are deduplicated using business keys such as:

- **Order Number**
- **Product Code**

Latest records are selected using the **latest processing timestamp** to guarantee correctness across re-processing or repeated file deliveries.

This ensures:

✅ Idempotent processing  
✅ Stable upstream merges  
✅ No duplicate downstream records

---

## 🥇 Gold Layer – Analytics Modeling

### Purpose

Transform cleaned Silver data into an **analytics-optimized Star Schema** using dimensions and a central fact table suitable for reporting and BI consumption.

---

### Product Dimension

Tracks unique product records.

Attributes:

- Product Code (Business Primary Key)
- Product Line
- MSRP

Updated using **MERGE UPSERT** logic.

---

### Date Dimension

Provides full temporal navigation.

Attributes:

- Date Key (generated surrogate key)
- Order Date
- Month ID
- Quarter ID
- Year ID

Date keys are derived directly from transaction dates to support efficient partitioning and joins.

---

### Customer Dimension — SCD Type-2

Tracks full historical changes to customer records.

Features:

- **Surrogate Customer ID generated**
- Versioned records using:
  - `Start_Date`
  - `End_Date`
  - `Is_Current` flag

Tracked attributes include:

- Customer address changes  
- Territory or country changes  
- Phone updates  
- Contact name changes  
- Deal-size classification updates

### SCD Behavior

| Scenario | Result |
|---------|---------|
| New customer | Insert new dimension record |
| Attribute change | Expire old record + insert new version |
| No change | No action |

This preserves **true customer change history** while maintaining a single current version for active lookups.

---

### Fact Table — Sales

Central transactional table that references all dimensions.

Contains:

- Order Number
- Product Code 
- Customer ID (SK)
- Date Key (FK)
- Quantity Ordered
- Unit Price
- Sales Amount
- Order Status

Fact loading is driven via **MERGE logic** keyed on:

- Order Number
- Product Code

This allows:

✅ Updates when customer assignments change  
✅ Correct history navigation  
✅ Idempotent re-load safety

---

## 🔁 Orchestration — Azure Data Factory (ADF)

### Pipeline Stages

1. **Bronze Ingestion Notebook**
2. **Silver Transformation Notebook**
3. **Gold Dimension Load Notebook**

ADF handles:

- Activity execution sequencing
- Job monitoring
- Failure retry
- Scheduling and parameter handling

---

## 📊 Data Model

### Star Schema Layout
                Date_Dim
                    |
Products_Dim - Fact_Sales - Customers_Dim (SCD-2)

Benefits:

- Efficient BI query performance
- Clean dimensional joins
- Full historical tracking for customers

---

## ✅ Final Outputs

### Bronze Tables
- `bronze.raw_bronze_tbl`

### Silver Tables
- `silver.silver_cleaned_tbl`

### Gold Dimensions
- `gold.products_dim`
- `gold.customers_dim`
- `gold.date_dim`

### Gold Fact
- `gold.fact_sales`

---

## 🎯 Key Engineering Capabilities Demonstrated

✅ Streaming ingestion using Auto Loader  
✅ Delta Lake structured processing  
✅ Medallion architecture design  
✅ SCD Type-2 implementation  
✅ Dimensional modeling best practices  
✅ Deduplication & idempotency controls  
✅ Production-style MERGE patterns  
✅ Cloud pipeline orchestration (ADF)

---

## 📘 Learning Outcomes

Through this project I gained hands-on experience with:

- Building lakehouse pipelines end to end on Azure
- Implementing real-world Slowly Changing Dimensions
- Delta Lake MERGE orchestration strategies
- Handling reprocessing & deduplication scenarios
- Star schema modeling for analytics platforms
- Enterprise orchestration tooling

---

✅ **This project represents a complete, production-style Azure Data Engineering pipeline built from ingestion to analytics layer with strong governance, scalability, and correctness guarantees.**

## **Contributors**
- Pragadeeswaran J

## **License**
Licensed under the [MIT License](LICENSE).
