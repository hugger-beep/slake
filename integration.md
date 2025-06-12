
### Scenario 1: Centralized CloudTrail with Security Lake
``` mermaid

graph TD
    subgraph "AWS Organization"
        subgraph "Management Account"
            CT[Organization CloudTrail]
            S3C[Centralized S3 Bucket]
        end
        
        subgraph "Member Account 1"
            MA1[CloudTrail Service]
        end
        
        subgraph "Member Account 2"
            MA2[CloudTrail Service]
        end
        
        subgraph "Security Lake Account"
            SL[Security Lake]
            SLDB[(Security Lake Data Lake)]
        end
    end
    
    MA1 -->|Logs| CT
    MA2 -->|Logs| CT
    CT -->|Stores| S3C
    S3C -->|Collects from central location| SL
    SL -->|Normalizes & stores| SLDB

```
### Scenario 2: Distributed CloudTrail with Security Lake

``` mermaid
graph TD
    subgraph "AWS Organization"
        subgraph "Member Account 1"
            MA1[CloudTrail Service]
            S3A1[Local S3 Bucket]
        end
        
        subgraph "Member Account 2"
            MA2[CloudTrail Service]
            S3A2[Local S3 Bucket]
        end
        
        subgraph "Security Lake Account"
            SL[Security Lake]
            SLDB[(Security Lake Data Lake)]
        end
    end
    
    MA1 -->|Logs| S3A1
    MA2 -->|Logs| S3A2
    S3A1 -->|Direct collection from service| SL
    S3A2 -->|Direct collection from service| SL
    MA1 -->|Direct API collection| SL
    MA2 -->|Direct API collection| SL
    SL -->|Normalizes & stores| SLDB
```

## VPC Flow Logs Integration with Security Lake
``` mermaid 
graph TD
    subgraph "AWS Organization"
        subgraph "Management/Log Archive Account"
            S3C[Centralized S3 Bucket]
        end
        
        subgraph "Member Account 1"
            VPC1[VPC Flow Logs]
        end
        
        subgraph "Member Account 2"
            VPC2[VPC Flow Logs]
        end
        
        subgraph "Security Lake Account"
            SL[Security Lake]
            SLDB[(Security Lake Data Lake)]
        end
    end
    
    VPC1 -->|Logs| S3C
    VPC2 -->|Logs| S3C
    S3C -->|Collects from central location| SL
    SL -->|Normalizes & stores| SLDB
```
