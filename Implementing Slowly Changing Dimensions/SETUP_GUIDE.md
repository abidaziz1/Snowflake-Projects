# Complete Setup Guide: Implementing SCD with Snowflake

This comprehensive guide will walk you through every step of setting up the Slowly Changing Dimensions pipeline from scratch.

## 📑 Table of Contents

1. [Prerequisites Checklist](#prerequisites-checklist)
2. [AWS Account Setup](#aws-account-setup)
3. [Snowflake Account Setup](#snowflake-account-setup)
4. [Creating AWS S3 Bucket](#creating-aws-s3-bucket)
5. [Setting Up EC2 Instance](#setting-up-ec2-instance)
6. [Docker Installation](#docker-installation)
7. [Deploying Docker Containers](#deploying-docker-containers)
8. [Configuring Apache NiFi](#configuring-apache-nifi)
9. [Snowflake Configuration](#snowflake-configuration)
10. [Testing the Pipeline](#testing-the-pipeline)

---

## 1. Prerequisites Checklist

Before starting, ensure you have:

- [ ] AWS account with billing enabled
- [ ] Snowflake trial or paid account
- [ ] Local machine with SSH client
- [ ] Terminal/Command Prompt access
- [ ] Web browser (Chrome/Firefox recommended)
- [ ] Basic understanding of:
  - SQL queries
  - Command line operations
  - Docker basics
  - Cloud computing concepts

### Estimated Setup Time
- **First-time setup**: 2-3 hours
- **With prior experience**: 45-60 minutes

### Estimated Monthly Costs
- **AWS EC2 (t2.xlarge)**: ~$120/month (free tier eligible for new accounts)
- **AWS S3**: ~$1-5/month for small datasets
- **Snowflake**: Free trial includes $400 credit

---

## 2. AWS Account Setup

### Step 2.1: Create AWS Account

1. Visit [aws.amazon.com](https://aws.amazon.com)
2. Click **"Create an AWS Account"**
3. Follow the registration process:
   - Provide email address
   - Set up payment method
   - Verify phone number
   - Choose support plan (Basic/Free is sufficient)

### Step 2.2: Set Up Billing Alerts

⚠️ **Critical Step**: Prevent unexpected charges

1. Navigate to **AWS Billing Console**
2. Click **"Billing preferences"**
3. Enable:
   - **Receive Free Tier Usage Alerts**
   - **Receive Billing Alerts**
4. Set up **Budget**:
   - Go to **AWS Budgets**
   - Create a monthly cost budget (e.g., $50)
   - Configure email alerts at 80% and 100%

### Step 2.3: Create IAM User (Recommended)

Instead of using root account:

1. Go to **IAM Console**
2. Click **"Users"** → **"Add User"**
3. Username: `snowflake-scd-user`
4. Enable **"Programmatic access"**
5. Attach policies:
   - `AmazonS3FullAccess`
   - `AmazonEC2FullAccess`
6. **Save credentials securely**:
   - Access Key ID
   - Secret Access Key

---

## 3. Snowflake Account Setup

### Step 3.1: Create Snowflake Account

1. Visit [signup.snowflake.com](https://signup.snowflake.com)
2. Choose:
   - **Cloud Provider**: AWS
   - **Region**: Same as your EC2 region (e.g., us-east-1)
   - **Edition**: Standard (for trial)
3. Complete registration

### Step 3.2: Initial Snowflake Configuration

```sql
-- Login to Snowflake Web UI
-- Use ACCOUNTADMIN role

-- Create warehouse (if not exists)
CREATE WAREHOUSE IF NOT EXISTS COMPUTE_WH
  WAREHOUSE_SIZE = 'SMALL'
  AUTO_SUSPEND = 300
  AUTO_RESUME = TRUE
  INITIALLY_SUSPENDED = TRUE;

-- Resume warehouse
ALTER WAREHOUSE COMPUTE_WH RESUME;
```

---

## 4. Creating AWS S3 Bucket

### Step 4.1: Create Bucket

1. Open **AWS Console** → Search for **"S3"**
2. Click **"Create bucket"**
3. Bucket Configuration:
   - **Bucket name**: `scd-demo-<your-unique-id>` (must be globally unique)
   - **Region**: Same as EC2 (e.g., us-east-1)
   - **Block Public Access**: Keep all boxes checked ✅
   - **Versioning**: Disabled (optional)
   - **Encryption**: Enable (default)
4. Click **"Create bucket"**

### Step 4.2: Verify Bucket

```bash
# List buckets using AWS CLI (optional)
aws s3 ls

# Should show your new bucket
```

---

## 5. Setting Up EC2 Instance

### Step 5.1: Launch EC2 Instance

1. Navigate to **EC2 Console**
2. Click **"Launch Instance"**
3. Configuration:

   **Name**: `snowflake-scd-pipeline`
   
   **Application and OS Images**:
   - AMI: **Amazon Linux 2023** (or Amazon Linux 2)
   
   **Instance Type**:
   - Select: **t2.xlarge**
   - vCPUs: 4, Memory: 16 GB
   
   **Key Pair**:
   - Click **"Create new key pair"**
   - Name: `snowflake-scd-key`
   - Type: **RSA**
   - Format: **.pem** (for Mac/Linux) or **.ppk** (for Windows/PuTTY)
   - Download and **save securely**
   
   **Network Settings**:
   - Allow SSH traffic from: **My IP** (recommended)
   - Or: **Anywhere** (less secure)
   
   **Configure Storage**:
   - Size: **30 GB** (recommended)
   - Volume Type: **gp3**

4. Click **"Launch Instance"**

### Step 5.2: Configure Security Group

1. Go to **EC2 Dashboard** → **Security Groups**
2. Find security group for your instance
3. Click **"Edit inbound rules"**
4. Add rules:

   | Type | Protocol | Port Range | Source |
   |------|----------|------------|--------|
   | SSH | TCP | 22 | My IP |
   | Custom TCP | TCP | 2080 | My IP |
   | Custom TCP | TCP | 4888 | My IP |
   | Custom TCP | TCP | 8050 | My IP |
   | Custom TCP | TCP | 4000-38888 | My IP |

5. Click **"Save rules"**

### Step 5.3: Connect to EC2

```bash
# Change key permissions (first time only)
chmod 400 snowflake-scd-key.pem

# SSH into instance
ssh -i "snowflake-scd-key.pem" ec2-user@<your-ec2-public-ip>

# You should see Amazon Linux prompt
```

---

## 6. Docker Installation

### Step 6.1: Install Docker

On your EC2 instance:

```bash
# Update system packages
sudo yum update -y

# Install Docker
sudo yum install docker -y

# Start Docker service
sudo systemctl start docker

# Enable Docker to start on boot
sudo systemctl enable docker

# Verify Docker installation
docker --version
```

### Step 6.2: Install Docker Compose

```bash
# Download Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make it executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker-compose --version
```

### Step 6.3: Configure Docker Permissions

```bash
# Add current user to docker group
sudo usermod -aG docker ec2-user

# Apply group changes
newgrp docker

# Test Docker without sudo
docker ps
```

---

## 7. Deploying Docker Containers

### Step 7.1: Prepare Directory Structure

```bash
# Create project directory
mkdir -p ~/docker_exp
cd ~/docker_exp
```

### Step 7.2: Copy docker-compose.yml

**From your local machine:**

```bash
# Navigate to project directory
cd /path/to/Implementing-Slowly-Changing-Dimensions

# Copy file to EC2
scp -i "snowflake-scd-key.pem" code/docker-compose.yml ec2-user@<your-ec2-ip>:~/docker_exp/
```

### Step 7.3: Start Docker Services

**On EC2 instance:**

```bash
cd ~/docker_exp

# Pull images and start containers
docker-compose up -d

# Verify containers are running
docker ps

# You should see:
# - jupyterlab
# - nifi
# - zookeeper
```

### Step 7.4: Set Up Port Forwarding

**On your local machine (new terminal):**

```bash
ssh -i "snowflake-scd-key.pem" ec2-user@<your-ec2-ip> \
  -L 4888:localhost:4888 \
  -L 2080:localhost:2080 \
  -L 8050:localhost:8050 \
  -L 4141:localhost:4141
```

Keep this terminal open during your work!

### Step 7.5: Access Services

1. **JupyterLab**: Open browser → `http://localhost:4888`
2. **Apache NiFi**: Open browser → `http://localhost:2080`

---

## 8. Configuring Apache NiFi

### Step 8.1: Copy Data Files

**SSH into EC2:**

```bash
# Get NiFi container ID
docker ps

# Access NiFi container
docker exec -it <nifi-container-id> bash

# Inside container, create directory
mkdir -p /opt/nifi/nifi-current/scd

# Exit container
exit
```

### Step 8.2: Create NiFi Flow

1. Open NiFi at `localhost:2080`
2. **Create Process Group**:
   - Drag "Process Group" icon to canvas
   - Name: `SCD-S3-Upload`
   - Double-click to enter

3. **Add ListFile Processor**:
   - Drag processor icon
   - Search: `ListFile`
   - Configure Properties:
     - Input Directory: `/opt/nifi/nifi-current/scd`
   - Settings → Auto-terminate: `failure`

4. **Add FetchFile Processor**:
   - Drag processor icon
   - Search: `FetchFile`
   - Leave default properties
   - Settings → Auto-terminate: `success`, `not.found`, `permission.denied`

5. **Add PutS3Object Processor**:
   - Drag processor icon
   - Search: `PutS3Object`
   - Configure Properties:
     - Bucket: `scd-demo-<your-unique-id>`
     - Access Key ID: `<your-aws-access-key>`
     - Secret Access Key: `<your-aws-secret-key>`
     - Region: `us-east-1` (or your region)
   - Settings → Auto-terminate: `success`, `failure`

6. **Connect Processors**:
   - ListFile → FetchFile (relationship: `success`)
   - FetchFile → PutS3Object (relationship: `success`)

7. **Start Process Group**:
   - Click "Start" button

---

## 9. Snowflake Configuration

### Step 9.1: Create Database and Schema

```sql
-- Create database
CREATE DATABASE SCD_DEMO;

-- Use database
USE DATABASE SCD_DEMO;

-- Create schema
CREATE SCHEMA scd2;

-- Set context
USE SCHEMA scd2;
```

### Step 9.2: Create Tables

```sql
-- Staging table for raw data
CREATE OR REPLACE TABLE customer_raw (
  customer_id NUMBER,
  first_name VARCHAR,
  last_name VARCHAR,
  email VARCHAR,
  street VARCHAR,
  city VARCHAR,
  state VARCHAR,
  country VARCHAR
);

-- Current dimension table (SCD Type 1)
CREATE OR REPLACE TABLE customer (
  customer_id NUMBER,
  first_name VARCHAR,
  last_name VARCHAR,
  email VARCHAR,
  street VARCHAR,
  city VARCHAR,
  state VARCHAR,
  country VARCHAR,
  update_timestamp TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP()
);

-- Historical dimension table (SCD Type 2)
CREATE OR REPLACE TABLE customer_history (
  customer_id NUMBER,
  first_name VARCHAR,
  last_name VARCHAR,
  email VARCHAR,
  street VARCHAR,
  city VARCHAR,
  state VARCHAR,
  country VARCHAR,
  start_time TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
  end_time TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
  is_current BOOLEAN
);
```

### Step 9.3: Create File Format

```sql
CREATE OR REPLACE FILE FORMAT csv
  TYPE = 'CSV'
  FIELD_OPTIONALLY_ENCLOSED_BY = '"'
  SKIP_HEADER = 1
  NULL_IF = ('NULL', '')
  FIELD_DELIMITER = ',';
```

### Step 9.4: Create External Stage

```sql
CREATE OR REPLACE STAGE customer_ext_stage
  URL='s3://scd-demo-<your-unique-id>/'
  CREDENTIALS=(
    AWS_KEY_ID='<your-aws-access-key-id>'
    AWS_SECRET_KEY='<your-aws-secret-key>'
  )
  FILE_FORMAT = csv;

-- Test stage
LIST @customer_ext_stage;
```

### Step 9.5: Create Snowpipe

```sql
CREATE OR REPLACE PIPE customer_s3_pipe
  AUTO_INGEST = TRUE
AS
  COPY INTO customer_raw
  FROM @customer_ext_stage
  FILE_FORMAT = csv;

-- Get notification channel ARN (save this!)
SHOW PIPES;
-- Copy the "notification_channel" value
```

### Step 9.6: Configure S3 Event Notification

1. Open **S3 Console** → Your bucket
2. Go to **Properties** → **Event notifications**
3. Click **"Create event notification"**
4. Configuration:
   - Name: `snowpipe-trigger-event`
   - Event types: **All object create events**
   - Destination: **SQS Queue**
   - SQS Queue ARN: Paste from Snowpipe notification_channel
5. Click **"Save changes"**

### Step 9.7: Create Stream

```sql
CREATE OR REPLACE STREAM customer_table_changes 
  ON TABLE customer;
```

### Step 9.8: Create View (SCD Type 2 Logic)

```sql
-- See code/sql/scd_type-2.sql for full view definition
CREATE OR REPLACE VIEW v_customer_change_data AS
  -- INSERT handling
  SELECT 
    CUSTOMER_ID, FIRST_NAME, LAST_NAME, EMAIL, 
    STREET, CITY, STATE, COUNTRY,
    start_time, end_time, is_current, 
    'I' as dml_type
  FROM (
    -- View logic for detecting inserts, updates, deletes
    ...
  );
```

### Step 9.9: Create Stored Procedures

**SCD Type 1 Procedure:**

```sql
CREATE OR REPLACE PROCEDURE pdr_scd_demo()
RETURNS STRING NOT NULL
LANGUAGE JAVASCRIPT
AS
$$
  var cmd = `
    MERGE INTO customer c
    USING customer_raw cr
    ON c.customer_id = cr.customer_id
    WHEN MATCHED AND (
      c.first_name <> cr.first_name OR
      c.last_name <> cr.last_name OR
      ...
    ) THEN UPDATE SET
      c.first_name = cr.first_name,
      c.last_name = cr.last_name,
      ...
    WHEN NOT MATCHED THEN INSERT ...
  `;
  
  var cmd1 = "TRUNCATE TABLE SCD_DEMO.SCD2.customer_raw;";
  
  var sql = snowflake.createStatement({ sqlText: cmd });
  var sql1 = snowflake.createStatement({ sqlText: cmd1 });
  
  sql.execute();
  sql1.execute();
  
  return cmd + '\n' + cmd1;
$$;
```

### Step 9.10: Create Tasks

**SCD Type 1 Task:**

```sql
CREATE OR REPLACE TASK tsk_scd_raw
  WAREHOUSE = COMPUTE_WH
  SCHEDULE = '1 MINUTE'
  ERROR_ON_NONDETERMINISTIC_MERGE = FALSE
AS
  CALL pdr_scd_demo();
```

**SCD Type 2 Task:**

```sql
CREATE OR REPLACE TASK tsk_scd_hist
  WAREHOUSE = COMPUTE_WH
  SCHEDULE = '1 MINUTE'
  ERROR_ON_NONDETERMINISTIC_MERGE = FALSE
AS
  MERGE INTO customer_history ch
  USING v_customer_change_data ccd
  ON ch.CUSTOMER_ID = ccd.CUSTOMER_ID
    AND ch.start_time = ccd.start_time
  WHEN MATCHED AND ccd.dml_type = 'U' THEN UPDATE
    SET ch.end_time = ccd.end_time,
        ch.is_current = FALSE
  -- Additional merge logic...
```

---

## 10. Testing the Pipeline

### Step 10.1: Generate Test Data

1. Open JupyterLab at `localhost:4888`
2. Create/upload `Faker.ipynb`
3. Run cells to generate customer data
4. Verify CSV created in `FakeDataset/` directory

### Step 10.2: Copy Data to NiFi

**On EC2:**

```bash
# Access JupyterLab container
docker exec -it <jupyter-container-id> bash

# Copy file to NiFi shared directory
cp /opt/workspace/nifi/FakeDataset/customer_*.csv /opt/nifi/nifi-current/scd/

# Exit
exit
```

### Step 10.3: Monitor Data Flow

1. **NiFi**: Check processor statistics
2. **S3**: Verify file appears in bucket
3. **Snowflake**: Check data loaded

```sql
-- Check raw table
SELECT COUNT(*) FROM customer_raw;

-- Should show number of records loaded
```

### Step 10.4: Test SCD Type 1

```sql
-- Resume task
ALTER TASK tsk_scd_raw RESUME;

-- Wait 1-2 minutes

-- Check customer table
SELECT COUNT(*) FROM customer;

-- Verify raw table is empty (truncated)
SELECT COUNT(*) FROM customer_raw;
```

### Step 10.5: Test SCD Type 2

```sql
-- Resume task
ALTER TASK tsk_scd_hist RESUME;

-- Make a test update
UPDATE customer 
SET first_name = 'TestUpdate' 
WHERE customer_id = 1;

-- Wait 1-2 minutes

-- Check history
SELECT * FROM customer_history 
WHERE customer_id = 1 
ORDER BY start_time DESC;

-- Should see old and new versions
```

---

## 🎉 Congratulations!

Your SCD pipeline is now fully operational!

### Next Steps:

1. **Monitor Performance**: Check task execution history
2. **Optimize**: Adjust task schedules based on data volume
3. **Scale**: Increase warehouse size for larger datasets
4. **Experiment**: Try different SCD scenarios
5. **Automate**: Schedule regular data generation

---

## 📞 Need Help?

- Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- Review [AWS EC2 documentation](https://docs.aws.amazon.com/ec2/)
- Consult [Snowflake documentation](https://docs.snowflake.com/)
- Open an [issue on GitHub](https://github.com/abidaziz1/Snowflake-Projects/issues)

---

**Happy Data Engineering! 🚀**
