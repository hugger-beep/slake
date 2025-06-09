# AWS Security Lake: Enabling and Disabling Impact

## Impacts of Enabling AWS Security Lake

### Positive Impacts
- **Centralized Security Data**: Consolidates security logs and events from various AWS services, third-party sources, and custom sources
- **Standardized Format**: Converts data to the Open Cybersecurity Schema Framework (OCSF) format
- **Enhanced Analysis**: Enables more comprehensive threat detection and investigation
- **Organization-wide Visibility**: Provides security visibility across multiple AWS accounts
- **Tool Integration**: Facilitates integration with security analysis tools and SIEM systems

### Operational Impacts
- **Storage Costs**: Incurs costs for storing security data in S3 buckets
- **Data Transfer Costs**: May incur costs for data movement between regions or to subscribers
- **Resource Creation**: Creates resources like S3 buckets, Glue databases, and Lake Formation permissions
- **IAM Requirements**: Requires specific IAM roles and permissions

## Resources Created When Enabling Security Lake

When Security Lake is enabled in an organization, AWS automatically creates and manages several resources:

### IAM Roles and Permissions
- **AmazonSecurityLakeServiceRolePolicy**: Service-linked role for Security Lake operations
- **Security Lake Collection Role**: IAM role for collecting logs from AWS services
- **Security Lake Access Role**: IAM role for subscribers to access data
- **Lake Formation Permissions**: Permissions to manage data lake resources

### AWS Resources
- **S3 Buckets**: 
  - Data lake bucket for storing normalized security data
  - Access logs bucket for S3 access logging
- **Glue Resources**:
  - Glue Database for Security Lake
  - Glue Tables for each data source type
  - Glue Crawlers to catalog data
- **Lake Formation**:
  - Data lake settings and permissions
  - Resource links and data locations
- **CloudWatch Logs**:
  - Log groups for Security Lake operations
- **KMS Keys** (if encryption enabled):
  - Customer managed keys for data encryption

### Networking
- **VPC Endpoints** (if configured):
  - Interface endpoints for private connectivity

### Resource Distribution in AWS Organizations

Resources are distributed across accounts as follows:

- **Delegated Administrator Account**:
  - Primary S3 buckets for centralized data storage
  - Glue databases and tables for the entire organization
  - Lake Formation permissions and settings
  - Subscriber access management resources
  - Service-linked roles for administration
  - CloudWatch log groups for centralized monitoring

- **Member/Participating Accounts**:
  - Service-linked roles for log collection
  - Collection IAM roles specific to each account
  - Local CloudWatch log groups for collection activities
  - Data collection processes and configurations

The delegated administrator account hosts most of the centralized infrastructure, while member accounts contain only the necessary resources for collecting and forwarding logs to the central repository.

## Security Lake Collection Process

### How Security Lake Collects Data
AWS Security Lake uses built-in collection mechanisms that:
- Collect log data from enabled AWS sources within an account
- Transform collected data into the Open Cybersecurity Schema Framework (OCSF) format
- Handle the extraction, transformation, and loading (ETL) processes
- Manage the secure transfer of normalized data to the central S3 bucket

