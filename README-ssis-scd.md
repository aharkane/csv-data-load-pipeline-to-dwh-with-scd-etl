# SSIS Product Data Pipeline

SSIS ETL implementation with Slowly Changing Dimension Type 2 for product catalog management.

## Overview

ETL solution featuring two processing approaches for product data: row-by-row direct processing and batch multi-file operations. Implements SCD Type 2 for complete historical tracking of product changes (name, price, new products).

## Architecture

**Dual-Package Design:**

```
Package 1: Row-by-Row Processing
CSV → Data Flow → Conditional Split → Direct Updates

Package 2: Batch Processing  
CSV Files → Foreach Loop → Staging → SQL Batch SCD
```

## Key Features

- SCD Type 2 with IsCurrent flag and validity dates
- Change detection for names, prices, and new products
- Row-by-row and batch processing approaches
- Historical tracking with audit trail
- Automated file archival

## Technology Stack

| Component | Technology |
|-----------|------------|
| **Platform** | SQL Server Integration Services |
| **Version** | SSIS 17.0 |
| **Storage** | SQL Server dimensional tables |
| **Source** | CSV files |

## SCD Type 2 Implementation

**Historical Tracking:**
- IsCurrent flag (0=Historical, 1=Current)
- ValidityDate_Start and ValidityDate_End
- 9999-12-31 for active records

**Processing:**
- Name/price changes: Close old record, insert new version
- New products: Insert with IsCurrent=1
- Unchanged: Skip processing

## Skills Demonstrated

<table>
<tr>
<td width="55%" valign="top">

**SSIS Development**
- Data Flow Task design
- Control Flow containers
- Foreach Loop implementation
- Execute SQL Task

**ETL Patterns**
- Staging layer architecture
- Change detection algorithms
- Row-by-row vs batch processing
- File processing automation

</td>
<td width="45%" valign="top">

**SCD Implementation**
- SCD Type 2 logic
- Historical tracking
- IsCurrent flag management
- Validity date handling

**Data Transformations**
- Lookup transformations
- Conditional Split
- Union All operations
- Derived Column calculations

</td>
</tr>
</table>