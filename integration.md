
Scenario 1: Centralized CloudTrail with Security Lake
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
