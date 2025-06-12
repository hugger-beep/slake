
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
### Scenario 2: Distributed VPC Flow Logs with Security Lake

``` mermaid

graph TD
    subgraph "AWS Organization"
        subgraph "Member Account 1"
            VPC1[VPC Flow Logs]
            CW1[CloudWatch Logs]
            S3A1[Local S3 Bucket]
        end
        
        subgraph "Member Account 2"
            VPC2[VPC Flow Logs]
            CW2[CloudWatch Logs]
            S3A2[Local S3 Bucket]
        end
        
        subgraph "Security Lake Account"
            SL[Security Lake]
            SLDB[(Security Lake Data Lake)]
        end
    end
    
    VPC1 -->|Option 1| CW1
    VPC1 -->|Option 2| S3A1
    VPC2 -->|Option 1| CW2
    VPC2 -->|Option 2| S3A2
    
    CW1 -->|Direct collection| SL
    S3A1 -->|Direct collection| SL
    CW2 -->|Direct collection| SL
    S3A2 -->|Direct collection| SL
    
    SL -->|Normalizes & stores| SLDB
```

### CloudTrail Filtering and Security Lake Collection

``` mermaid

graph TD
    subgraph "CloudTrail Configuration"
        CT[CloudTrail]
        Filter[Event Filter: Only S3 Delete Events]
    end
    
    subgraph "S3 Events"
        Create[S3 Create Events]
        Read[S3 Read Events]
        Delete[S3 Delete Events]
    end
    
    subgraph "Security Lake"
        SL[Security Lake]
        SLDB[(Security Lake Data Lake)]
    end
    
    Create -->|Generated| CT
    Read -->|Generated| CT
    Delete -->|Generated| CT
    
    CT -->|Applies| Filter
    Filter -->|Only passes| Delete
    Delete -->|Collected| SL
    SL -->|Stores| SLDB
```

### VPC Flow Logs Filtering and Security Lake Collection
``` mermaid

graph TD
    subgraph "VPC Flow Logs Configuration"
        VPC[VPC Flow Logs]
        Filter[Filter: Only REJECT Traffic]
    end
    
    subgraph "Network Traffic"
        Accept[ACCEPT Traffic]
        Reject[REJECT Traffic]
    end
    
    subgraph "Destination"
        CW[CloudWatch Logs]
        S3[S3 Bucket]
    end
    
    subgraph "Security Lake"
        SL[Security Lake]
        SLDB[(Security Lake Data Lake)]
    end
    
    Accept -->|Generated| VPC
    Reject -->|Generated| VPC
    
    VPC -->|Applies| Filter
    Filter -->|Only passes| Reject
    
    Reject -->|Stored in| CW
    Reject -->|Stored in| S3
    
    CW -->|Reads configuration & collects| SL
    S3 -->|Reads configuration & collects| SL
    
    SL -->|Stores| SLDB
```


```mermaid
graph TD
    subgraph "AWS Services"
        CT[CloudTrail]
        SH[Security Hub]
        VPC[VPC Flow Logs]
        WAF[AWS WAF]
        R53[Route 53]
    end
    
    subgraph "Storage Services"
        CWL[CloudWatch Logs]
        S3[S3 Buckets]
    end
    
    subgraph "Security Lake"
        SL[Security Lake Service]
        SLDB[(Security Lake Data Lake)]
    end
    
    %% Direct API Integration
    CT -->|Direct API Integration| SL
    SH -->|Direct API Integration| SL
    
    %% Indirect Collection via Storage
    VPC -->|Logs| CWL
    VPC -->|Logs| S3
    WAF -->|Logs| S3
    R53 -->|Logs| CWL
    
    CWL -->|Reads via AWS APIs| SL
    S3 -->|Reads via AWS APIs| SL
    
    SL -->|Normalizes & stores| SLDB
    
    %% Styling
    classDef directAPI fill:#f9f,stroke:#333,stroke-width:2px
    classDef indirectAPI fill:#f96,stroke:#333,stroke-width:2px
    classDef storage fill:#bfb,stroke:#333,stroke-width:2px
    classDef securityLake fill:#bbf,stroke:#333,stroke-width:2px
    
    class CT,SH directAPI
    class VPC,WAF,R53 indirectAPI
    class CWL,S3 storage
    class SL,SLDB securityLake
```

