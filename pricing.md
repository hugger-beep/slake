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
  - Processing fee charged when data first enters Security Lake
  - Applies to all data ingested into Security Lake (AWS native, third-party, and custom sources)
  - Charged only once when data is initially ingested, regardless of retention period
  - Charged in addition to S3 storage costs
  - Represents the cost of collecting, normalizing, and organizing the data
  - Often the largest cost component for Security Lake implementations

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
  - $0.02 per GB for data transfer between regions
  - Additional S3 storage costs in destination region (you pay for storing the same data twice)
  - Applies when using Security Lake's rollup region feature (e.g., replicating from us-east-2 to us-east-1)
  - Costs accumulate for each region you replicate to
  - No additional ingestion fee ($0.50/GB) is charged for the replicated data
  - Consider selective replication of only critical log sources to minimize costs

#### 5. Third-Party Integrations
- **AWS PrivateLink**: $0.01 per GB processed if used
- **API calls**: Standard AWS API pricing applies

## Cost Optimization Strategies

1. **Selective Source Configuration**
   - Only enable log sources you need for security analysis
   - Consider sampling high-volume logs (e.g., VPC Flow Logs)
   - Filter logs at the source when possible (e.g., CloudTrail management events only)
   - Start with critical sources and add others incrementally as needed
   
   **Implementation:**
   - Prioritize sources based on security value: CloudTrail > VPC Flow Logs > Route 53 > S3 Data Events
   - Use the Security Lake console to selectively enable only required log sources
   - For CloudTrail, consider enabling management events only and excluding high-volume read-only events
   - For VPC Flow Logs, use sampling rates (10-50%) in high-traffic networks
   - Evaluate each source against your security use cases and compliance requirements
   
   **Example:** A company initially enabled all log sources, generating 500GB/day at $250/day in ingestion costs. By disabling Route 53 logs and sampling VPC Flow Logs at 50%, they reduced volume to 300GB/day, saving $100/day ($3,000/month) in ingestion costs.

2. **Data Retention Policies**
   - Set appropriate retention periods for different log types
   - Use S3 Lifecycle policies to transition older data to cheaper storage tiers
   - Consider shorter retention for high-volume, low-value logs
   - Implement tiered retention (e.g., 30 days in S3 Standard, 90 days in S3 IA, 1 year in Glacier)
   
   **Implementation:**
   - Configure retention periods in Security Lake console for each log source
   - Manually create S3 Lifecycle rules directly on the S3 buckets created by Security Lake:
     * 0-30 days: S3 Standard ($0.023/GB/month)
     * 31-90 days: S3 Intelligent-Tiering or S3 Standard-IA ($0.0125/GB/month)
     * 91-365 days: S3 Glacier Instant Retrieval ($0.004/GB/month)
     * 365+ days: S3 Glacier Deep Archive ($0.00099/GB/month)
   - For compliance-critical logs (CloudTrail, Config), maintain longer retention
   - For operational logs (VPC Flow, Route 53), consider shorter retention periods
   
   **Impact on Historical Data Access:**
   - Data in S3 Standard and S3 Standard-IA remains immediately queryable via Athena
   - Data in Glacier storage classes (including Instant Retrieval) requires restoration before querying with Athena
   - Plan for restoration time and costs when using Glacier storage classes
   - Consider creating periodic summary tables of frequently needed historical data and storing them in Standard storage
   
   **Example:** For 100GB/day of logs with a flat 1-year retention in S3 Standard ($0.023/GB/month), storage costs would be approximately $839.50/month. With tiered storage (30 days Standard, 90 days IA, 245 days Glacier), costs drop to approximately $383.25/month—a 54% savings.

3. **Query Optimization**
   - Use partitioning to reduce data scanned by Athena
   - Create and use table statistics
   - Compress data to reduce storage and query costs
   - Save and reuse query results in Athena workgroups
   - Create views for commonly used queries to optimize repeated analysis
   
   **Implementation:**
   - Security Lake automatically partitions data by date, region, and source
   - Further optimize queries by:
     * Always including date filters in WHERE clauses (e.g., `time_dt BETWEEN x AND y`)
     * Using LIMIT clauses when exploring data
     * Creating standard views (not materialized) for common security reports
     * Configuring Athena workgroups with query result reuse enabled
     * Setting up result caching with a TTL appropriate for your use case
   - Use column-level statistics in AWS Glue Data Catalog for better query planning
   - Convert frequently used queries into views to standardize filtering
   - For regular reports, schedule queries and store results in a separate, smaller table
   
   **SQL Example:**
   ```sql
   -- Inefficient query (scans all data)
   SELECT * FROM security_lake_table WHERE action = 'BLOCK';
   
   -- Optimized query (uses partitioning)
   SELECT * FROM security_lake_table 
   WHERE time_dt BETWEEN CURRENT_DATE - INTERVAL '7' DAY AND CURRENT_DATE
   AND region = 'us-east-1'
   AND action = 'BLOCK'
   LIMIT 1000;
   ```
   
   **Example:** A security team running daily queries scanning 1TB each time (30TB/month) at $5/TB would spend $150/month. By implementing partitioning by date and saving query results, they reduced scan volume by 70% to 9TB/month, lowering costs to $45/month.

