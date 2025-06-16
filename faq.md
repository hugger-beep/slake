## Historical Data Handling

### Custom Source Configuration
- When configuring Security Lake to use an existing centralized log bucket (e.g., CloudTrail bucket)
- Security Lake only processes new logs going forward from the time of configuration
- Historical data in the bucket (even large volumes like 10TB spanning multiple years) is not automatically processed
- Security Lake does not perform a backfill or initial scan of existing data
- The historical data remains in the bucket but is not normalized or ingested into Security Lake

### Real-Time Data Identification
- Security Lake automatically configures S3 event notifications during custom source setup
- No manual configuration of S3 event notifications is required for the bucket
- For complex buckets with many prefixes (e.g., 200+ folders with 10TB of data), Security Lake does not scan all folders
- It uses object creation timestamps and metadata to identify new data without examining every existing file
- You can specify exact prefixes for Security Lake to monitor, limiting the scope to only relevant paths
- This approach avoids unnecessary S3 LIST or GET operations against your entire bucket structure
- No initial scanning costs are incurred for historical data, and processing costs only apply to new data

### Custom Source Processing Flow
- When adding a raw centralized CloudTrail bucket as a custom source:
  1. Security Lake creates an S3 event notification on your source bucket
  2. The notification triggers a Security Lake-managed Lambda function when new objects are created
  3. The Lambda function reads the new logs from your source bucket
  4. It converts the logs to OCSF format and normalizes them
  5. The processed logs are then stored in the Security Lake bucket under the `/ext/` prefix
- The original raw logs remain in your source bucket untouched
- Security Lake requires appropriate permissions (via IAM roles) to:
  1. Configure event notifications on your source bucket
  2. Read objects from your source bucket
  3. Write processed data to the Security Lake bucket

### Accessing Historical Data
- To analyze historical data, use other AWS services like Athena directly against the S3 bucket
- Custom solutions would be required to process historical data into Security Lake's format
- There is no built-in mechanism to backfill historical data into Security Lake

## Implications for Log Filtering

When configuring filtering at the source service level (e.g., CloudTrail event selectors or VPC Flow Logs traffic type):

1. **CloudTrail and Security Hub**: Since Security Lake has direct API integration, it will only receive the events/findings that pass through the filters configured in these services.

2. **VPC Flow Logs, WAF, and Route 53**: Since Security Lake reads from the storage destination, it will only see logs that were stored after filtering was applied at the source.


### Global Events Handling
- Security Lake treats Security Hub findings as regional data, even for global services
- For global services (like IAM, CloudFront, Route 53), Security Hub findings are collected from the region where they are generated
- To collect global findings, Security Lake must be configured in the regions where Security Hub aggregates these findings
- In multi-region deployments, Security Lake will collect the same global finding only once from its originating region

### Global Service Resilience
- By default, CloudTrail logs for global services (like IAM) are only delivered to us-east-1
- If us-east-1 experiences an outage, IAM events generated in other regions (e.g., ca-central-1) would not be recorded until us-east-1 recovers
- For resilience, enable multi-region CloudTrail with "Include global service events" in additional regions
- This configuration creates redundancy, allowing Security Lake to collect global service events from alternative regions during outages

### Centralized vs. Distributed Collection
- **Centralized CloudTrail**: Security Lake can collect from the centralized CloudTrail configuration
- **Distributed CloudTrail**: Security Lake connects to each account's CloudTrail service directly

### Direct API vs. Centralized Bucket Approach

 **Direct API Integration** (recommended):
- 
  - Optimized for real-time log collection with lower latency
  - Simplified architecture with fewer components and potential points of failure
  - Cost-efficient by eliminating S3 GET request costs and reducing Lambda processing
  - Ensures consistent OCSF schema handling and automatic updates
  - Managed by AWS with automatic updates when services evolve
    
 **Centralized Bucket as Custom Source**:
  
  - May be preferred when existing analytics are already configured against the centralized bucket
  - Useful when organizations need to maintain the exact same data in both original and Security Lake formats
  - Can support specific data governance requirements that mandate a single copy of CloudTrail logs
  - Requires additional configuration and permissions management
  - May incur additional costs for S3 operations and Lambda processing


## How Security Lake Discovers New Data

Security Lake doesn't use a traditional Glue crawler to discover new data. Instead, it uses a combination of techniques to keep track of new files:

