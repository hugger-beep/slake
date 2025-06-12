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