> **Official Documentation**: According to the [AWS Security Lake User Guide](https://docs.aws.amazon.com/security-lake/latest/userguide/data-collection.html): "When you add a source to Security Lake, Security Lake automatically begins collecting logs and events from that source. Security Lake converts the collected data to the OCSF format and stores it in your Amazon S3 bucket."

### Deployment and Management
- **Who Deploys**: AWS automatically deploys and manages the Security Lake Collection Process
- **How It's Deployed**: 
  - Deployed automatically when Security Lake is enabled and sources are configured
  - No manual installation or configuration required by customers
  - Provisioned in each member account that has Security Lake enabled
  - Operates as a service-linked component with appropriate IAM permissions

### Maintenance and Updates
- **Fully AWS-Managed**: AWS handles all maintenance, updates, and scaling
- **No Customer Overhead**: Customers don't need to patch, update, or monitor the collection process
- **Automatic Scaling**: Scales automatically based on log volume
- **Resilience**: Built with redundancy to ensure reliable log collection

### Configuration
- Customers only need to:
  - Enable desired log sources through the Security Lake console or API
  - Configure which regions to collect from
  - Set up appropriate IAM permissions
  - No direct agent configuration is required

## Impacts of Disabling AWS Security Lake

- **Data Retention**: Existing collected data remains in S3 buckets until manually deleted
- **Historical Data Access**: You can still access historical data but no new data will be collected
- **Subscriber Impact**: Subscribers will no longer receive new security data
- **Resource Persistence**: AWS resources created by Security Lake remain until manually removed
- **Cost Implications**: Stops incurring costs for new data ingestion, but storage costs continue for existing data

## Can Customers Disable Security Lake Without Issues?

Yes, customers can disable AWS Security Lake at any time, but should be aware of these considerations:

1. **Resource Cleanup**: Disabling Security Lake does not automatically delete created resources (S3 buckets, Glue tables, etc.). These must be manually removed to avoid ongoing storage costs.

2. **Data Access**: After disabling, you'll still have access to previously collected data, but no new data will be ingested.

3. **Subscriber Notification**: It's important to notify any subscribers that they will no longer receive new security data.

4. **Re-enabling Process**: If you decide to re-enable Security Lake later, you'll need to reconfigure your settings and data sources.

5. **Billing Transition**: While new ingestion costs will stop immediately, you'll continue to be billed for stored data and any remaining resources.

There are no technical blockers to disabling Security Lake, but proper planning for resource management and communication with stakeholders is recommended to ensure a smooth transition.

## Organizational Best Practices

- **Regular Review**: Periodically review Security Lake usage and costs
- **Data Lifecycle Policies**: Implement lifecycle policies for security data to manage storage costs
- **Documentation**: Maintain documentation of Security Lake configuration for future reference
- **Testing**: Consider testing the disable/re-enable process in a non-production environment first

## Security Lake Data Flow Scenarios

### Scenario 1: Separate Delegated Administrator and Member Account

```mermaid
flowchart TB
    subgraph MemberAccount[Member Account]
        VPC[VPC Flow Logs] -->|Generate logs| VPCRole[Collection IAM Role]
        CT[CloudTrail] -->|Generate logs| CTRole[Collection IAM Role]
        VPCRole -->|Collect| SLProcess[Collection Process]
        CTRole -->|Collect| SLProcess
        SLProcess -->|Transform to OCSF| SLService[Security Lake Service]
    end
    
    subgraph DelegatedAccount[Delegated Administrator Account]
        SLS3[Central S3 Bucket] 
        SLGlue[Glue Database/Tables]
        SLLF[Lake Formation]
        SLSub[Subscriber Access]
    end
    
    SLService -->|Forward normalized data| SLS3
    SLS3 -->|Catalog| SLGlue
    SLGlue -->|Manage permissions| SLLF
    SLLF -->|Grant access| SLSub
    
    subgraph Subscribers[Subscribers]
        SIEM[SIEM Systems]
        SecTools[Security Tools]
        Analytics[Analytics Platforms]
    end
    
    SLSub -->|Query data| SIEM
    SLSub -->|Query data| SecTools
    SLSub -->|Query data| Analytics
    
    classDef memberClass fill:#E8F4F8,stroke:#0072AA
    classDef delegatedClass fill:#F8F0E8,stroke:#AA7200
    classDef subscriberClass fill:#F0E8F8,stroke:#7200AA
    
    class MemberAccount memberClass
    class DelegatedAccount delegatedClass
    class Subscribers subscriberClass
```

### Scenario 2: Single Account as Both Delegated Administrator and Member

```mermaid
flowchart TB
    subgraph SingleAccount[Single Account - Delegated Admin + Member]
        subgraph LogSources[Log Sources]
            VPC[VPC Flow Logs] -->|Generate logs| VPCRole[Collection IAM Role]
            CT[CloudTrail] -->|Generate logs| CTRole[Collection IAM Role]
        end
        
        subgraph CollectionLayer[Collection Layer]
            VPCRole -->|Collect| SLProcess[Collection Process]
            CTRole -->|Collect| SLProcess
            SLProcess -->|Transform to OCSF| SLService[Security Lake Service]
        end
        
        subgraph StorageLayer[Storage & Management Layer]
            SLService -->|Store normalized data| SLS3[S3 Bucket]
            SLS3 -->|Catalog| SLGlue[Glue Database/Tables]
            SLGlue -->|Manage permissions| SLLF[Lake Formation]
            SLLF -->|Grant access| SLSub[Subscriber Access]
        end
    end
    
    subgraph Subscribers[Subscribers]
        SIEM[SIEM Systems]
        SecTools[Security Tools]
        Analytics[Analytics Platforms]
    end
    
    SLSub -->|Query data| SIEM
    SLSub -->|Query data| SecTools
    SLSub -->|Query data| Analytics
    
    classDef singleClass fill:#EFF8E8,stroke:#72AA00
    classDef subscriberClass fill:#F0E8F8,stroke:#7200AA
    classDef logClass fill:#E8F4F8,stroke:#0072AA
    classDef collectionClass fill:#F8F0E8,stroke:#AA7200
    classDef storageClass fill:#F0E8F8,stroke:#7200AA
    
    class SingleAccount singleClass
    class Subscribers subscriberClass
    class LogSources logClass
    class CollectionLayer collectionClass
    class StorageLayer storageClass
```

In Scenario 1, the member account collects and transforms data before forwarding it to the delegated administrator account for centralized storage and management. The delegated administrator maintains the central infrastructure and controls subscriber access.

In Scenario 2, all components exist within a single account that serves as both the delegated administrator and member. The data flow is more direct since collection, transformation, storage, and access management all occur within the same account.

# Detailed Architecture: RDS Data Streams to AWS Security Lake

## Component Diagram

```mermaid
graph TD
    subgraph "Account 1 RDS Source"
        RDS[RDS Database] -->|Enable Activity Streams| KDS1[Kinesis Data Stream]
        KDS1 -->|Consume events| LF1[Lambda Function]
        LF1 -->|Transform to OCSF| KF1[Kinesis Firehose]
        KF1 -->|Batch & deliver| S3B1[S3 Bucket]
        IAM1[IAM Role] -.->|Permissions| S3B1
        CWL1[CloudWatch Logs] -.->|Monitor| LF1
    end
    
    subgraph "Security Lake Account"
        S3B1 -->|Cross-account access| SL[Security Lake]
        SL -->|Normalize & store| SLS3[Security Lake S3]
        SL -->|Catalog| SLGC[Glue Catalog]
        SLGC -->|Query| Athena[Amazon Athena]
        SL -->|Notify| SNS[SNS Topic]
        SNS -->|Alert| SUB[Security Lake Subscribers]
        IAM2[IAM Role] -.->|Permissions| SLS3
    end
    
    subgraph "Security Operations"
        SUB -->|Access data| SIEM[SIEM Solution]
        SUB -->|Access data| SA[Security Analytics]
        Athena -->|Ad-hoc analysis| TH[Threat Hunting]
        QS[QuickSight] -->|Visualize| SLGC
    end
```

## Data Flow Sequence

```mermaid
sequenceDiagram
    participant RDS as RDS Database
    participant KDS as Kinesis Data Stream
    participant Lambda as Lambda Function
    participant Firehose as Kinesis Firehose
    participant S3 as S3 Bucket Acct1
    participant SecLake as Security Lake
    participant SLS3 as Security Lake S3
    participant Glue as Glue Catalog
    participant SIEM as SIEM/Security Tools
    
    RDS->>KDS: Stream database activity events
    KDS->>Lambda: Deliver events in batches
    Lambda->>Lambda: Transform to OCSF format
    Lambda->>Firehose: Send transformed events
    Firehose->>S3: Batch and deliver to S3
    S3->>SecLake: Custom source ingestion
    SecLake->>SLS3: Store normalized data
    SecLake->>Glue: Update catalog with new data
    SecLake->>SIEM: Notify of new data via SNS
    SIEM->>SLS3: Query/access security data
```

## Technical Implementation Details

### RDS Database Activity Streams Configuration

```json
{
  "AWS::RDS::DBInstance": {
    "Properties": {
      "EnableActivityStream": true,
      "ActivityStreamKinesisStreamName": "rds-activity-stream",
      "ActivityStreamMode": "async",
      "ActivityStreamEngineNativeAuditFieldsIncluded": true
    }
  }
}
```

### Lambda Function for OCSF Transformation

```python
import json
import base64
import boto3
from datetime import datetime

def lambda_handler(event, context):
    firehose = boto3.client('firehose')
    
    output_records = []
    for record in event['Records']:
        # Decode and parse the RDS activity stream record
        payload = json.loads(base64.b64decode(record['kinesis']['data']))
        
        # Transform to OCSF format
        ocsf_event = {
            "metadata": {
                "version": "1.0.0-rc.2",
                "class_name": "database_activity",
                "class_uid": 3002,
                "timestamp": datetime.utcnow().isoformat() + "Z"
            },
            "database": {
                "name": payload.get('databaseName', ''),
                "schema": payload.get('schema', ''),
                "operation": payload.get('command', ''),
                "user": payload.get('dbUserName', '')
            },
            "src": {
                "ip": payload.get('sourceIpAddress', ''),
                "port": payload.get('sourcePort', 0)
            },
            "dst": {
                "ip": payload.get('databaseHost', ''),
                "port": payload.get('databasePort', 0)
            },
            "resources": [{
                "type": "Database",
                "name": payload.get('databaseName', ''),
                "uid": payload.get('dbInstanceId', '')
            }],
            "cloud": {
                "provider": "AWS",
                "account": {
                    "uid": context.invoked_function_arn.split(":")[4]
                },
                "region": context.invoked_function_arn.split(":")[3]
            }
        }
        
        # Add to output batch
        output_records.append({
            'Data': json.dumps(ocsf_event) + '\n'
        })
    
    # Send to Firehose
    firehose.put_record_batch(
        DeliveryStreamName='rds-activity-to-s3',
        Records=output_records
    )
    
    return {
        'statusCode': 200,
        'body': f'Processed {len(output_records)} records'
    }
```

### Security Lake Custom Source Configuration

```json
{
  "source": {
    "sourceArn": "arn:aws:s3:::account1-rds-activity-logs",
    "sourceName": "rds-activity-streams",
    "sourceVersion": "1.0",
    "dataFormat": "OCSF",
    "ocsf": {
      "class": "database_activity",
      "version": "1.0.0-rc.2"
    },
    "s3SourceConfig": {
      "bucketArn": "arn:aws:s3:::account1-rds-activity-logs",
      "prefix": "rds-activity/",
      "roleArn": "arn:aws:iam::security-account-id:role/SecurityLakeRDSIngestionRole"
    }
  }
}
```

### Cross-Account IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::security-account-id:role/SecurityLakeRDSIngestionRole"
      },
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::account1-rds-activity-logs",
        "arn:aws:s3:::account1-rds-activity-logs/*"
      ]
    }
  ]
}
```

## Monitoring and Operations

**CloudWatch Metrics to Monitor:**
* Lambda invocation and error rates
* Kinesis Data Stream throttling events
* Firehose delivery success rate
* Security Lake ingestion metrics

**Operational Considerations:**
* Set appropriate retention periods in Security Lake
* Configure alerts for ingestion failures
* Implement data quality checks
* Regularly validate OCSF schema compliance


# Security Lake PHI/PII Handling Scenarios

## Scenario 1: Redact PHI/PII Before Storage

```mermaid
graph TD
    subgraph "Application Source"
        APP[Application] -->|Generate logs| LOGS[Application Logs with PHI/PII]
        LOGS -->|Stream| KDS[Kinesis Data Stream]
        KDS -->|Process| LF1[Lambda Function]
        LF1 -->|Redact PHI/PII| KF[Kinesis Firehose]
        KF -->|Store| S3[S3 Bucket]
    end
    
    subgraph "Security Lake Account"
        S3 -->|Ingest| SL[Security Lake]
        SL -->|Store redacted data| SLS3[Security Lake S3]
        SL -->|Catalog| GLUE[AWS Glue Catalog]
        GLUE -->|Query| ATHENA[Amazon Athena]
    end
    
    subgraph "Security Operations"
        ATHENA -->|Safe queries| ANALYST[Security Analyst]
        ATHENA -->|No PHI/PII exposure| DASH[Dashboards]
    end
    
    %% Components for redaction
    LF1 -.->|Uses| COMP[Amazon Comprehend]
    COMP -.->|Detect PHI/PII| LF1
    LF1 -.->|Audit| CW[CloudWatch Logs]