4. **Regional Considerations**
   - Store data in regions with lower S3 costs when possible
   - Limit cross-region replication to only essential log sources
   - Use a single rollup region instead of multiple replications
   - Consider data sovereignty and compliance requirements
   
   **Implementation:**
   - When configuring Security Lake, select a primary region with lower S3 costs
   - For multi-region deployments:
     * Use the "rollup region" feature for centralized analysis
     * Note that rollup region replicates ALL log sources (cannot select specific sources)
     * Consider enabling fewer log sources in non-primary regions to reduce replication costs
     * Be strategic about which regions have Security Lake enabled
   - Create region-specific Athena views that union data across regions when needed
   - Consider using AWS Organizations integration to centrally manage Security Lake
   
   **Regional Strategy Example:**
   * Primary Security Operations: us-east-1 (rollup region)
   * Secondary Regions: us-west-2, eu-west-1, ap-southeast-1
   * In secondary regions: Enable only critical log sources to minimize replication costs
   * All enabled sources will replicate to rollup region automatically
   
   **Example:** An organization replicating 50GB/day of logs from 3 regions to a central region was paying: 50GB × 3 regions × $0.02/GB × 30 days = $90/month in transfer costs, plus duplicate storage costs. By replicating only CloudTrail and GuardDuty logs (10GB/day), they reduced costs to $18/month in transfer fees.

5. **Subscriber Model Optimization**
   - Use cross-account access instead of duplicating data
   - Share read-only access with security teams rather than creating copies
   - Implement access controls through Security Lake's subscriber model
   - Choose the appropriate access method for each subscriber (direct query OR notification-based)
   
   **Example:** A MSSP initially requested copies of all security data (100GB/day) to their own account, requiring the customer to pay ingestion costs twice. By implementing a subscriber model with direct query access, they eliminated the duplicate $0.50/GB ingestion fee, saving $1,500/month.

6. **Operational Efficiency**
   - Monitor usage patterns and adjust configurations accordingly
   - Use S3 Storage Lens and AWS Cost Explorer to track Security Lake storage growth
   - Implement automated cost anomaly detection
   - Consider AWS Budgets to set alerts for unexpected cost increases
   
   **Implementation:**
   - Set up S3 Storage Lens dashboard focused on Security Lake buckets
   - Create custom AWS Cost Explorer reports with Security Lake resource tags
   - Configure AWS Budgets with alerts for:
     * Monthly Security Lake S3 storage costs
     * Athena query usage exceeding thresholds
     * Unexpected increases in data transfer costs
   - Use AWS Cost Anomaly Detection to identify unusual spending patterns
   - Schedule regular Athena queries to analyze log volume by source:
   
   ```sql
   -- Query to monitor log volume by source
   SELECT 
     DATE_TRUNC('day', time_dt) AS day,
     metadata.product.name AS source,
     COUNT(*) AS event_count,
     SUM(LENGTH(CAST(observables AS VARCHAR)))/1024/1024 AS approx_size_mb
   FROM security_lake_table
   WHERE time_dt BETWEEN CURRENT_DATE - INTERVAL '30' DAY AND CURRENT_DATE
   GROUP BY 1, 2
   ORDER BY 1 DESC, 4 DESC;
   ```
   
   **Example:** A company set up AWS Budgets to monitor Security Lake costs with a $1,000 monthly threshold. After a new application deployment, their daily Cost Explorer report showed S3 storage growing 3x faster than normal. They ran the volume analysis query and discovered WAF logs had increased dramatically. By adjusting WAF logging configuration at the source (in AWS WAF console) to enable sampling and filter out lower-risk traffic patterns within 24 hours, they avoided a potential $3,000 overage that month.

**WAF Log Volume Control:**
- Security Lake automatically ingests all WAF logs that are enabled at the source
- To reduce WAF log volume in Security Lake:
  1. In the AWS WAF console, modify your Web ACLs' logging configuration
  2. Enable sampling to capture only a percentage of requests (e.g., 5% or 10%)
  3. Disable WAF logging for less critical web ACLs or development environments
  4. Selectively disable the WAF source in Security Lake for specific regions
  5. Note: WAF doesn't support filtering logs by IP or path before sending to destinations

**Important:** Unlike CloudWatch Logs or S3 destinations, you cannot selectively filter which WAF logs go to Security Lake. When WAF logging is enabled and Security Lake is configured to collect WAF logs, all logs will flow to Security Lake. The only controls available are at the WAF level (sampling) or completely disabling the WAF source in Security Lake.

7. **Query Architecture**
   - Use Athena federated queries instead of duplicating data
   - Consider Amazon QuickSight for dashboards instead of repeated queries
   - Implement caching for frequently accessed reports
   
   **Example:** A security team running the same 10 compliance reports weekly was scanning 5TB per week (20TB/month) at $100/month. By implementing QuickSight with SPICE caching for these reports, they reduced Athena query volume by 80%, saving $80/month while providing faster dashboard access.

## Example Cost Calculation - an estimation

For an organization ingesting 100GB of security data per day:

- **Monthly Data Ingestion**: 100GB × 30 days × $0.50/GB = $1,500
- **S3 Storage** (assuming 90-day retention): 100GB × 90 days × $0.023/GB = $207
- **Athena Queries** (assuming 10TB scanned/month): 10TB × $5.00/TB = $50
- **Glue ETL** (assuming 4 hours/day of processing with 2 DPUs): 4 hours × 30 days × 2 DPUs × $0.44/DPU-hour = $105.60

**Total Estimated Monthly Cost**: $1,862.60

## Additional Considerations

- Costs scale with data volume and retention periods
- Multi-account deployments multiply storage costs but centralize analysis
- Custom sources may require additional Lambda or Kinesis resources (billed separately)
- Actual costs will vary based on usage patterns and AWS region

*Note: All prices are examples based on US East-1 region. Actual prices may vary by region and are subject to change. Refer to the [AWS Pricing Calculator](https://calculator.aws) for the most current pricing information.*
