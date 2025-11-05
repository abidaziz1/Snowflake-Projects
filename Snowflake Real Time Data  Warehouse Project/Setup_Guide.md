# Complete Setup Guide: Snowflake Real-Time Data Warehouse

This comprehensive guide will walk you through every step of building a real-time data warehouse using Snowflake, AWS S3, and AWS QuickSight.

## 📑 Table of Contents

1. [Prerequisites Checklist](#prerequisites-checklist)
2. [AWS Account Setup](#aws-account-setup)
3. [Snowflake Account Setup](#snowflake-account-setup)
4. [Creating S3 Bucket](#creating-s3-bucket)
5. [Installing SnowSQL](#installing-snowsql)
6. [Method 1: Loading Local Data](#method-1-loading-local-data-via-snowsql)
7. [Method 2: Loading from S3 (Manual)](#method-2-loading-from-s3-manual)
8. [Method 3: Real-Time Loading (Snowpipe)](#method-3-real-time-loading-with-snowpipe)
9. [AWS QuickSight Visualization](#aws-quicksight-visualization)
10. [Testing Time Travel](#testing-time-travel)

---

## 1. Prerequisites Checklist

Before starting, ensure you have:

- [ ] AWS account with billing enabled
- [ ] Snowflake free trial account (or paid)
- [ ] Terminal/Command Prompt access
- [ ] Web browser (Chrome/Firefox recommended)
- [ ] Dataset downloaded (Tesla stock market data)
- [ ] Basic understanding of:
  - SQL queries
  - Cloud computing concepts
  - Command-line operations

### Estimated Setup Time
- **First-time setup**: 1.5-2 hours
- **With prior experience**: 30-45 minutes

### Estimated Costs
- **AWS S3**: ~$0.01-0.50/month for small datasets
- **Snowflake**: $400 free trial credits (lasts several weeks)
- **AWS QuickSight**: 30-day free trial, then $9/month

---

## 2. AWS Account Setup

### Step 2.1: Create AWS Account

1. Visit [aws.amazon.com](https://aws.amazon.com)
2. Click **"Create an AWS Account"**
3. Complete registration:
   - Email address
   - Payment method (for verification)
   - Phone verification
   - Support plan: Basic (Free)

### Step 2.2: Set Up Billing Alerts

⚠️ **Critical**: Prevent unexpected charges

1. Navigate to **AWS Billing Console**
2. Go to **Billing Preferences**
3. Enable:
   - **Receive Free Tier Usage Alerts**
   - **Receive Billing Alerts**
4. Create **Budget**:
   - Monthly budget: $20-50 (adjust as needed)
   - Alert at 80% and 100%

### Step 2.3: Create IAM Access Keys

1. Go to **IAM Console**
2. Click **"Users"** → **"Security credentials"**
3. Scroll to **"Access keys"**
4. Click **"Create access key"**
5. **Download and save securely**:
   - Access Key ID
   - Secret Access Key

⚠️ **Important**: Never share these keys publicly or commit them to Git!

---

## 3. Snowflake Account Setup

### Step 3.1: Create Snowflake Account

1. Visit [signup.snowflake.com](https://signup.snowflake.com)
2. Fill in registration details:
   - First name, last name
   - Work email
   - Reason: "Learning" or "Personal project"
3. Choose edition: **Standard**
4. Cloud provider: **AWS**
5. Region: **Same as your S3 bucket** (e.g., us-east-1)
6. Click **"Get Started"**

### Step 3.2: Activate Account

1. Check your email for activation link
2. Click **"Activate"**
3. Set password
4. Complete setup questions

### Step 3.3: Initial Configuration

```sql
-- Login to Snowflake Web UI
-- These commands are optional but recommended

-- Verify warehouse
SHOW WAREHOUSES;

-- Create database (if needed)
CREATE DATABASE IF NOT EXISTS TEST_DB;

-- Create schema
CREATE SCHEMA IF NOT EXISTS TEST_DB.PUBLIC;
```

### Step 3.4: Configure Auto-Suspend

```sql
-- Set warehouse to suspend after 5 minutes of inactivity
ALTER WAREHOUSE COMPUTE_WH SET AUTO_SUSPEND = 300;

-- Enable auto-resume
ALTER WAREHOUSE COMPUTE_WH SET AUTO_RESUME = TRUE;
```

---

## 4. Creating S3 Bucket

### Step 4.1: Create Bucket

1. Open **AWS Console** → Search for **"S3"**
2. Click **"Create bucket"**
3. Configuration:
   - **Bucket name**: `snowflake-tesla-data-<unique-id>`
   - **Region**: us-east-1 (or your preferred region)
   - **Block Public Access**: Keep all boxes checked ✅
   - **Versioning**: Disabled
   - **Encryption**: Enable (default)
4. Click **"Create bucket"**

### Step 4.2: Create Folder Structure

1. Open your bucket
2. Click **"Create folder"**
3. Name: `Input`
4. Click **"Create folder"**

### Step 4.3: Upload Tesla Data

1. Download `TSLA.csv` from project data folder
2. Open `Input` folder
3. Click **"Upload"**
4. Select `TSLA.csv` file
5. Click **"Upload"**

**Verify**: You should see `TSLA.csv` in `s3://snowflake-tesla-data/Input/`

---

## 5. Installing SnowSQL

### Step 5.1: Download SnowSQL

1. Search "install SnowSQL" in browser
2. Go to official Snowflake documentation
3. Download for your OS:
   - **Windows**: `.msi` installer
   - **macOS**: `.pkg` or `.dmg`
   - **Linux**: `.bash` installer

### Step 5.2: Install SnowSQL

**Windows**:
```bash
# Double-click the .msi file
# Follow installation wizard
```

**macOS**:
```bash
# Open .pkg file
# Follow installation steps
```

**Linux**:
```bash
# Download installer
curl -O https://sfc-repo.snowflakecomputing.com/snowsql/bootstrap/1.2/linux_x86_64/snowsql-1.2.28-linux_x86_64.bash

# Make executable
chmod +x snowsql-*.bash

# Run installer
./snowsql-*.bash
```

### Step 5.3: Verify Installation

```bash
# Check version
snowsql --version

# Should output: Version: 1.2.x
```

### Step 5.4: Configure SnowSQL

**Locate config file**:
- Windows: `C:\Users\<username>\.snowsql\config`
- Mac/Linux: `~/.snowsql/config`

**Edit config file**:
```ini
[connections.my_snowflake]
accountname = <your-account-identifier>
username = <your-snowflake-username>
password = <your-password>
region = us-east-1

# Optional settings
warehousename = COMPUTE_WH
dbname = TEST_DB
schemaname = PUBLIC
```

**Finding your account identifier**:
1. Login to Snowflake Web UI
2. Look at URL: `https://<account>.snowflakecomputing.com`
3. Account identifier is the `<account>` part

### Step 5.5: Test Connection

```bash
# Connect using SnowSQL
snowsql

# If config is correct, you'll see:
# * SnowSQL * v1.2.x
# * Username: <your-username>
# * Account: <your-account>
# <username>#COMPUTE_WH@TEST_DB.PUBLIC>
```

---

## 6. Method 1: Loading Local Data via SnowSQL

### Step 6.1: Prepare Sample Data

Create a test file `customer_detail.csv`:
```csv
first_name|last_name|address|city|state
John|Doe|123 Main St|New York|NY
Jane|Smith|456 Oak Ave|Los Angeles|CA
Bob|Johnson|789 Pine Rd|Chicago|IL
```

### Step 6.2: Create Database and Table

**In SnowSQL terminal**:
```sql
-- Create database
CREATE DATABASE TEST_DB;

-- Use database
USE DATABASE TEST_DB;

-- Create table
CREATE OR REPLACE TABLE customer_detail (
  first_name STRING,
  last_name STRING,
  address STRING,
  city STRING,
  state STRING
);

-- Verify table is empty
SELECT * FROM customer_detail;
```

### Step 6.3: Define File Format

```sql
CREATE OR REPLACE FILE FORMAT PIPE_FORMAT_CLI
  TYPE = 'CSV'
  FIELD_DELIMITER = '|'
  SKIP_HEADER = 1;
```

### Step 6.4: Create Stage

```sql
CREATE OR REPLACE STAGE PIPE_CLI_STAGE
  FILE_FORMAT = PIPE_FORMAT_CLI;
```

### Step 6.5: Upload File to Stage

```sql
-- Upload from local machine
PUT file:///path/to/customer_detail.csv @PIPE_CLI_STAGE auto_compress=true;

-- Example Windows:
-- PUT file://C:\Users\YourName\Downloads\customer_detail.csv @PIPE_CLI_STAGE auto_compress=true;

-- Example Mac/Linux:
-- PUT file:///Users/YourName/Downloads/customer_detail.csv @PIPE_CLI_STAGE auto_compress=true;

-- Verify upload
LIST @PIPE_CLI_STAGE;
```

### Step 6.6: Load Data into Table

```sql
-- Resume warehouse (if suspended)
ALTER WAREHOUSE COMPUTE_WH RESUME;

-- Copy data from stage to table
COPY INTO customer_detail
  FROM @PIPE_CLI_STAGE
  FILE_FORMAT = (FORMAT_NAME = PIPE_FORMAT_CLI)
  ON_ERROR = 'skip_file';

-- Verify data loaded
SELECT * FROM customer_detail;
SELECT COUNT(*) FROM customer_detail;
```

**Expected Output**: 3 rows loaded

---

## 7. Method 2: Loading from S3 (Manual)

### Step 7.1: Create Tesla Data Table

**In Snowflake Web UI or SnowSQL**:
```sql
USE DATABASE TEST_DB;

CREATE OR REPLACE TABLE TESLA_DATA (
  Date DATE,
  Open_value DOUBLE,
  High_value DOUBLE,
  Low_value DOUBLE,
  Close_value DOUBLE,
  Adj_Close DOUBLE,
  volume BIGINT
);
```

### Step 7.2: Create External Stage (Simple Method)

```sql
-- Using AWS credentials directly
CREATE OR REPLACE STAGE BULK_COPY_TESLA_STAGE
  URL='s3://snowflake-tesla-data/Input/'
  CREDENTIALS=(
    AWS_KEY_ID='<your-access-key-id>'
    AWS_SECRET_KEY='<your-secret-access-key>'
  );

-- List files in stage
LIST @BULK_COPY_TESLA_STAGE;
```

### Step 7.3: Load Data from Stage

```sql
-- Copy data into table
COPY INTO TESLA_DATA
  FROM @BULK_COPY_TESLA_STAGE
  FILE_FORMAT = (
    TYPE = CSV 
    FIELD_DELIMITER = ',' 
    SKIP_HEADER = 1
  );

-- Verify data loaded
SELECT * FROM TESLA_DATA LIMIT 10;
SELECT COUNT(*) FROM TESLA_DATA;
```

---

## 8. Method 3: Real-Time Loading with Snowpipe

### Step 8.1: Create IAM Role in AWS

1. Go to **AWS IAM Console**
2. Click **"Roles"** → **"Create role"**
3. Select **"AWS service"**
4. Use case: **Custom trust policy**
5. Trust policy (temporary - will update later):
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "Service": "s3.amazonaws.com"
    },
    "Action": "sts:AssumeRole"
  }]
}
```
6. Click **"Next"**

### Step 8.2: Attach Permissions Policy

Create and attach this policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:GetObjectVersion",
        "s3:DeleteObject",
        "s3:DeleteObjectVersion"
      ],
      "Resource": "arn:aws:s3:::snowflake-tesla-data/Input/*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::snowflake-tesla-data",
      "Condition": {
        "StringLike": {
          "s3:prefix": ["Input/*"]
        }
      }
    }
  ]
}
```

### Step 8.3: Name and Create Role

- Role name: `SnowflakeAccessRole`
- Click **"Create role"**
- **Copy the Role ARN** (you'll need this)

Example ARN: `arn:aws:iam::123456789012:role/SnowflakeAccessRole`

### Step 8.4: Create Storage Integration in Snowflake

```sql
-- Grant privilege (if needed)
GRANT CREATE INTEGRATION ON ACCOUNT TO ROLE ACCOUNTADMIN;

-- Create storage integration
CREATE STORAGE INTEGRATION S3_INTEGRATION
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = S3
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::123456789012:role/SnowflakeAccessRole'
  ENABLED = TRUE
  STORAGE_ALLOWED_LOCATIONS = ('s3://snowflake-tesla-data/Input/');

-- Get Snowflake IAM user and external ID
DESC INTEGRATION S3_INTEGRATION;
```

**Copy these values** from the output:
- `STORAGE_AWS_IAM_USER_ARN`
- `STORAGE_AWS_EXTERNAL_ID`

### Step 8.5: Update IAM Trust Relationship

1. Go back to AWS IAM Console
2. Open `SnowflakeAccessRole`
3. Go to **"Trust relationships"** tab
4. Click **"Edit trust policy"**
5. Replace with:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "<STORAGE_AWS_IAM_USER_ARN>"
    },
    "Action": "sts:AssumeRole",
    "Condition": {
      "StringEquals": {
        "sts:ExternalId": "<STORAGE_AWS_EXTERNAL_ID>"
      }
    }
  }]
}
```
6. Click **"Update policy"**

### Step 8.6: Create File Format and Stage

```sql
-- Create file format
CREATE OR REPLACE FILE FORMAT CSV_FORMAT
  TYPE = 'CSV'
  FIELD_OPTIONALLY_ENCLOSED_BY = '"'
  SKIP_HEADER = 1
  NULL_IF = ('NULL', '')
  EMPTY_FIELD_AS_NULL = TRUE
  TRIM_SPACE = TRUE;

-- Create stage with storage integration
CREATE OR REPLACE STAGE TESLA_DATA_STAGE
  URL='s3://snowflake-tesla-data/Input/'
  STORAGE_INTEGRATION = S3_INTEGRATION
  FILE_FORMAT = CSV_FORMAT;

-- Test stage access
LIST @TESLA_DATA_STAGE;
```

### Step 8.7: Load Data Manually First (Test)

```sql
-- Load existing data
COPY INTO TESLA_DATA
  FROM @TESLA_DATA_STAGE
  PATTERN='.*\.csv';

-- Verify
SELECT COUNT(*) FROM TESLA_DATA;
```

### Step 8.8: Create Snowpipe

```sql
-- Create pipe for auto-ingestion
CREATE OR REPLACE PIPE tesla_pipe_test
  AUTO_INGEST = TRUE
AS
  COPY INTO TESLA_DATA
  FROM @TESLA_DATA_STAGE
  FILE_FORMAT = CSV_FORMAT;

-- Show pipe details
SHOW PIPES;
```

**Copy the `notification_channel` value** (SQS ARN)

Example: `arn:aws:sqs:us-east-1:123456789012:sf-snowpipe-AIDAI...`

### Step 8.9: Configure S3 Event Notification

1. Go to **AWS S3 Console**
2. Open your bucket: `snowflake-tesla-data`
3. Go to **Properties** tab
4. Scroll to **Event notifications**
5. Click **"Create event notification"**
6. Configuration:
   - **Name**: `SnowpipeTrigger`
   - **Event types**: 
     - ✅ All object create events
   - **Destination**: SQS queue
   - **SQS queue ARN**: Paste the `notification_channel` from Snowpipe
7. Click **"Save changes"**

### Step 8.10: Test Real-Time Ingestion

**Test the pipeline**:
1. Delete the existing `TSLA.csv` from S3 (or use a different file)
2. Re-upload `TSLA.csv` to S3 Input folder
3. Wait 30-60 seconds
4. Query Snowflake:

```sql
-- Check if new data loaded
SELECT COUNT(*) FROM TESLA_DATA;

-- Check pipe status
SELECT SYSTEM$PIPE_STATUS('tesla_pipe_test');

-- Check load history
SELECT *
FROM TABLE(INFORMATION_SCHEMA.COPY_HISTORY(
  TABLE_NAME => 'TESLA_DATA',
  START_TIME => DATEADD(hours, -1, CURRENT_TIMESTAMP())
));
```

**Success**: Data should load automatically without manual COPY command!

---

## 9. AWS QuickSight Visualization

### Step 9.1: Sign Up for QuickSight

1. Go to **AWS QuickSight Console**
2. Click **"Sign up for QuickSight"**
3. Edition: **Standard** (free trial)
4. Authentication: Use IAM
5. Region: Same as S3 bucket
6. Account name: `your-name-quicksight`
7. Email: Your email
8. Click **"Finish"**

### Step 9.2: Create Snowflake Connection

1. In QuickSight, click **"Datasets"**
2. Click **"New dataset"**
3. Search and select **"Snowflake"**
4. Connection details:
   - **Data source name**: `tesla_snowflake_data`
   - **Server**: `<account>.snowflakecomputing.com`
     - Find in Snowflake URL
     - Example: `xy12345.us-east-1.snowflakecomputing.com`
   - **Database**: `TEST_DB`
   - **Warehouse**: `COMPUTE_WH`
   - **Username**: Your Snowflake username
   - **Password**: Your Snowflake password
5. Click **"Validate connection"**
6. If successful, click **"Create data source"**

### Step 9.3: Select Table

1. **Schema**: PUBLIC
2. **Table**: TESLA_DATA
3. Click **"Select"**
4. Import: **"Directly query your data"** (for real-time)
5. Click **"Visualize"**

### Step 9.4: Create Visualizations

**Example 1: Stock Price Over Time (Line Chart)**
1. Visual type: **Line chart**
2. X-axis: `Date`
3. Value: `Close_value`
4. Title: "Tesla Stock Price Over Time"

**Example 2: Daily Volume (Bar Chart)**
1. Add new visual
2. Visual type: **Vertical bar chart**
3. X-axis: `Date`
4. Value: `volume`
5. Title: "Daily Trading Volume"

**Example 3: Price Range (Combo Chart)**
1. Add new visual
2. Visual type: **Combo chart**
3. X-axis: `Date`
4. Lines: `High_value`, `Low_value`
5. Bars: `volume`

### Step 9.5: Publish Dashboard

1. Click **"Share"** → **"Publish dashboard"**
2. Dashboard name: `Tesla Stock Analysis`
3. Click **"Publish"**

---

## 10. Testing Time Travel

### Step 10.1: Understand Time Travel

**Retention Period**:
- Standard Edition: 1 day
- Enterprise Edition: Up to 90 days

### Step 10.2: Test Drop and Restore

```sql
-- Check current data
SELECT COUNT(*) FROM TESLA_DATA;

-- Drop table
DROP TABLE TESLA_DATA;

-- Try to query (should fail)
SELECT * FROM TESLA_DATA;

-- Restore table
UNDROP TABLE TESLA_DATA;

-- Verify restoration
SELECT COUNT(*) FROM TESLA_DATA;
```

### Step 10.3: Test Historical Query

```sql
-- Make an update
UPDATE TESLA_DATA 
SET Open_value = 999 
WHERE Date = '2010-06-29';

-- Verify change
SELECT * FROM TESLA_DATA WHERE Date = '2010-06-29';

-- Query data BEFORE the update
SELECT * FROM TESLA_DATA 
  BEFORE (STATEMENT => '<statement-id>')
WHERE Date = '2010-06-29';
```

**To get statement ID**:
```sql
-- Check query history
SELECT query_id, query_text, start_time
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY())
WHERE query_text LIKE '%UPDATE TESLA_DATA%'
ORDER BY start_time DESC
LIMIT 5;
```

### Step 10.4: Create Historical Clone

```sql
-- Clone table as it was 1 hour ago
CREATE TABLE TESLA_DATA_BACKUP 
  CLONE TESLA_DATA 
  AT (OFFSET => -3600);

-- Compare current vs. backup
SELECT COUNT(*) FROM TESLA_DATA;
SELECT COUNT(*) FROM TESLA_DATA_BACKUP;
```

---

## 🎉 Congratulations!

You've successfully built a real-time data warehouse pipeline!

### What You've Accomplished:

✅ Created AWS S3 bucket for data storage  
✅ Set up Snowflake account and warehouse  
✅ Installed and configured SnowSQL CLI  
✅ Loaded data using three different methods  
✅ Implemented real-time ingestion with Snowpipe  
✅ Connected AWS QuickSight for visualization  
✅ Tested Snowflake Time Travel feature  

### Next Steps:

1. **Optimize**: Adjust warehouse size and auto-suspend settings
2. **Monitor**: Track credits usage and costs
3. **Expand**: Add more data sources and tables
4. **Automate**: Schedule regular data refreshes
5. **Share**: Create QuickSight dashboards for stakeholders

---

## 📞 Need Help?

- Check [Troubleshooting section](README.md#troubleshooting) in main README
- Review [Snowflake Documentation](https://docs.snowflake.com/)
- Consult [AWS Documentation](https://docs.aws.amazon.com/)
- Open an [issue on GitHub](https://github.com/abidaziz1/Snowflake-Projects/issues)

---

**Happy Data Engineering! 🚀**