```

## Scenario 2: Column-Level Access Control

```mermaid
graph TD
    subgraph "Application Source"
        APP[Application] -->|Generate logs| LOGS[Application Logs with PHI/PII]
        LOGS -->|Stream| KDS[Kinesis Data Stream]
        KDS -->|Process| LF1[Lambda Function]
        LF1 -->|Tag PHI/PII columns| KF[Kinesis Firehose]
        KF -->|Store| S3[S3 Bucket]
    end
    
    subgraph "Security Lake Account"
        S3 -->|Ingest| SL[Security Lake]
        SL -->|Store with metadata| SLS3[Security Lake S3]
        SL -->|Catalog with tags| GLUE[AWS Glue Catalog]
        GLUE -->|Query with LF permissions| ATHENA[Amazon Athena]
    end
    
    subgraph "Access Control"
        LF[Lake Formation] -->|Column-level security| GLUE
        IAM[IAM Roles] -->|Define permissions| LF
        ATHENA -->|Filtered results| ANALYST[Security Analyst]
        ATHENA -->|Full access| ADMIN[Security Admin]
    end
    
    %% Data classification
    LF1 -.->|Uses| MACIE[Amazon Macie]
    MACIE -.->|Classify sensitive data| LF1
```

## Scenario 3: Role-Based Access Control

```mermaid
graph TD
    subgraph "Application Source"
        APP[Application] -->|Generate logs| LOGS[Application Logs with PHI/PII]
        LOGS -->|Stream| KF[Kinesis Firehose]
        KF -->|Store| S3[S3 Bucket]
    end
    
    subgraph "Security Lake Account"
        S3 -->|Ingest| SL[Security Lake]
        SL -->|Store data| SLS3[Security Lake S3]
        SL -->|Register tables| GLUE[AWS Glue Catalog]
    end
    
    subgraph "Access Control Layer"
        GLUE -->|Admin access| ADMIN_VIEW[Admin View]
        GLUE -->|Limited access| ANALYST_VIEW[Analyst View]
        LF[Lake Formation] -->|Manage permissions| GLUE
        IAM[IAM Roles] -->|Define roles| LF
    end
    
    subgraph "Query Layer"
        ADMIN_VIEW -->|Full data access| ADMIN_ATHENA[Admin Athena]
        ANALYST_VIEW -->|No PHI/PII access| ANALYST_ATHENA[Analyst Athena]
        ADMIN_ATHENA -->|Complete queries| ADMIN[Security Admin]
        ANALYST_ATHENA -->|Limited queries| ANALYST[Security Analyst]
    end
