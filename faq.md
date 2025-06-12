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
