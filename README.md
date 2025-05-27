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
  - Data collection agents and configurations

The delegated administrator account hosts most of the centralized infrastructure, while member accounts contain only the necessary resources for collecting and forwarding logs to the central repository.

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