```

## Implementation Details

### Scenario 1: Redaction Implementation

* Lambda uses Amazon Comprehend to detect PHI/PII entities
* Replaces sensitive data with tokens like [REDACTED-SSN], [REDACTED-NAME]
* Maintains log structure and context while removing sensitive values
* Advantages: Simplest access model, no risk of PHI/PII exposure
* Disadvantages: Original data is permanently altered

### Scenario 2: Column-Level Access Control

* Data is stored intact with sensitive fields tagged
* Lake Formation provides column-level security
* Data catalog tags identify PHI/PII columns
* Permission sets control which roles can see which columns
* Advantages: Preserves original data, granular access control
* Disadvantages: More complex to set up, requires careful permission management

### Scenario 3: Role-Based Access Control

* Creates separate views of the same data for different roles
* Admin view contains all fields including PHI/PII
* Analyst view filters out or masks sensitive fields
* Uses Lake Formation row-level and column-level security
* Advantages: Most flexible approach, maintains data integrity
* Disadvantages: Most complex to implement and maintain

## Scenario 4: Batch Redaction with AWS Glue (Cost-Optimized)

```mermaid
graph TD
    subgraph "Application Source"
        APP[Application] -->|Generate logs| LOGS[Application Logs with PHI/PII]
        LOGS -->|Direct upload| S3RAW[S3 Raw Bucket]
    end
    
    subgraph "Batch Processing"
        S3RAW -->|Daily/Weekly| GLUE[AWS Glue ETL Job]
        GLUE -->|Redact PHI/PII| S3CLEAN[S3 Clean Bucket]
        GLUE -.->|Uses| REGEX[Regex Patterns]
    end
    
    subgraph "Security Lake Account"
        S3CLEAN -->|Ingest| SL[Security Lake]
        SL -->|Store redacted data| SLS3[Security Lake S3]
        SL -->|Catalog| GLUEC[AWS Glue Catalog]
        GLUEC -->|Query| ATHENA[Amazon Athena]
    end
    
    subgraph "Security Operations"
        ATHENA -->|Safe queries| ANALYST[Security Analyst]
        ATHENA -->|No PHI/PII exposure| DASH[Dashboards]
    end
