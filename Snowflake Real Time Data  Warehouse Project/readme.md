# Snowflake Real-Time Data Warehouse Project for Beginners

[![Snowflake](https://img.shields.io/badge/Snowflake-29B5E8?style=for-the-badge&logo=snowflake&logoColor=white)](https://www.snowflake.com/)
[![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![AWS QuickSight](https://img.shields.io/badge/AWS%20QuickSight-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/quicksight/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Technologies Used](#technologies-used)
- [Prerequisites](#prerequisites)
- [Dataset](#dataset)
- [Project Flow](#project-flow)
- [Installation & Setup](#installation--setup)
- [Usage](#usage)
- [Key Concepts](#key-concepts)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

This project demonstrates how to build a **cloud-native data warehousing solution** using **Snowflake** on AWS. The focus is on designing an efficient, flexible, and cost-effective architecture that enables seamless data ingestion, transformation, and **real-time analytics** with minimal manual intervention.

The project uses **Tesla historical stock market data** to showcase real-world data engineering scenarios including external stage creation, storage integration, SnowSQL scripting, and automated streaming ingestion using **Snowpipe**.

### 🎪 Real-World Use Cases

**1. Customer Behavior Insights**
- Analyze customer purchase patterns across products and geographies
- Discover preferences and seasonal buying trends
- Personalize customer experiences based on historical data

**2. Operational Efficiency Monitoring**
- Monitor warehousing costs and data processing time
- Track user query behavior and optimize compute usage
- Ensure cost-effective and performant analytics

**3. Stock Price Trend Analysis**
- Track daily changes in stock prices
- Identify patterns and anomalies over long periods
- Support investment strategies based on historical trends

**4. Real-Time Market Alerts**
- Leverage Snowpipe and S3 event notifications
- Detect and ingest new data in near real-time
- Enable up-to-date analytics for financial analysts

## 🏗️ Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Local System / Data Source                │
│                                                                  │
│  ┌──────────────┐         ┌──────────────┐                     │
│  │  SnowSQL CLI │         │ CSV Files    │                     │
│  │  (Local Data)│         │ (Tesla Data) │                     │
│  └──────┬───────┘         └──────┬───────┘                     │
└─────────┼────────────────────────┼─────────────────────────────┘
          │                        │
          │                        ▼
          │              ┌─────────────────┐
          │              │   AWS S3 Bucket │
          │              │   (Data Lake)   │
          │              └────────┬────────┘
          │                       │
          │                       │ S3 Event Notification
          │                       │        (SQS)
          │                       ▼
          │              ┌─────────────────┐
          └─────────────▶│   Snowflake     │
                         │  Data Warehouse │
                         │                 │
                         │  • External Stage
                         │  • Snowpipe      │
                         │  • Storage Intg  │
                         │  • Tables        │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ AWS QuickSight  │
                         │  (Visualization)│
                         └─────────────────┘
```

### Data Flow Components

1. **Data Source**: Tesla stock market CSV files (local or S3)
2. **Storage Layer**: AWS S3 bucket for raw data storage
3. **Ingestion Layer**: SnowSQL CLI and Snowpipe for automated loading
4. **Processing Layer**: Snowflake warehouse for data transformation
5. **Visualization Layer**: AWS QuickSight for dashboards and analytics

## ✨ Features

### Core Capabilities

- ✅ **Multiple Data Loading Methods**:
  - SnowSQL CLI for local file uploads
  - External stages for S3 integration
  - Snowpipe for real-time automated ingestion
  
- ✅ **Secure Cloud Integration**:
  - Storage integration with IAM roles
  - Trust relationships for secure access
  - Encrypted data in transit and at rest

- ✅ **Real-Time Data Pipeline**:
  - S3 event notifications with SQS
  - Automatic Snowpipe triggers
  - Near real-time data availability

- ✅ **Advanced Snowflake Features**:
  - Time Travel for data recovery
  - Virtual warehouse management
  - Role-based access control (RBAC)

- ✅ **Business Intelligence**:
  - AWS QuickSight integration
  - Interactive dashboards
  - Stock market trend analysis

## 🛠️ Technologies Used

| Technology | Purpose | Version |
|-----------|---------|---------|
| **Snowflake** | Cloud Data Warehouse | Standard Edition |
| **AWS S3** | Object Storage / Data Lake | - |
| **AWS QuickSight** | Business Intelligence & Visualization | Standard |
| **SnowSQL** | Command-Line Client | Latest |
| **Python** | Data Processing | 3.x |
| **AWS IAM** | Identity and Access Management | - |
| **AWS SQS** | Simple Queue Service (Event Notifications) | - |

## 📦 Prerequisites

### Required Accounts

1. **AWS Account**
   - Free tier eligible
   - Access to S3, IAM, and QuickSight services
   - Programmatic access (Access Key ID & Secret Key)

2. **Snowflake Account**
   - Free trial ($400 credit)
   - Standard edition or higher
   - ACCOUNTADMIN role access

### Local Requirements

- Terminal/Command Prompt access
- SnowSQL CLI installed
- Web browser (Chrome, Firefox, or Edge)
- Basic knowledge of:
  - SQL queries
  - Cloud computing concepts
  - Command-line operations

### Estimated Costs

**Free Tier / Trial Period**:
- AWS S3: ~$0.01-0.50 for small datasets
- Snowflake: $400 free credits (lasts several weeks for this project)
- AWS QuickSight: 30-day free trial

**Monthly Costs** (if exceeding free tier):
- Snowflake Standard: ~$2 per credit/hour
- AWS S3: ~$0.023 per GB
- AWS QuickSight: ~$9/month (Standard)

## 📊 Dataset

### Tesla Stock Market Data

The project uses **Tesla (TSLA) historical stock market data** stored in CSV format.

**Columns**:
- `Date`: Trading date (e.g., 2010-06-29)
- `Open`: Opening stock price
- `High`: Highest price of the day
- `Low`: Lowest price of the day
- `Close`: Closing stock price
- `Adj Close`: Adjusted closing price
- `Volume`: Number of shares traded

**Sample Data**:
```csv
Date,Open,High,Low,Close,Adj Close,Volume
2010-06-29,19.00,25.00,17.54,23.89,23.89,18766300
2010-06-30,25.79,30.42,23.30,23.83,23.83,17187100
```

The dataset is ideal for showcasing both batch and real-time ingestion into Snowflake using manual and automated approaches.

## 🔄 Project Flow

### Complete Pipeline

```
1. Raw Data Storage (S3)
   ↓
2. Snowflake Setup (Database, Schema, Warehouse)
   ↓
3. Table Creation (TESLA_DATA table)
   ↓
4a. Manual Loading via SnowSQL
    • Create storage integration
    • Define external stage
    • Execute COPY INTO command
   ↓
4b. Automated Loading via Snowpipe
    • Configure S3 event notifications
    • Create Snowpipe with AUTO_INGEST
    • Automatic data loading on file upload
   ↓
5. Data Transformation (SQL queries)
   ↓
6. Visualization (AWS QuickSight dashboards)
```

## 🚀 Installation & Setup

### Quick Start

**Step 1: Create AWS S3 Bucket**
```bash
# Using AWS CLI
aws s3 mb s3://snowflake-tesla-data --region us-east-1

# Or use AWS Console (recommended for beginners)
```

**Step 2: Upload Data to S3**
```bash
aws s3 cp TSLA.csv s3://snowflake-tesla-data/Input/
```

**Step 3: Create Snowflake Account**
- Visit [signup.snowflake.com](https://signup.snowflake.com)
- Choose AWS as cloud provider
- Select Standard edition
- Choose same region as your S3 bucket

**Step 4: Install SnowSQL**
```bash
# Download from Snowflake documentation
# https://docs.snowflake.com/en/user-guide/snowsql-install-config

# Verify installation
snowsql --version
```

**Step 5: Configure SnowSQL**
```bash
# Edit config file
# Windows: C:\Users\<username>\.snowsql\config
# Mac/Linux: ~/.snowsql/config

[connections.my_snowflake]
accountname = <your-account-name>
username = <your-username>
password = <your-password>
region = <your-region>
```

For detailed step-by-step instructions, see [SETUP_GUIDE.md](SETUP_GUIDE.md)

## 💡 Usage

### Method 1: Loading Data via SnowSQL (Local Files)

```sql
-- Create database and table
CREATE DATABASE TEST_DB;
USE DATABASE TEST_DB;

CREATE TABLE customer_detail (
  first_name STRING,
  last_name STRING,
  address STRING,
  city STRING,
  state STRING
);

-- Create file format
CREATE FILE FORMAT PIPE_FORMAT_CLI
  TYPE = 'CSV'
  FIELD_DELIMITER = '|'
  SKIP_HEADER = 1;

-- Create stage
CREATE STAGE PIPE_CLI_STAGE
  FILE_FORMAT = PIPE_FORMAT_CLI;

-- Upload file from local
PUT file:///path/to/customer_detail.csv @PIPE_CLI_STAGE auto_compress=true;

-- Load data into table
COPY INTO customer_detail
  FROM @PIPE_CLI_STAGE
  FILE_FORMAT = PIPE_FORMAT_CLI;
```

### Method 2: Loading Data from S3 (Manual)

```sql
-- Create table for Tesla data
CREATE TABLE TESLA_DATA (
  Date DATE,
  Open_value DOUBLE,
  High_value DOUBLE,
  Low_value DOUBLE,
  Close_value DOUBLE,
  Adj_Close DOUBLE,
  volume BIGINT
);

-- Create external stage
CREATE STAGE BULK_COPY_TESLA_STAGE
  URL='s3://snowflake-tesla-data/Input/'
  CREDENTIALS=(
    AWS_KEY_ID='<your-access-key>'
    AWS_SECRET_KEY='<your-secret-key>'
  );

-- Copy data from stage
COPY INTO TESLA_DATA
  FROM @BULK_COPY_TESLA_STAGE
  FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER = ',' SKIP_HEADER = 1);
```

### Method 3: Real-Time Loading with Snowpipe

```sql
-- Create storage integration
CREATE STORAGE INTEGRATION S3_INTEGRATION
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = S3
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::xxxx:role/SnowflakeRole'
  ENABLED = TRUE
  STORAGE_ALLOWED_LOCATIONS = ('s3://snowflake-tesla-data/Input/');

-- Create file format
CREATE FILE FORMAT CSV_FORMAT
  TYPE = 'CSV'
  FIELD_OPTIONALLY_ENCLOSED_BY = '"'
  SKIP_HEADER = 1;

-- Create stage with storage integration
CREATE STAGE TESLA_DATA_STAGE
  URL='s3://snowflake-tesla-data/Input/'
  STORAGE_INTEGRATION = S3_INTEGRATION
  FILE_FORMAT = CSV_FORMAT;

-- Create Snowpipe for auto-ingestion
CREATE PIPE tesla_pipe_test
  AUTO_INGEST = TRUE
AS
  COPY INTO TESLA_DATA
  FROM @TESLA_DATA_STAGE
  FILE_FORMAT = CSV_FORMAT;

-- Get notification channel ARN
SHOW PIPES;
```

### Configuring S3 Event Notifications

1. Go to AWS S3 Console → Your bucket → Properties
2. Create event notification:
   - Event type: "All object create events"
   - Destination: SQS Queue
   - SQS Queue ARN: (from SHOW PIPES output)

### Connecting AWS QuickSight

1. Open AWS QuickSight Console
2. Create new dataset → Select Snowflake
3. Provide connection details:
   - Server: `<account>.snowflakecomputing.com`
   - Database: `TEST_DB`
   - Warehouse: `COMPUTE_WH`
   - Username/Password: Your credentials
4. Select `TESLA_DATA` table
5. Create visualizations (line charts, bar charts, etc.)

## 🔑 Key Concepts

### Snowflake Architecture

**Three-Layer Architecture**:

1. **Cloud Services Layer**: Authentication, metadata, query optimization
2. **Query Processing Layer**: Virtual warehouses (compute clusters)
3. **Database Storage Layer**: Columnar storage (decoupled from compute)

### Virtual Warehouse

A **virtual warehouse** is an independent compute cluster in Snowflake:

- Sizes: X-Small, Small, Medium, Large, X-Large, 2X-Large, 3X-Large, 4X-Large
- Billing: Per-second usage in Snowflake credits
- Auto-suspend: Automatically pause when idle
- Auto-resume: Restart on new query

**Cost Optimization**:
```sql
-- Set auto-suspend to 5 minutes (300 seconds)
ALTER WAREHOUSE COMPUTE_WH SET AUTO_SUSPEND = 300;

-- Manually suspend when done
ALTER WAREHOUSE COMPUTE_WH SUSPEND;
```

### Storage Integration

**Purpose**: Securely connect Snowflake to AWS S3 without hardcoded credentials

**Benefits**:
- No credentials stored in Snowflake
- Uses IAM roles and trust relationships
- Enhanced security and compliance
- Easier credential rotation

### Snowpipe

**What it does**: Automatically loads data from external stages as new files arrive

**How it works**:
1. S3 event notification → SQS queue
2. SQS message → Snowpipe
3. Snowpipe → COPY INTO (automated)

**Key Features**:
- Serverless (no warehouse required during ingestion)
- Near real-time latency
- Automatic deduplication
- Metadata tracking

### Time Travel

**Capability**: Query, restore, or clone data as it existed at a previous point in time

**Use Cases**:
- Recover accidentally deleted data
- Audit historical changes
- Compare current vs. past data

**Examples**:
```sql
-- Restore dropped table
DROP TABLE tesla_data;
UNDROP TABLE tesla_data;

-- Query data before a specific statement
SELECT * FROM tesla_data 
  BEFORE (STATEMENT => '<statement-id>');

-- Clone table as it was 1 hour ago
CREATE TABLE tesla_data_backup 
  CLONE tesla_data AT (OFFSET => -3600);
```

## 🐛 Troubleshooting

### Common Issues

**1. SnowSQL Connection Failed**
```bash
# Check account name format
# Should be: <account>.<region_id>
# Example: xy12345.us-east-1

# Verify credentials in config file
cat ~/.snowsql/config
```

**2. S3 Access Denied**
- Verify IAM role has correct permissions
- Check trust relationship includes Snowflake ARN
- Ensure External ID is configured correctly

**3. Snowpipe Not Triggering**
- Confirm S3 event notification is configured
- Check SQS ARN matches pipe notification channel
- Verify STORAGE_ALLOWED_LOCATIONS includes correct path

**4. QuickSight Connection Error**
- Ensure Snowflake warehouse is running
- Verify VPC/security group settings
- Check username/password are correct

**5. Data Not Loading**
```sql
-- Check copy history
SELECT * FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
  TABLE_NAME => 'TESLA_DATA',
  START_TIME => DATEADD(hours, -1, CURRENT_TIMESTAMP())
));

-- Check pipe status
SELECT SYSTEM$PIPE_STATUS('tesla_pipe_test');
```

## 📋 Best Practices

### Cost Management

1. **Warehouse Management**:
   - Use smallest warehouse size that meets performance needs
   - Set aggressive auto-suspend (60-300 seconds)
   - Suspend warehouses when not in use

2. **Storage Optimization**:
   - Compress data before uploading to S3
   - Use appropriate file sizes (100-250 MB compressed)
   - Clean up old files in S3 regularly

3. **Query Optimization**:
   - Use clustering keys for large tables
   - Leverage query result caching
   - Monitor query history and costs

### Security Best Practices

1. **Credentials**:
   - Never hardcode AWS credentials
   - Use storage integrations instead
   - Rotate access keys regularly

2. **Access Control**:
   - Use RBAC (role-based access control)
   - Grant minimum required privileges
   - Create separate roles for different use cases

3. **Networking**:
   - Use private connectivity when possible
   - Enable MFA for Snowflake accounts
   - Restrict IP access where applicable

### Data Pipeline Best Practices

1. **File Organization**:
   - Use consistent naming conventions
   - Include timestamps in filenames
   - Organize by date partitions (year/month/day)

2. **Error Handling**:
   - Monitor COPY command errors
   - Set up alerts for failed loads
   - Implement retry logic for transient errors

3. **Testing**:
   - Test with small datasets first
   - Validate data quality after loading
   - Monitor end-to-end latency

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/AmazingFeature`
3. Commit your changes: `git commit -m 'Add AmazingFeature'`
4. Push to the branch: `git push origin feature/AmazingFeature`
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Snowflake documentation and community
- AWS documentation and tutorials
- Open source contributors

## 📞 Contact & Support

- **Project Maintainer**: Abid Aziz
- **Repository**: [Snowflake-Projects](https://github.com/abidaziz1/Snowflake-Projects)
- **Issues**: [Report a bug](https://github.com/abidaziz1/Snowflake-Projects/issues)

---

## 🎓 Learning Resources

- [Snowflake Documentation](https://docs.snowflake.com/)
- [AWS S3 Documentation](https://docs.aws.amazon.com/s3/)
- [AWS QuickSight User Guide](https://docs.aws.amazon.com/quicksight/)
- [SnowSQL Reference](https://docs.snowflake.com/en/user-guide/snowsql)

---

**⭐ If you find this project helpful, please consider giving it a star!**

**Made with ❤️ for aspiring data engineers**
