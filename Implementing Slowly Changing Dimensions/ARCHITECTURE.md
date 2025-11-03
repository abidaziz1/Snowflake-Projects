# Architecture Documentation

## Overview

This document provides a comprehensive technical overview of the Slowly Changing Dimensions (SCD) implementation using Snowflake, AWS, and Apache NiFi.

## Table of Contents

1. [System Architecture](#system-architecture)
2. [Component Details](#component-details)
3. [Data Flow](#data-flow)
4. [Database Schema](#database-schema)
5. [SCD Implementation](#scd-implementation)
6. [Infrastructure](#infrastructure)
7. [Security](#security)
8. [Scalability](#scalability)

---

## System Architecture

### High-Level Architecture
<img width="865" height="523" alt="image" src="https://github.com/user-attachments/assets/032bfe50-3f47-4185-bf76-e44d06c9c7af" />

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           AWS Cloud Environment                           │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │                      EC2 Instance (t2.xlarge)                       │  │
│  │  ┌──────────────────────────────────────────────────────────────┐  │  │
│  │  │              Docker Containerized Services                    │  │  │
│  │  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │  │  │
│  │  │  │  JupyterLab  │  │ Apache NiFi  │  │  Zookeeper   │       │  │  │
│  │  │  │              │  │              │  │              │       │  │  │
│  │  │  │ Port: 4888   │  │ Port: 2080   │  │ Port: 2181   │       │  │  │
│  │  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘       │  │  │
│  │  │         │                  │                  │               │  │  │
│  │  │         └──────────────────┴──────────────────┘               │  │  │
│  │  │                            │                                  │  │  │
│  │  │                   Shared Volume                               │  │  │
│  │  │              /opt/workspace (HDFS-like)                       │  │  │
│  │  └──────────────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────────────┘  │
│                                   │                                       │
│                                   ▼                                       │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │                    Amazon S3 Bucket (Data Lake)                    │  │
│  │                                                                    │  │
│  │    s3://scd-demo/customer_YYYYMMDDHHMMSS.csv                      │  │
│  │                                                                    │  │
│  │    Event Notification → SQS Queue                                 │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────┘
                                   │
                                   │ Snowpipe Auto-Ingest
                                   ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                        Snowflake Cloud Data Warehouse                     │
│                                                                           │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────┐                 │
│  │  External   │──▶│  Snowpipe    │──▶│ customer_raw │                 │
│  │   Stage     │   │ (Auto-load)  │   │   (Staging)  │                 │
│  └─────────────┘   └──────────────┘   └──────┬───────┘                 │
│                                               │                          │
│                                               ▼                          │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐                │
│  │   Stream     │──▶│    Task      │──▶│   customer   │                │
│  │ (CDC Track)  │   │ (Scheduler)  │   │   (Current)  │                │
│  └──────────────┘   └──────────────┘   └──────┬───────┘                │
│                                               │                          │
│                                               ▼                          │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐                │
│  │     View     │──▶│Stored Proc   │──▶│customer_     │                │
│  │ (SCD Logic)  │   │(Merge Logic) │   │   history    │                │
│  └──────────────┘   └──────────────┘   └──────────────┘                │
│                                                                           │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## Component Details

### 1. Data Generation Layer

**Technology**: JupyterLab + Python + Faker

**Purpose**: Generate synthetic customer data for testing and demonstration

**Components**:
- **Jupyter Notebook**: Interactive development environment
- **Faker Library**: Generates realistic fake data
- **CSV Writer**: Outputs data in structured format

**Key Features**:
- Generates 10,000 customer records per execution
- Timestamp-based file naming: `customer_YYYYMMDDHHMMSS.csv`
- Configurable data schema
- Shared volume for file access

**Output Schema**:
```python
{
    'customer_id': int,      # Unique identifier (0-9999)
    'first_name': string,    # Realistic first name
    'last_name': string,     # Realistic last name
    'email': string,         # Valid email format
    'street': string,        # Full street address
    'city': string,          # City name
    'state': string,         # State/Province
    'country': string        # Full country name
}
```

### 2. Orchestration Layer

**Technology**: Apache NiFi

**Purpose**: Automate data movement from local storage to S3

**Components**:

#### ListFile Processor
- **Function**: Monitor directory for new files
- **Configuration**:
  - Input Directory: `/opt/nifi/nifi-current/scd`
  - Polling Interval: Continuous
  - File Filter: `*.csv`

#### FetchFile Processor
- **Function**: Read file content
- **Configuration**:
  - Default settings
  - Handles large files efficiently

#### PutS3Object Processor
- **Function**: Upload files to S3
- **Configuration**:
  - Bucket: `scd-demo-{unique-id}`
  - Region: Configurable
  - Credentials: AWS Access Key/Secret

**Data Flow**:
```
ListFile → FetchFile → PutS3Object
```

### 3. Storage Layer

**Technology**: Amazon S3

**Purpose**: Cloud-based data lake for raw files

**Configuration**:
- **Bucket Policy**: Private (no public access)
- **Encryption**: Server-side (SSE-S3)
- **Versioning**: Disabled (can be enabled)
- **Lifecycle**: No automatic expiration
- **Event Notifications**: SQS queue for Snowpipe

**File Structure**:
```
s3://scd-demo/
├── customer_20250101120000.csv
├── customer_20250101130000.csv
├── customer_20250101140000.csv
└── ...
```

### 4. Processing Layer

**Technology**: Snowflake

**Purpose**: Data warehousing, transformation, and SCD implementation

**Key Components**:

#### External Stage
```sql
CREATE STAGE customer_ext_stage
  URL='s3://scd-demo/'
  CREDENTIALS=(...)
  FILE_FORMAT = csv;
```

#### Snowpipe (Auto-Ingestion)
```sql
CREATE PIPE customer_s3_pipe
  AUTO_INGEST = TRUE
AS
  COPY INTO customer_raw
  FROM @customer_ext_stage
  FILE_FORMAT = csv;
```

#### Stream (Change Data Capture)
```sql
CREATE STREAM customer_table_changes 
  ON TABLE customer;
```

#### Tasks (Scheduled Execution)
```sql
-- SCD Type 1
CREATE TASK tsk_scd_raw
  SCHEDULE = '1 MINUTE'
AS CALL pdr_scd_demo();

-- SCD Type 2
CREATE TASK tsk_scd_hist
  SCHEDULE = '1 MINUTE'
AS MERGE INTO customer_history ...;
```

---

## Data Flow

### End-to-End Data Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Data Generation                              │
│                                                                      │
│  1. User executes Faker.ipynb in JupyterLab                        │
│  2. Python script generates 10,000 customer records                 │
│  3. CSV file saved to /opt/workspace/nifi/FakeDataset/             │
│     Filename: customer_20250101120000.csv                           │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       File Transfer (Manual)                         │
│                                                                      │
│  4. User copies file from Jupyter to NiFi workspace                 │
│     cp /opt/workspace/.../customer_*.csv /opt/nifi/.../scd/         │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       NiFi Processing                                │
│                                                                      │
│  5. ListFile processor detects new CSV file                         │
│  6. FetchFile reads file content (10,000 rows)                      │
│  7. PutS3Object uploads to S3 bucket                                │
│     → S3 Event Notification triggered                               │
│     → SQS message sent to Snowpipe                                  │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    Snowpipe Auto-Ingestion                           │
│                                                                      │
│  8. Snowpipe receives SQS notification                              │
│  9. Executes COPY INTO customer_raw                                 │
│  10. 10,000 records loaded into staging table                       │
│      Duration: ~10-30 seconds                                       │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      SCD Type 1 Processing                           │
│                                                                      │
│  11. Task tsk_scd_raw runs every 1 minute                           │
│  12. Stored procedure pdr_scd_demo() executes                       │
│  13. MERGE operation:                                               │
│      - Existing customers: UPDATE if changes detected               │
│      - New customers: INSERT new records                            │
│  14. customer_raw truncated after processing                        │
│  15. customer table updated with latest data                        │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      SCD Type 2 Processing                           │
│                                                                      │
│  16. Stream customer_table_changes captures DML                     │
│  17. View v_customer_change_data classifies changes:                │
│      - 'I' (Insert): New customer or new version                    │
│      - 'U' (Update): Close old version, mark inactive               │
│      - 'D' (Delete): Logical deletion with end_time                 │
│  18. Task tsk_scd_hist runs every 1 minute                          │
│  19. MERGE into customer_history:                                   │
│      - INSERT new versions                                          │
│      - UPDATE old versions (set end_time, is_current=FALSE)         │
│  20. Complete historical audit trail maintained                     │
└─────────────────────────────┬───────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      Analytics Ready                                 │
│                                                                      │
│  ✓ Current state: customer table                                    │
│  ✓ Historical state: customer_history table                         │
│  ✓ Query both for comprehensive analysis                            │
│  ✓ Audit trail for compliance                                       │
│  ✓ Trend analysis capabilities                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Database Schema

### Table Definitions

#### customer_raw (Staging)
```sql
CREATE TABLE customer_raw (
  customer_id     NUMBER,
  first_name      VARCHAR,
  last_name       VARCHAR,
  email           VARCHAR,
  street          VARCHAR,
  city            VARCHAR,
  state           VARCHAR,
  country         VARCHAR
);
```

**Purpose**: Temporary landing zone for raw data from S3

**Characteristics**:
- No primary key (staging area)
- No constraints
- Truncated after processing
- High ingestion throughput

#### customer (Current Dimension)
```sql
CREATE TABLE customer (
  customer_id        NUMBER,
  first_name         VARCHAR,
  last_name          VARCHAR,
  email              VARCHAR,
  street             VARCHAR,
  city               VARCHAR,
  state              VARCHAR,
  country            VARCHAR,
  update_timestamp   TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP()
);
```

**Purpose**: Maintain current state of each customer (SCD Type 1)

**Characteristics**:
- One row per customer
- Latest information only
- No historical tracking
- Fast queries for current state

#### customer_history (Historical Dimension)
```sql
CREATE TABLE customer_history (
  customer_id     NUMBER,
  first_name      VARCHAR,
  last_name       VARCHAR,
  email           VARCHAR,
  street          VARCHAR,
  city            VARCHAR,
  state           VARCHAR,
  country         VARCHAR,
  start_time      TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
  end_time        TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
  is_current      BOOLEAN
);
```

**Purpose**: Preserve complete change history (SCD Type 2)

**Characteristics**:
- Multiple rows per customer (versions)
- Complete audit trail
- start_time: When version became active
- end_time: When version was superseded ('9999-12-31' for current)
- is_current: Boolean flag (TRUE for active version)

### Entity Relationship

```
┌─────────────────┐
│  customer_raw   │  (Staging - Temporary)
│─────────────────│
│ customer_id     │
│ first_name      │
│ last_name       │
│ email           │
│ street          │
│ city            │
│ state           │
│ country         │
└────────┬────────┘
         │ MERGE
         ▼
┌─────────────────┐
│    customer     │  (Current State - SCD Type 1)
│─────────────────│
│ customer_id     │◀─┐
│ first_name      │  │
│ last_name       │  │
│ email           │  │ Stream captures
│ street          │  │ changes
│ city            │  │
│ state           │  │
│ country         │  │
│ update_timestamp│  │
└────────┬────────┘  │
         │           │
         └───────────┘
         │ MERGE (with history logic)
         ▼
┌─────────────────┐
│customer_history │  (Historical State - SCD Type 2)
│─────────────────│
│ customer_id     │
│ first_name      │
│ last_name       │
│ email           │
│ street          │
│ city            │
│ state           │
│ country         │
│ start_time      │
│ end_time        │
│ is_current      │
└─────────────────┘
```

---

## SCD Implementation

### Type 1: Overwrite (Current State)

**Logic**: Replace old values with new values

**Use Cases**:
- Correcting data entry errors
- Updating non-critical information
- Real-time dashboards requiring current state

**Implementation**:
```sql
MERGE INTO customer c
USING customer_raw cr
  ON c.customer_id = cr.customer_id
WHEN MATCHED AND (
  c.first_name <> cr.first_name OR
  c.email <> cr.email OR
  c.city <> cr.city
) THEN UPDATE SET
  c.first_name = cr.first_name,
  c.email = cr.email,
  c.city = cr.city,
  c.update_timestamp = CURRENT_TIMESTAMP()
WHEN NOT MATCHED THEN INSERT (...)
  VALUES (...);
```

**Result**:
- One row per customer
- No historical data
- Minimal storage overhead

### Type 2: Historical Tracking

**Logic**: Create new version, close old version

**Use Cases**:
- Audit requirements
- Compliance tracking
- Trend analysis
- "As of" reporting

**Implementation**:
```sql
-- View classifies changes
CREATE VIEW v_customer_change_data AS
  SELECT *, 
    CASE 
      WHEN metadata$action = 'INSERT' THEN 'I'
      WHEN metadata$action = 'DELETE' AND metadata$isupdate = 'TRUE' THEN 'U'
      WHEN metadata$action = 'DELETE' THEN 'D'
    END as dml_type
  FROM customer_table_changes;

-- Task merges into history
MERGE INTO customer_history ch
USING v_customer_change_data ccd
  ON ch.customer_id = ccd.customer_id 
  AND ch.start_time = ccd.start_time
WHEN MATCHED AND ccd.dml_type = 'U' THEN
  UPDATE SET 
    ch.end_time = ccd.end_time,
    ch.is_current = FALSE
WHEN NOT MATCHED AND ccd.dml_type = 'I' THEN
  INSERT (...) VALUES (...);
```

**Result**:
- Multiple rows per customer (versions)
- Complete change history
- start_time/end_time for temporal queries
- is_current flag for active version

---

## Infrastructure

### AWS Components

#### EC2 Instance
- **Instance Type**: t2.xlarge
- **vCPUs**: 4
- **Memory**: 16 GB
- **Storage**: 30 GB gp3 SSD
- **OS**: Amazon Linux 2023
- **Networking**: VPC with public IP

#### S3 Bucket
- **Storage Class**: STANDARD
- **Encryption**: SSE-S3
- **Versioning**: Optional
- **Event Notifications**: Enabled for Snowpipe

### Docker Architecture

```yaml
version: "3.6"

services:
  jupyterlab:
    image: pavansrivathsa/jupyterlab
    ports: ["4888:4888"]
    volumes: [shared-workspace:/opt/workspace]
    
  nifi:
    image: pavansrivathsa/nifi
    ports: ["2080:2080"]
    volumes: [shared-workspace:/opt/workspace/nifi]
    environment:
      - NIFI_WEB_HTTP_PORT=2080
      - NIFI_ZK_CONNECT_STRING=zookeeper:2181
    
  zookeeper:
    image: bitnami/zookeeper
    ports: ["2181:2181"]
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes

volumes:
  shared-workspace:
    name: hadoop-distributed-file-system
```

### Snowflake Architecture

**Components**:
- **Virtual Warehouse**: COMPUTE_WH (SMALL)
- **Database**: SCD_DEMO
- **Schema**: scd2
- **Storage**: Cloud storage (S3-backed)

**Features Utilized**:
- External Stages
- Snowpipe (Auto-Ingest)
- Streams (CDC)
- Tasks (Scheduling)
- Stored Procedures
- Views

---

## Security

### Authentication & Authorization

#### AWS
- IAM user with programmatic access
- Least privilege policy
- Access keys rotated regularly
- MFA enabled on root account

#### Snowflake
- ACCOUNTADMIN role for setup
- Service accounts for automation
- Role-based access control (RBAC)
- Network policy restrictions

### Data Protection

#### In Transit
- SSL/TLS for all connections
- SSH for EC2 access
- HTTPS for S3 upload

#### At Rest
- S3 server-side encryption
- Snowflake encryption (always on)
- Docker volume encryption (optional)

### Credentials Management

**Best Practices**:
- No hardcoded credentials in code
- Use environment variables
- AWS Secrets Manager (recommended)
- Rotate keys regularly

**Never Commit**:
- .pem files
- AWS access keys
- Snowflake passwords
- Connection strings with credentials

---

## Scalability

### Horizontal Scaling

#### Data Generation
- Multiple Jupyter instances
- Parallel CSV generation
- Distributed file storage

#### NiFi Processing
- NiFi cluster mode
- Load balancing
- Multiple input directories

#### Snowflake
- Virtual warehouse clustering
- Multi-cluster warehouses
- Automatic scaling (Enterprise)

### Vertical Scaling

#### EC2
- Upgrade to larger instance types
- t2.xlarge → t2.2xlarge → c5.4xlarge
- Add more storage

#### Snowflake
- Increase warehouse size
- SMALL → MEDIUM → LARGE → X-LARGE
- Pay for what you use

### Performance Optimization

#### NiFi
- Buffer sizing
- Concurrent tasks
- Back pressure settings

#### Snowflake
- Clustering keys
- Materialized views
- Result caching
- Warehouse auto-suspend/resume

---

## Monitoring & Observability

### Key Metrics

#### Infrastructure
- EC2 CPU/Memory utilization
- Docker container health
- Network throughput

#### Data Pipeline
- Files processed per hour
- Average processing time
- Error rate
- Queue depth

#### Snowflake
- Credits consumed
- Query performance
- Storage growth
- Task execution history

### Monitoring Tools

**AWS CloudWatch**:
- EC2 metrics
- S3 access logs
- Custom metrics

**Snowflake**:
```sql
-- Task history
SELECT * FROM TABLE(information_schema.task_history());

-- Pipe status
SELECT SYSTEM$PIPE_STATUS('customer_s3_pipe');

-- Query history
SELECT * FROM TABLE(information_schema.query_history());
```

---

## Disaster Recovery

### Backup Strategy

#### Data
- S3 versioning enabled
- Cross-region replication
- Snowflake Time Travel (90 days)
- Regular snapshots

#### Configuration
- Infrastructure as Code (IaC)
- Docker Compose files version-controlled
- SQL scripts in Git repository

### Recovery Procedures

1. **EC2 Failure**: Launch new instance, restore Docker setup
2. **S3 Data Loss**: Restore from versioning or backup
3. **Snowflake Issues**: Use Time Travel, Fail-safe
4. **Network Partition**: Retry mechanisms, queue buffering

---

## Cost Optimization

### AWS
- Use Spot Instances for non-critical workloads
- Right-size EC2 instances
- S3 Intelligent-Tiering
- Delete old data with lifecycle policies

### Snowflake
- Auto-suspend warehouses (5 minutes)
- Auto-resume on demand
- Monitor credit usage
- Use query result caching
- Optimize table clustering

### Monitoring Costs
```sql
-- Snowflake credit usage
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY;

-- Storage usage
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.STORAGE_USAGE;
```

---

**For questions or clarifications, please refer to the main README or open an issue on GitHub.**
