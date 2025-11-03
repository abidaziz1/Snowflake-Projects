# Implementing Slowly Changing Dimensions using Snowflake

[![Snowflake](https://img.shields.io/badge/Snowflake-29B5E8?style=for-the-badge&logo=snowflake&logoColor=white)](https://www.snowflake.com/)
[![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Apache NiFi](https://img.shields.io/badge/Apache%20NiFi-017CEE?style=for-the-badge&logo=apache&logoColor=white)](https://nifi.apache.org/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)

## 📋 Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Technologies Used](#technologies-used)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Installation & Setup](#installation--setup)
- [Usage](#usage)
- [SCD Implementation](#scd-implementation)
- [Data Flow](#data-flow)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

This project demonstrates a comprehensive end-to-end data pipeline that implements **Slowly Changing Dimensions (SCD)** Types 1 and 2 using **Snowflake** as the cloud data warehouse. The pipeline simulates real-world data warehousing scenarios involving dimensional modeling and historical data tracking.

The solution leverages modern data engineering tools including Python, Apache NiFi, Docker, AWS S3, AWS EC2, and Snowflake to build a robust, automated, and scalable data pipeline capable of handling evolving customer data while maintaining complete historical accuracy.

### 🎪 Use Cases

1. **Customer Address History for Fraud Detection & Compliance Audits**
   - Track complete address change history for KYC and AML compliance
   - Flag suspicious activities based on frequent address changes
   - Maintain audit trail for regulatory requirements

2. **Marketing Campaign Optimization Based on Evolving Customer Demographics**
   - Preserve historical customer preferences and behavior patterns
   - Enable time-relevant, location-specific promotions
   - Improve customer segmentation and engagement

3. **Dynamic Customer Profile Management in Subscription-Based Platform**
   - Real-time profile updates with historical tracking
   - Support billing analytics and customer lifecycle analysis
   - Enable accurate monthly revenue reporting

## 🏗️ Architecture

The project implements a modern cloud-native ETL pipeline with the following components:

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────┐     ┌──────────────┐
│   Jupyter Lab   │────▶│ Apache NiFi  │────▶│   AWS S3    │────▶│  Snowflake   │
│ (Data Gen)      │     │ (ETL Tool)   │     │ (Storage)   │     │ (DW + SCD)   │
└─────────────────┘     └──────────────┘     └─────────────┘     └──────────────┘
        │                      │                     │                    │
        └──────────────────────┴─────────────────────┴────────────────────┘
                         Containerized on AWS EC2
```

### Architecture Components:

1. **Data Generation Layer**: Jupyter Notebook with Faker library generates synthetic customer data
2. **Orchestration Layer**: Apache NiFi automates data flow and movement to S3
3. **Storage Layer**: Amazon S3 acts as the data lake for raw CSV files
4. **Processing Layer**: Snowflake performs SCD transformations and historical tracking
5. **Automation Layer**: Snowpipe, Streams, Tasks, and Stored Procedures handle real-time ingestion

## ✨ Features

### Core Capabilities

- ✅ **Automated End-to-End ETL Pipeline**: From data generation to Snowflake storage
- ✅ **SCD Type 1 & Type 2 Implementation**: Handle both overwrite and historical tracking patterns
- ✅ **Real-Time Data Ingestion**: Using Snowpipe with S3 event notifications
- ✅ **Hybrid Cloud Integration**: Seamless AWS and Snowflake integration
- ✅ **Containerized Deployment**: Docker-based services for consistency and portability
- ✅ **Schema-Driven Data Handling**: Dimensional modeling with fact and dimension tables
- ✅ **Change Data Capture**: Snowflake Streams for tracking table changes
- ✅ **Automated Scheduling**: Snowflake Tasks for periodic SCD logic execution

### Technical Highlights

- **Dimensional Modeling**: Proper separation of facts and dimensions
- **Historical Tracking**: Complete audit trail with start_time, end_time, and is_current flags
- **Scalable Architecture**: Can handle both streaming and batch processing workloads
- **Data Quality**: Built-in validation and error handling mechanisms

## 🛠️ Technologies Used

| Technology | Purpose | Version |
|-----------|---------|---------|
| **Snowflake** | Cloud Data Warehouse | Latest |
| **AWS S3** | Object Storage | - |
| **AWS EC2** | Compute Instance | Amazon Linux |
| **Apache NiFi** | Data Orchestration | - |
| **Docker** | Containerization | Latest |
| **Docker Compose** | Multi-container Management | v3.6 |
| **Python** | Data Generation & Processing | 3.x |
| **Faker** | Synthetic Data Generation | Latest |
| **JupyterLab** | Interactive Development | Latest |

## 📦 Prerequisites

### Required Accounts & Access

1. **AWS Account** with the following:
   - EC2 instance creation permissions
   - S3 bucket creation and management
   - IAM user with programmatic access
   - Access Key ID and Secret Access Key

2. **Snowflake Account**
   - Standard or Enterprise edition
   - ACCOUNTADMIN or equivalent role
   - Virtual warehouse access

### Local Requirements

- SSH client (OpenSSH, PuTTY, or similar)
- Web browser (Chrome, Firefox, or Edge)
- Terminal/Command Prompt access
- Basic knowledge of:
  - SQL
  - Docker
  - AWS services
  - Command line operations

### Cost Considerations

⚠️ **Important**: This project uses cloud resources that may incur costs:
- AWS EC2 instance (t2.xlarge recommended)
- AWS S3 storage and data transfer
- Snowflake compute and storage
- Monitor your spending and set up billing alerts

## 📁 Project Structure

```
Implementing-SCD-Snowflake/
├── README.md                       # Main documentation
├── ARCHITECTURE.md                 # Detailed architecture documentation
├── SETUP_GUIDE.md                  # Step-by-step setup instructions
├── LICENSE                         # Project license
│
├── code/
│   ├── docker-compose.yml         # Docker services configuration
│   ├── faker/
│   │   └── Faker.ipynb           # Data generation notebook
│   └── sql/
│       ├── scd_type-1.sql        # SCD Type 1 implementation
│       └── scd_type-2.sql        # SCD Type 2 implementation
│
├── installation/
│   └── Presentation.ipynb         # Installation commands and guide
│
├── docs/
│   ├── images/                    # Architecture diagrams and screenshots
│   ├── SCD_TYPE_1.md             # SCD Type 1 documentation
│   ├── SCD_TYPE_2.md             # SCD Type 2 documentation
│   └── TROUBLESHOOTING.md        # Common issues and solutions
│
└── scripts/
    ├── setup_ec2.sh              # EC2 setup automation script
    ├── docker_setup.sh           # Docker installation script
    └── snowflake_setup.sql       # Snowflake initial setup
```

## 🚀 Installation & Setup

### Quick Start

1. **Clone the Repository**
```bash
git clone https://github.com/abidaziz1/Snowflake-Projects.git
cd Snowflake-Projects/Implementing-Slowly-Changing-Dimensions
```

2. **AWS Setup**
   - Create an S3 bucket for data storage
   - Launch an EC2 instance (t2.xlarge, Amazon Linux)
   - Configure security groups (ports: 22, 2080, 4888)
   - Create IAM access keys

3. **EC2 Configuration**
```bash
# SSH into EC2
ssh -i "your-key.pem" ec2-user@your-ec2-public-ip

# Install Docker
sudo yum update -y
sudo yum install docker -y
sudo systemctl start docker
sudo usermod -aG docker ec2-user

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

4. **Copy docker-compose.yml to EC2**
```bash
# From local machine
scp -i "your-key.pem" code/docker-compose.yml ec2-user@your-ec2-ip:/home/ec2-user/docker_exp/
```

5. **Start Docker Services**
```bash
# On EC2
cd docker_exp
docker-compose up -d
```

6. **Port Forwarding (Local Machine)**
```bash
ssh -i "your-key.pem" ec2-user@your-ec2-ip \
  -L 4888:localhost:4888 \
  -L 2080:localhost:2080 \
  -L 8050:localhost:8050
```

7. **Snowflake Setup**
   - Create database: `SCD_DEMO`
   - Create schema: `scd2`
   - Run setup SQL scripts from `code/sql/`

For detailed setup instructions, see [SETUP_GUIDE.md](SETUP_GUIDE.md)

## 💡 Usage

### 1. Generate Sample Data

Access Jupyter Lab at `localhost:4888` and run the Faker notebook:

```python
# Creates customer data CSV files
from faker import Faker
import csv

# Generate 10,000 customer records
# Files saved to FakeDataset/customer_<timestamp>.csv
```

### 2. Configure NiFi Flow

Access NiFi UI at `localhost:2080` and set up processors:

- **ListFile**: Monitor `/opt/nifi/nifi-current/scd` directory
- **FetchFile**: Read CSV files
- **PutS3Object**: Upload to S3 bucket

### 3. Snowflake Ingestion

The pipeline automatically:
- Detects new files in S3 via event notifications
- Triggers Snowpipe for real-time ingestion
- Loads data into `customer_raw` staging table

### 4. Execute SCD Logic

**For SCD Type 1:**
```sql
-- Resume the task
ALTER TASK tsk_scd_raw RESUME;

-- Verify data
SELECT COUNT(*) FROM customer;
```

**For SCD Type 2:**
```sql
-- Resume the task
ALTER TASK tsk_scd_hist RESUME;

-- Check historical records
SELECT * FROM customer_history WHERE IS_CURRENT = FALSE;
```

## 🔄 SCD Implementation

### SCD Type 1: Overwrite

**Characteristics:**
- Overwrites old data with new values
- No historical tracking
- Simple and space-efficient
- Use case: Correcting typos, updating current information

**Implementation:**
```sql
MERGE INTO customer c
USING customer_raw cr
  ON c.customer_id = cr.customer_id
WHEN MATCHED THEN
  UPDATE SET c.first_name = cr.first_name,
             c.email = cr.email,
             c.update_timestamp = CURRENT_TIMESTAMP()
WHEN NOT MATCHED THEN
  INSERT VALUES (cr.customer_id, cr.first_name, ...);
```

### SCD Type 2: Historical Tracking

**Characteristics:**
- Preserves complete change history
- Uses versioning with start_time/end_time
- Maintains is_current flag
- Use case: Audit trails, trend analysis, compliance

**Key Fields:**
- `start_time`: When record became active
- `end_time`: When record was superseded ('9999-12-31' for current)
- `is_current`: Boolean flag for active record

**Implementation:**
```sql
-- View to detect changes
CREATE OR REPLACE VIEW v_customer_change_data AS
SELECT 
  customer_id,
  first_name,
  last_name,
  ...,
  start_time,
  end_time,
  is_current,
  'I' AS dml_type  -- Insert, Update, or Delete
FROM customer_table_changes;

-- Task to merge changes
MERGE INTO customer_history ch
USING v_customer_change_data ccd
  ON ch.customer_id = ccd.customer_id
  AND ch.start_time = ccd.start_time
WHEN MATCHED AND ccd.dml_type = 'U' THEN
  UPDATE SET ch.end_time = ccd.end_time,
             ch.is_current = FALSE
WHEN NOT MATCHED AND ccd.dml_type = 'I' THEN
  INSERT VALUES (...);
```

## 📊 Data Flow

### Complete Pipeline Flow:

1. **Data Generation**
   - Jupyter Notebook generates synthetic customer data
   - CSV files created with timestamp: `customer_YYYYMMDDHHMMSS.csv`
   - Stored in shared workspace: `/opt/workspace/nifi/FakeDataset/`

2. **File Transfer**
   - Files copied to NiFi directory: `/opt/nifi/nifi-current/scd/`
   - NiFi processors monitor directory for new files

3. **Upload to S3**
   - **ListFile** processor detects new CSV files
   - **FetchFile** reads file content
   - **PutS3Object** uploads to S3 bucket

4. **Automated Ingestion**
   - S3 event notification triggers SQS queue
   - Snowpipe receives notification
   - Data loaded into `customer_raw` table

5. **SCD Processing**
   - Stream `customer_table_changes` captures changes
   - View `v_customer_change_data` classifies change types
   - Scheduled tasks execute merge operations
   - History preserved in dimension tables

6. **Analytics Ready**
   - Current state: `customer` table
   - Historical state: `customer_history` table
   - Query both for comprehensive analysis

## 🔍 Monitoring & Validation

### Check Snowpipe Status
```sql
SELECT SYSTEM$PIPE_STATUS('customer_s3_pipe');
SHOW PIPES;
```

### Verify Data Loads
```sql
-- Check staging table
SELECT COUNT(*) FROM customer_raw;

-- Check dimension table
SELECT COUNT(*) FROM customer;

-- Check historical records
SELECT * FROM customer_history 
WHERE customer_id = 123 
ORDER BY start_time DESC;
```

### Monitor Tasks
```sql
SHOW TASKS;

SELECT * FROM TABLE(information_schema.task_history())
WHERE name = 'TSK_SCD_RAW'
ORDER BY scheduled_time DESC
LIMIT 10;
```

## 🐛 Troubleshooting

### Common Issues

**1. Cannot access localhost:4888 or localhost:2080**
- Verify port forwarding is active
- Check Docker containers are running: `docker ps -a`
- Restart Docker: `sudo systemctl restart docker`

**2. Snowpipe not triggering**
- Verify S3 event notification is configured
- Check notification channel ARN matches Snowpipe
- Validate AWS credentials in external stage

**3. Task not executing**
- Resume task: `ALTER TASK task_name RESUME;`
- Check warehouse is active
- Verify task schedule: `SHOW TASKS;`

**4. Stream not capturing changes**
- Ensure stream is created on correct table
- Check stream has not been consumed
- Refresh stream if needed

For detailed troubleshooting, see [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/AmazingFeature`
3. Commit your changes: `git commit -m 'Add some AmazingFeature'`
4. Push to the branch: `git push origin feature/AmazingFeature`
5. Open a Pull Request

### Contribution Guidelines

- Follow existing code style and conventions
- Add comments and documentation for new features
- Update README.md if adding new functionality
- Test thoroughly before submitting PR

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Snowflake documentation and community
- Apache NiFi community
- AWS documentation and tutorials
- Open source contributors

## 📞 Contact & Support

- **Project Maintainer**: Abid Aziz
- **Repository**: [Snowflake-Projects](https://github.com/abidaziz1/Snowflake-Projects)
- **Issues**: [Report a bug](https://github.com/abidaziz1/Snowflake-Projects/issues)

---

## 🎓 Learning Resources

- [Snowflake Documentation](https://docs.snowflake.com/)
- [Apache NiFi Documentation](https://nifi.apache.org/docs.html)
- [Docker Documentation](https://docs.docker.com/)
- [AWS S3 Documentation](https://docs.aws.amazon.com/s3/)
- [SCD Pattern Guide](https://en.wikipedia.org/wiki/Slowly_changing_dimension)

---

**⭐ If you find this project helpful, please consider giving it a star!**

**Made with ❤️ by data engineers, for data engineers**