1. **Partition Projection**: Security Lake uses partition projection in the Glue Data Catalog to automatically recognize the partitioning structure (`region=<region>/accountid=<account-id>/dt=<YYYY-MM-DD>/hr=<HH>/`). This means it doesn't need to crawl the data to discover new partitions - it can infer them based on the partition structure.

2. **S3 Event Notifications**: Security Lake sets up S3 event notifications to be alerted when new data is written to the bucket. When your Glue job writes new Parquet files to the target location, S3 events trigger Security Lake to update its metadata.

3. **Manifest Files**: In some cases, Security Lake uses manifest files to track which data has been processed. These are small metadata files that list the data files that have been added.

4. **Periodic Scans**: Security Lake also performs periodic scans of the S3 bucket to discover any new data that might have been missed by the event notifications.

### Using Existing Centralized CloudTrail Logs

In a Control Tower organization where CloudTrail logs are already centralized in the logging account's S3 bucket, you can leverage these existing logs for Security Lake without enabling CloudTrail collection directly in Security Lake:

1. **Create a Glue ETL job** that:
   - Reads CloudTrail logs from your existing centralized S3 bucket
   - Transforms them to OCSF format
   - Writes them to a target location with the Security Lake partitioning structure

2. **Configure the job parameters**:
   - `source_bucket`: Your existing CloudTrail logs bucket in the logging account
   - `source_prefix`: The prefix where organization CloudTrail logs are stored
   - `target_bucket`: A bucket where you want to store the OCSF-formatted data
   - `target_prefix`: "ext/ct1" or another prefix of your choice

3. **Schedule the Glue job** to run regularly (e.g., hourly) to process new CloudTrail logs as they arrive

4. **Add a custom source in Security Lake**:
   - Go to the Security Lake console
   - Navigate to "Custom sources"
   - Click "Add custom source"
   - Enter a name (e.g., "Organization CloudTrail")
   - For source location, select the target S3 bucket and prefix where your Glue job writes the OCSF data
   - Select "API Activity" as the OCSF class
   - Select "Parquet" as the format

5. **Grant necessary permissions**:
   - Ensure the Glue job has read access to the source CloudTrail bucket
   - Ensure the Glue job has write access to the target bucket
   - Ensure Security Lake has read access to the target bucket

This approach:
- Leverages your existing CloudTrail setup without duplicating data collection
- Converts the logs to OCSF format for better integration with Security Lake
- Preserves the multi-account context in the OCSF data
- Avoids the need to enable CloudTrail collection directly in Security Lake


## What Happens When You Add a Custom Source to Security Lake

When you add a custom source via the Security Lake console and point it to your S3 bucket with OCSF files, Security Lake will:

1. **Register the data location**: Security Lake will record the S3 bucket and prefix (`ext/ct1`) as a valid data source.

2. **Create metadata tables**: Security Lake will create metadata tables in AWS Glue Data Catalog that map to your OCSF data, making it queryable.

3. **Set up partitioning**: Security Lake will recognize the partitioning structure in your data and set up partition projections for efficient querying.

4. **Enable querying**: The data will become available for querying through:
   - Amazon Athena
   - Security Lake's query interface
   - Any subscribers you've configured

5. **Apply data lifecycle policies**: Security Lake will apply any configured retention policies to your data.

6. **Monitor for new data**: Security Lake will periodically check for new data in your S3 location and automatically update its metadata tables.

7. **Make data available to subscribers**: If you've configured subscribers, they'll be able to access this data through Security Lake's subscription mechanisms.

## Requirements for Custom Source Data

For this to work properly:

### 1. Ensure proper partitioning

Your data should be written with the partition structure:
```
s3://your-bucket/ext/ct1/region=<region>/accountid=<account-id>/dt=<YYYY-MM-DD>/hr=<HH>/
```

### 2. Maintain OCSF schema

Your data must follow the OCSF schema you specified (API Activity class 3002).

### 3. Use Parquet format

Files must be in Parquet format for optimal performance.

### 4. Include required fields

All required OCSF fields must be present in your data, including:
- `class_uid`: 3002 for API Activity
- `category_uid`: 3 for Activity events
- `activity_id`: 1
- `type_uid`: 1 for general API activity
- Time fields and partition fields
  
https://github.com/ocsf/examples/tree/main/mappings/markdown/AWS/v1.1.0/CloudTrail
