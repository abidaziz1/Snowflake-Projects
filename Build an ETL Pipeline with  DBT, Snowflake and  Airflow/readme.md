# Build an ETL Pipeline with DBT, Snowflake and Airflow

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue)](https://www.python.org/)
[![DBT](https://img.shields.io/badge/DBT-Latest-orange)](https://www.getdbt.com/)
[![Snowflake](https://img.shields.io/badge/Snowflake-Cloud-blue)](https://www.snowflake.com/)
[![Apache Airflow](https://img.shields.io/badge/Apache%20Airflow-2.0%2B-red)](https://airflow.apache.org/)
[![AWS](https://img.shields.io/badge/AWS-EC2%20%7C%20S3%20%7C%20SNS-yellow)](https://aws.amazon.com/)

## 📋 Table of Contents
- [Business Overview](#business-overview)
- [Project Architecture](#project-architecture)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Project Structure](#project-structure)
- [Usage](#usage)
- [Testing](#testing)
- [Monitoring & Alerts](#monitoring--alerts)
- [Key Takeaways](#key-takeaways)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Business Overview

**DBT (Data Build Tool)** is a popular open-source command-line tool used in data transformation and modeling processes. This project demonstrates building a production-grade ETL pipeline that integrates DBT with Snowflake and Apache Airflow, leveraging AWS cloud services for deployment.

The pipeline automates the entire data workflow from ingestion to visualization, implementing industry best practices for:
- **Retail Sales Forecasting & Reporting** - Real-time sales trends and automated reporting
- **Financial Fraud Detection & Compliance** - Real-time anomaly detection with automated alerts
- **Healthcare Patient Data Processing** - Critical condition monitoring and data integration

### Why This Architecture?

**Airflow Integration** eliminates manual execution by:
- Automating data loading and transformations
- Providing robust error handling and monitoring
- Reducing human intervention for repetitive tasks
- Enabling scheduled and event-driven workflows

**DBT Tags** make orchestration seamless by grouping related models for efficient batch execution.

## 🏗️ Project Architecture

```
┌─────────────┐      ┌──────────────┐      ┌─────────────┐      ┌──────────────┐      ┌─────────┐
│  Source     │─────▶│   AWS S3     │─────▶│   Airflow   │─────▶│  Snowflake  │───▶ │  Users  │
│  CSV Files  │      │   Landing    │      │   + DBT     │      │  Data Warehouse│    │QuickSight│
└─────────────┘      └──────────────┘      └─────────────┘      └──────────────┘      └─────────┘
                                                   │
                                                   │ Pipeline Failure
                                                   ▼
                                            ┌──────────────┐
                                            │   AWS SNS    │
                                            │  + Slack     │
                                            └──────────────┘
                                                   │
                                          ┌────────┴────────┐
                                          ▼                 ▼
                                    ┌──────────┐    ┌──────────┐
                                    │  Email   │    │  Slack   │
                                    │  Alert   │    │ Message  │
                                    └──────────┘    └──────────┘
```

### Pipeline Components

1. **Data Ingestion** - Raw CSV files uploaded to AWS S3 (landing layer)
2. **Data Processing** - DBT and Airflow orchestrate ETL workflows on AWS EC2
3. **Data Storage** - Transformed datasets stored in Snowflake
4. **Visualization** - AWS QuickSight dashboards for business insights
5. **Error Handling** - AWS SNS and Slack notifications for real-time alerts

## ✨ Features

- 🔄 **Automated ETL Pipeline** - End-to-end workflow orchestration
- 📊 **DBT Data Modeling** - Modular, testable SQL transformations
- ☁️ **Cloud-Native Architecture** - Scalable AWS infrastructure
- 🔔 **Real-Time Alerts** - Slack and email notifications via SNS
- 📈 **Data Visualization** - Interactive QuickSight dashboards
- 🧪 **Built-in Testing** - DBT test cases for data quality validation
- 🏷️ **Tag-Based Execution** - Efficient model orchestration
- 🔐 **Secure Credentials** - AWS Systems Manager Parameter Store integration

## 🛠️ Tech Stack

**Languages:**
- Python 3.8+
- SQL

**Tools & Frameworks:**
- DBT (Data Build Tool)
- Apache Airflow 2.0+
- PostgreSQL (Airflow metadata)

**Cloud Services:**
- **AWS EC2** - Compute instance for Airflow/DBT
- **AWS S3** - Data landing zone
- **AWS SNS** - Notification service
- **AWS QuickSight** - Data visualization
- **AWS Systems Manager** - Secure credential storage
- **Snowflake** - Cloud data warehouse

**DevOps:**
- Git/GitHub
- PuTTY (Windows SSH client)
- Virtual Environment (venv)

## 📋 Prerequisites

### Required Accounts
- AWS Account with appropriate permissions
- Snowflake Account
- Slack Workspace (for notifications)
- GitHub Account

### Required Software
- PuTTY (Windows) or SSH client (Mac/Linux)
- Web browser for AWS Console access
- Git

### AWS Best Practices
- ✅ Enable MFA security
- ✅ Create IAM users with least privilege
- ✅ Set up billing alerts and budgets
- ✅ Use appropriate AWS region
- ✅ Delete unused resources to minimize costs
- ✅ Rotate credentials periodically
- ✅ Follow consistent naming conventions

## 🚀 Installation & Setup

### 1. EC2 Instance Setup

Create an Ubuntu EC2 instance (t2.large recommended):

```bash
# Instance specifications
Instance Type: t2.large
AMI: Ubuntu 22.04 LTS
Storage: 16 GB (increased from default 8GB)
Key Pair: Create and download .ppk file
```

Configure security group inbound rules:
- **Port 8080** - Custom TCP (Airflow UI) - Restrict to "My IP"
- **Port 22** - SSH - Restrict to "My IP"

### 2. Connect to EC2 Instance

Using PuTTY (Windows):
1. Open PuTTY Application
2. Enter EC2 Public IPv4 address
3. Navigate to SSH → Auth → Credentials
4. Load your .ppk file
5. Click "Open" and accept the connection
6. Login as `ubuntu`

### 3. Install PostgreSQL and Configure Airflow User

```bash
# Update system packages
sudo apt-get update

# Install PostgreSQL and Python adapter
sudo apt-get install python3-psycopg2
sudo apt-get install postgresql postgresql-contrib

# Create Airflow system user
sudo adduser airflow
sudo usermod -aG sudo airflow

# Switch to Airflow user
su - airflow
```

### 4. Configure PostgreSQL Database

```bash
# Access PostgreSQL as postgres user
sudo -u postgres psql

# Create Airflow database and user
CREATE USER airflow PASSWORD 'airflow';
CREATE DATABASE airflow;

# Switch to Airflow database
\c airflow

# Grant necessary privileges
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA PUBLIC TO airflow;
GRANT ALL PRIVILEGES ON SCHEMA public TO airflow;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO airflow;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO airflow;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON FUNCTIONS TO airflow;

# Exit PostgreSQL
\q

# Restart PostgreSQL service
sudo service postgresql restart
```

### 5. Configure PostgreSQL for Remote Access

```bash
# Edit authentication configuration
sudo vi /etc/postgresql/14/main/pg_hba.conf
# Change: host all all 127.0.0.1/32 md5
# To: host all all 0.0.0.0/0 trust

# Edit PostgreSQL configuration
sudo vi /etc/postgresql/14/main/postgresql.conf
# Uncomment and change: listen_addresses = '*'

# Restart PostgreSQL
sudo service postgresql restart
```

### 6. Set Up Python Virtual Environment

```bash
# Install virtualenv tools
sudo apt install python3-virtualenv
sudo apt install python3-venv

# Create and activate virtual environment
python3 -m venv ~/dbt-env
source ~/dbt-env/bin/activate

# Install required packages
pip install virtualenv
pip install apache-airflow[postgres,s3,aws,slack]
pip install pandas
pip install snowflake-connector-python
pip install boto3
pip install dbt-snowflake
```

### 7. Configure Airflow

```bash
# Navigate to Airflow directory
cd ~/airflow

# Edit Airflow configuration
vi airflow.cfg

# Update database connection
# Change: sql_alchemy_conn = sqlite:////home/airflow/airflow/airflow.db
# To: sql_alchemy_conn = postgresql+psycopg2://airflow:airflow@localhost:5432/airflow

# Change executor
# Change: executor = SequentialExecutor
# To: executor = LocalExecutor

# Initialize Airflow database
airflow db init

# Create admin user
airflow users create \
    --username admin \
    --password admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@airflow.com
```

### 8. Clone Project Repository

```bash
# Install Git
sudo apt install git

# Clone Airflow DAGs repository
git clone https://<YOUR_GITHUB_TOKEN>@github.com/<YOUR_USERNAME>/airflow-code.git

# Clone DBT project repository
git clone https://<YOUR_GITHUB_TOKEN>@github.com/<YOUR_USERNAME>/dbt-code.git
```

**Generating GitHub Personal Access Token:**
1. Go to GitHub Settings → Developer settings
2. Select Personal Access Tokens → Tokens (classic)
3. Generate new token with scopes: `repo`, `write:discussion`, `project`
4. Copy and securely store the token

### 9. Configure DBT

```bash
# Create DBT directory
mkdir ~/.dbt
cd ~/.dbt

# Create profiles configuration
vi profiles.yml
```

Add the following configuration (replace placeholders):

```yaml
Netflix:
  outputs:
    default:
      type: snowflake
      account: <SNOWFLAKE_ACCOUNT>
      user: <SNOWFLAKE_USER>
      password: <SNOWFLAKE_PASSWORD>
      role: ACCOUNTADMIN
      database: prod
      warehouse: COMPUTE_WH
      schema: DBT_RAW
    
    dev:
      type: snowflake
      account: <SNOWFLAKE_ACCOUNT>
      user: <SNOWFLAKE_USER>
      password: <SNOWFLAKE_PASSWORD>
      role: ACCOUNTADMIN
      database: prod
      warehouse: COMPUTE_WH
      schema: DBT_TRANSFORM
    
    prod:
      type: snowflake
      account: <SNOWFLAKE_ACCOUNT>
      user: <SNOWFLAKE_USER>
      password: <SNOWFLAKE_PASSWORD>
      role: ACCOUNTADMIN
      database: prod
      warehouse: COMPUTE_WH
      schema: DBT_PROD
```

### 10. Set Up Snowflake

In Snowflake UI:

```sql
-- Create database
CREATE DATABASE prod;

-- Create schemas
CREATE SCHEMA prod.dbt_raw;
CREATE SCHEMA prod.dbt_transform;
CREATE SCHEMA prod.dbt_prod;
```

### 11. Configure AWS Resources

**Create S3 Bucket:**
1. Navigate to AWS S3
2. Create bucket: `netflix-analytics-data`
3. Create folder: `raw_files`

**Store Snowflake Credentials in Parameter Store:**
1. Navigate to AWS Systems Manager → Parameter Store
2. Create parameters:
   - `/snowflake/accountname` (SecureString)
   - `/snowflake/username` (SecureString)
   - `/snowflake/password` (SecureString)

**Create IAM Role for EC2:**
1. Navigate to IAM → Roles → Create Role
2. Select AWS Service → EC2
3. Attach policies:
   - `AmazonS3FullAccess`
   - `AmazonSSMFullAccess`
   - `AmazonEC2RoleforSSM`
4. Name the role: `AirflowS3SSMRole`
5. Attach to EC2 instance: Instance → Actions → Security → Modify IAM Role

### 12. Update Airflow DAG Configuration

```bash
cd ~/airflow
vi airflow.cfg

# Update DAGs folder path
# Change dags_folder to: /home/airflow/airflow-code/dags
```

### 13. Start Airflow Services

```bash
# Activate virtual environment
source ~/dbt-env/bin/activate

# Start Airflow webserver and scheduler
airflow webserver &
airflow scheduler &
```

### 14. Access Airflow UI

Open browser and navigate to:
```
http://<EC2_PUBLIC_IP>:8080
```

Login credentials:
- Username: `admin`
- Password: `admin`

## 📁 Project Structure

```
.
├── airflow-code/
│   ├── dags/
│   │   ├── Netflix_Data_Analytics.py    # Main DAG file
│   │   ├── source_load/
│   │   │   └── data_load.py             # S3 to Snowflake loader
│   │   └── alerting/
│   │       └── slack_alert.py           # Slack notification script
│   └── README.md
│
├── dbt-code/
│   ├── models/
│   │   ├── staging/                     # Raw data models
│   │   ├── intermediate/                # Transformation models
│   │   └── marts/                       # Final fact/dim tables
│   ├── tests/                           # DBT test cases
│   ├── dbt_project.yml                  # DBT project configuration
│   └── profiles.yml                     # Snowflake connection profiles
│
└── datasets/
    ├── titles.csv                       # Netflix titles data
    └── credits.csv                      # Netflix credits data
```

## 💻 Usage

### Running the Complete Pipeline

1. **Upload Data to S3:**
   ```bash
   # Upload CSV files to s3://netflix-analytics-data/raw_files/
   ```

2. **Trigger DAG in Airflow UI:**
   - Navigate to http://<EC2_IP>:8080
   - Locate `Netflix_Data_Analytics` DAG
   - Click the play button to trigger

3. **Monitor Execution:**
   - View task progress in Graph View
   - Check logs for each task
   - Verify success/failure status

### Pipeline Tasks Flow

```
start_task 
  → check_s3_files (S3 Sensor)
  → load_data_to_snowflake (Python Script)
  → run_stage_model (DBT Staging)
  → run_fact_dim_models (DBT Fact/Dimension)
  → run_test_cases (DBT Tests)
  → slack_success_alert
  → end_task
```

### Running DBT Models Manually

```bash
# Activate virtual environment
source ~/dbt-env/bin/activate

# Navigate to DBT project
cd ~/dbt-code/dbt-code

# Run all models
dbt run

# Run specific model
dbt run --select credits_dim --target prod

# Run models by tag
dbt run --select tag:dimension

# Run tests
dbt test

# Generate documentation
dbt docs generate
dbt docs serve
```

### DBT Tagging Strategy

Models are organized using tags for efficient orchestration:

```sql
-- In model SQL file
{{ config(
    tags=[var('TAG_DIMENSION')]
) }}

SELECT * FROM {{ ref('source_table') }}
```

Tags defined in `dbt_project.yml`:
```yaml
vars:
  TAG_DIMENSION: 'DIMENSION'
  TAG_FACT: 'FACT'
  TAG_STAGING: 'STAGING'
```

Execute by tag in Airflow:
```python
dbt_task = BashOperator(
    task_id='run_dimension_models',
    bash_command=f'cd {DBT_PROJECT_DIR} && dbt run --select tag:dimension --target prod'
)
```

## 🧪 Testing

### DBT Test Cases

DBT provides built-in testing for data validation:

```sql
-- tests/show_details_not_null.sql
SELECT * 
FROM {{ ref('MOVIES_SERIES_SHARE') }} 
WHERE GENRES IS NULL
```

**Test Logic:**
- If query returns **0 rows** → Test **PASSES** ✅
- If query returns **any rows** → Test **FAILS** ❌

### Running Tests

```bash
# Run all tests
dbt test

# Run tests for specific model
dbt test --select credits_dim

# Run tests by tag
dbt test --select tag:dimension
```

### Test Types

1. **Schema Tests** (in `schema.yml`):
```yaml
models:
  - name: credits_dim
    columns:
      - name: id
        tests:
          - unique
          - not_null
      - name: name
        tests:
          - not_null
```

2. **Custom Data Tests** (SQL files in `tests/`):
```sql
-- Check for future dates
SELECT * 
FROM {{ ref('titles_fact') }}
WHERE release_date > CURRENT_DATE()
```

## 🔔 Monitoring & Alerts

### Slack Integration

**1. Create Slack App:**
- Visit [Slack API Apps](https://api.slack.com/apps)
- Create New App → From scratch
- Select your workspace
- Enable Incoming Webhooks
- Generate Webhook URL

**2. Configure Airflow Connection:**
```
Connection ID: slack-connection
Connection Type: HTTP
Host: https://hooks.slack.com/services/
Password: <Last part of webhook URL>
```

**3. Test Webhook:**
```bash
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Pipeline test successful!"}' \
  <YOUR_WEBHOOK_URL>
```

### AWS SNS Integration

**1. Create SNS Topic:**
```bash
# In AWS SNS Console
Topic Name: airflow-pipeline-alerts
Protocol: Email
Endpoint: your-email@example.com
```

**2. Subscribe to Topic:**
- Confirm subscription via email
- Add multiple subscribers (email, SMS)

**3. Configure Airflow Callback:**
```python
from airflow.providers.amazon.aws.hooks.sns import SnsHook

def failure_callback(context):
    sns_hook = SnsHook(aws_conn_id='aws_default')
    sns_hook.publish_to_target(
        target_arn='arn:aws:sns:region:account:airflow-pipeline-alerts',
        message=f"DAG {context['dag'].dag_id} failed!",
        subject='Airflow Pipeline Failure Alert'
    )

dag = DAG(
    dag_id='Netflix_Data_Analytics',
    default_args={'on_failure_callback': failure_callback},
    ...
)
```

### Alert Types

- ✅ **Success Alerts** - Pipeline completion notifications
- ❌ **Failure Alerts** - Task failure with error details
- ⚠️ **Warning Alerts** - Data quality issues from DBT tests
- 📊 **Summary Alerts** - Daily/weekly pipeline statistics

## 📚 Key Takeaways

- ✅ Understanding DBT fundamentals and data modeling concepts
- ✅ Installing and configuring Airflow with PostgreSQL backend
- ✅ Setting up DBT with Snowflake integration
- ✅ Implementing tag-based model orchestration
- ✅ Creating automated ETL pipelines with error handling
- ✅ Integrating Slack and SNS for real-time notifications
- ✅ Building fact and dimension tables following best practices
- ✅ Implementing data quality tests with DBT
- ✅ Visualizing data insights with AWS QuickSight
- ✅ Managing cloud infrastructure on AWS (EC2, S3, IAM)
- ✅ Securing credentials with AWS Systems Manager

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Development Guidelines

- Follow PEP 8 style guide for Python code
- Write meaningful commit messages
- Add tests for new features
- Update documentation as needed
- Ensure all tests pass before submitting PR

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [DBT Documentation](https://docs.getdbt.com/)
- [Apache Airflow Documentation](https://airflow.apache.org/docs/)
- [Snowflake Documentation](https://docs.snowflake.com/)
- [AWS Documentation](https://docs.aws.amazon.com/)

## 📧 Contact

For questions or support, please open an issue in the GitHub repository.

---

**Note:** Remember to terminate your EC2 instance when not in use to avoid unnecessary AWS charges. Always follow security best practices and never commit sensitive credentials to version control.