```

### Scenario 4: Cost-Optimized Batch Redaction

* Uses scheduled AWS Glue ETL jobs instead of real-time Lambda processing
* Processes logs in batches (daily/weekly) to reduce compute costs
* Relies on pattern matching and custom transformations in PySpark
* Avoids Amazon Comprehend costs by using pre-defined patterns
* Advantages: Significantly lower cost, simpler implementation
* Disadvantages: Not real-time, potentially less accurate redaction

## Scenario 5: Near Real-Time Redaction with Glue Streaming

```mermaid
graph TD
    subgraph "Application Source"
        APP[Application] -->|Generate logs| LOGS[Application Logs with PHI/PII]
        LOGS -->|Stream| KDS[Kinesis Data Stream]
    end
    
    subgraph "Stream Processing"
        KDS -->|Continuous processing| GLUE[AWS Glue Streaming ETL]
        GLUE -->|Redact PHI/PII| S3CLEAN[S3 Clean Bucket]
        GLUE -.->|Uses| REGEX[Regex Patterns]
    end
    
    subgraph "Security Lake Account"
        S3CLEAN -->|Ingest| SL[Security Lake]
        SL -->|Store redacted data| SLS3[Security Lake S3]
        SL -->|Catalog| GLUEC[AWS Glue Catalog]
        GLUEC -->|Query| ATHENA[Amazon Athena]
    end
    
    subgraph "Security Operations"
        ATHENA -->|Safe queries| ANALYST[Security Analyst]
        ATHENA -->|No PHI/PII exposure| DASH[Dashboards]
    end
```

### Scenario 5: Near Real-Time Streaming Redaction

* Uses AWS Glue Streaming ETL for continuous processing
* Processes logs with 2-3 minute latency
* Uses micro-batching with configurable window size
* Maintains cost advantage over Lambda+Comprehend approach
* Advantages: Near real-time processing, moderate cost, scalable
* Disadvantages: Higher cost than batch, requires Kinesis, more complex setup

# AWS Security Lake Pricing Breakdown

## Overview
AWS Security Lake is a security data lake service that centralizes security data from various AWS services, third-party sources, and custom sources into a purpose-built data lake stored in your account.

## Cost Components

### Free Resources
- **Security Lake service activation** - No charge for enabling the service
- **Security Lake console usage** - No charge for using the management console
- **AWS native source integration** - No charge for connecting AWS services as log sources
- **AWS Glue Data Catalog** - Tables created by Security Lake are free

### Resources That Incur Costs

#### 1. Data Ingestion and Storage
- **S3 Storage**: $0.023 per GB/month (Standard tier, US East-1)
  - All log data is stored in your S3 buckets
  - Costs vary by region and storage class
  - Data lifecycle policies can reduce costs by transitioning to cheaper storage tiers

- **Data Ingestion**: $0.50 per GB
  - Applies to all data ingested into Security Lake
  - Includes AWS native sources, custom sources, and third-party sources

#### 2. Data Transformation
- **AWS Glue ETL**: $0.44 per DPU-hour
  - Used to normalize data into the OCSF format
  - Minimum of 10-minute billing
  - Typically 2 DPUs per job

#### 3. Data Access and Query
- **Amazon Athena**: $5.00 per TB of data scanned
  - Used when querying Security Lake data
  - Compression and partitioning can reduce query costs
  - Consider using Athena workgroups with query result reuse

- **AWS Lake Formation**: No additional cost
  - Used for access control and permissions

#### 4. Data Sharing
- **Cross-account access**: No additional cost for sharing
- **Cross-region replication**: Standard S3 cross-region replication costs apply
  - $0.02 per GB for data transfer
  - Additional S3 storage costs in destination region

#### 5. Third-Party Integrations
- **AWS PrivateLink**: $0.01 per GB processed if used
- **API calls**: Standard AWS API pricing applies

## Cost Optimization Strategies

1. **Selective Source Configuration**
   - Only enable log sources you need for security analysis
   - Consider sampling high-volume logs

2. **Data Retention Policies**
   - Set appropriate retention periods for different log types
   - Use S3 Lifecycle policies to transition older data to cheaper storage tiers

3. **Query Optimization**
   - Use partitioning to reduce data scanned by Athena
   - Create and use table statistics
   - Compress data to reduce storage and query costs

4. **Regional Considerations**
   - Store data in regions with lower S3 costs when possible
   - Consider data sovereignty and compliance requirements

## Example Cost Calculation

For an organization ingesting 100GB of security data per day:

- **Monthly Data Ingestion**: 100GB × 30 days × $0.50/GB = $1,500
- **S3 Storage** (assuming 90-day retention): 100GB × 90 days × $0.023/GB = $207
- **Athena Queries** (assuming 10TB scanned/month): 10TB × $5.00/TB = $50
- **Glue ETL** (assuming 24 hours/day of processing with 2 DPUs): 24 hours × 30 days × 2 DPUs × $0.44/DPU-hour = $633.60

**Total Estimated Monthly Cost**: $2,390.60

## Additional Considerations

- Costs scale with data volume and retention periods
- Multi-account deployments multiply storage costs but centralize analysis
- Custom sources may require additional Lambda or Kinesis resources (billed separately)
- Actual costs will vary based on usage patterns and AWS region

*Note: All prices are examples based on US East-1 region. Actual prices may vary by region and are subject to change. Refer to the [AWS Pricing Calculator](https://calculator.aws) for the most current pricing information.*
