# SSIS ETL Packages: Products Data Pipeline

Advanced SQL Server Integration Services (SSIS) implementation showcasing enterprise-grade extract, transform, and load operations for product catalog management with slowly changing dimension handling.

---

## Table of Contents

1. [Overview](#overview)
2. [Technical Skills Demonstrated](#technical-skills-demonstrated)
3. [Project Architecture](#project-architecture)
4. [Package 1: Products RowByRow ETL](#package-1-products-rowbyrow-etl)
5. [Package 2: Products Loop ETL](#package-2-products-loop-etl)
6. [Data Flow Design Patterns](#data-flow-design-patterns)
7. [Key Transformations](#key-transformations)
8. [Implementation Highlights](#implementation-highlights)
9. [Data Processing Workflow](#data-processing-workflow)

---

## Overview

This project demonstrates a comprehensive ETL solution built with SQL Server Integration Services (SSIS) for managing product data lifecycle. The implementation includes two distinct approaches to data processing: a row-by-row streaming pipeline and a multi-file loop processing engine, both implementing slowly changing dimension (SCD) Type 2 methodology for handling product data changes over time.

The solution processes product information including categories, subcategories, pricing, sizing, and validity dates—transforming raw flat file data into a clean, dimensional data warehouse structure with full change tracking and historical audit trails.

---

## Technical Skills Demonstrated

### SSIS Fundamentals

- **Package Development & Design**: Created production-ready SSIS packages with version control (v17.0.1008.3)
- **Data Flow Tasks**: Implemented complex pipeline architectures with multiple source-to-destination transformations
- **Control Flow**: Designed container hierarchies with nested loops and conditional execution logic
- **Variables & Expressions**: Utilized parameterized variables for dynamic configuration and file path management
- **Connection Management**: Configured OLE DB connections for SQL Server staging and destination tables

### Data Transformation Techniques

- **Derived Column Transformations**: Created calculated fields including change tracking (`IsCurrent`) columns and date trimming operations
- **Conditional Split Logic**: Implemented branch-based routing to separate new inserts from product updates
- **Lookup Transformations**: Performed dimensional lookups to identify existing product IDs and detect changes
- **Union All Operations**: Merged multiple data streams from different transformation branches
- **Row Sampling & Counting**: Implemented row count metrics and change detection filtering
- **OLE DB Command Execution**: Direct SQL execution for handling product updates including price and name changes

### Advanced ETL Patterns

- **Slowly Changing Dimension Type 2**: Full historical tracking with `IsCurrent` flags and validity date ranges (`ValidityDate_Start`, `ValidityDate_End`)
- **Change Data Detection**: Sophisticated logic to identify which products have been modified (name changes, price updates)
- **Incremental Processing**: Row counting and staging mechanisms for monitoring data flow volumes
- **File-Based Looping**: Foreach enumerator pattern for processing multiple CSV source files in sequence

### SQL Server Integration

- **Staging Tables**: Implemented multi-layer staging architecture for data validation and error handling
- **Destination Mapping**: Direct OLE DB destination writes with column mapping and data type conversions
- **Audit Trails**: Captured row counts and processing metrics for data quality monitoring
- **Dimension Tables**: Created and maintained slowly changing dimension tables with historical records

### Data Quality & Validation

- **Data Type Handling**: Proper conversion of dates, strings, decimals (pricing), and dimensional keys
- **Null Handling**: Conditional routing for records with missing category/subcategory data
- **Date Standardization**: Trimming and formatting of SellStartDate, SellEndDate, and DiscontinuedDate fields
- **Dimensional Integrity**: Lookup-based validation ensuring referential integrity with product categories

### Development Environment

- **Version Control**: SSIS projects ready for Git/GitHub integration and collaborative development
- **Documentation**: Comprehensive task naming and description conventions for maintainability
- **Performance Optimization**: Designed for both row-by-row precision and bulk processing efficiency

---

## Project Architecture

### High-Level Design Pattern

```
CSV Source Data
       ↓
   [Parser/Validation]
       ↓
   [Transformation Pipeline]
   ├── Derived Columns (IsCurrent, Date Trim)
   ├── Lookup (Product ID Matching)
   ├── Conditional Split (Insert vs. Update)
   └── Union All (Merge Streams)
       ↓
[Staging Layer]
       ↓
[Destination Tables]
   ├── Products DB Table (Final Warehouse)
   └── Staging Table (Intermediate)
       ↓
[SQL Scripts]
   ├── SCD Type 2 Processing
   ├── Row Count Metrics
   └── Table Maintenance
```

### Processing Approaches

| Aspect | Package 1 (RowByRow) | Package 2 (Loop) |
|--------|----------------------|------------------|
| **Processing Model** | Single Data Flow Task | Foreach Loop Container |
| **Input Source** | Single CSV File | Multiple CSV Files |
| **Scaling** | Row-by-row precision | Batch processing efficiency |
| **Iteration** | Direct execution | Enumerated file loop |
| **Data Staging** | Direct to destination | Multi-layer staging |
| **Change Handling** | In-pipeline splitting | Staging + SQL post-processing |

---

## Package 1: Products RowByRow ETL

### Purpose

High-fidelity, row-by-row ETL pipeline designed for precision processing of product updates with immediate change detection and routing.

### Package Metadata

- **Name**: Products RowByRow ETL
- **Creation Date**: August 7, 2025
- **SSIS Version**: 17.0.1008.3
- **Execution Type**: Single Data Flow Task (Microsoft.Pipeline)

### Data Flow Architecture

#### 1. **Products Source** (FlatFileSource)
- **Input**: Flat file (CSV) containing product records
- **Purpose**: Primary data ingestion point
- **Output**: Raw product dataset with all dimensional attributes

#### 2. **Trim Dates** (Derived Column - #1)
- **Input**: Raw product data from source
- **Transformation**: Date field standardization and trimming
- **Applies to**: SellStartDate, SellEndDate, DiscontinuedDate fields
- **Output**: Cleansed date columns

#### 3. **Lookup for New ProductID** (Lookup)
- **Purpose**: Cross-reference with existing product dimension
- **Logic**: Matches current products against historical product master
- **Output**: New product ID flag or existing reference ID
- **Handles**: Detection of new vs. existing products

#### 4. **Add IsCurrent Column** (Derived Column - #2)
- **Logic**: Creates slowly changing dimension Type 2 flag
- **Expression**: IsCurrent = 1 (for all rows - marking as current version)
- **Complements**: ValidityDate_Start and ValidityDate_End for historical tracking
- **Output**: Dimensional flag for temporal tracking

#### 5. **Split Based on Changes** (Conditional Split)
- **Routing Logic**: 
  - **Change Stream**: Records with ProductName or UnitPrice modifications
  - **No Change Stream**: Records without detected changes
- **Uses**: Lookup results and field comparison conditions
- **Output**: Two branched streams for differential processing

#### 6. **Change Processing Branch**
- **ProductName Change** (OLE DB Command): Direct SQL UPDATE for name modifications
- **UnitPrice Change** (OLE DB Command): Direct SQL UPDATE for price modifications
- **Execution**: Direct command execution against destination database

#### 7. **No Change Processing Branch**
- **No changes Row Count** (Row Sampling): Counting and sampling unchanged records
- **Purpose**: Metrics collection for audit trails

#### 8. **Union All** (UnionAll)
- **Merge Logic**: Combines both change and no-change processing branches
- **Purpose**: Unified stream before final destination write

#### 9. **OLE DB Destination** (OLEDBDestination)
- **Target Table**: Products warehouse table
- **Mode**: Bulk insert of processed records
- **Output**: Final dimensional table with all transformations applied

### Transformation Logic Summary

```
Raw CSV → Date Trim → Lookup → IsCurrent Flag → Split Changes
                                                  ├→ UPDATE Changes → Union All → Destination
                                                  └→ Count No-Changes
```

---

## Package 2: Products Loop ETL

### Purpose

Enterprise-scale ETL pipeline with foreach looping capability, processing multiple CSV source files sequentially with comprehensive staging and SQL post-processing for slowly changing dimension implementation.

### Package Metadata

- **Name**: Products Loop ETL
- **Creation Date**: August 7, 2025
- **SSIS Version**: 17.0.1008.3
- **Variables**: 2 user-defined parameters

### User Variables

| Variable Name | Type | Purpose | Example Value |
|---------------|------|---------|---|
| `FileNamePath` | String | Root directory for CSV source files | `D:\...\SSIS\1. Products Data Flow\Data` |
| `RowCountVar` | Integer | Counter for row-level metrics | 0 (initialized) |

### Control Flow Structure

#### 1. **Products Csv Source Foreach Loop** (Foreach Loop Container)

**Enumerator Configuration**:
- **Type**: ForEachFileEnumerator
- **Source Folder**: Points to `FileNamePath` variable
- **File Pattern**: Wildcard pattern `*.csv` (all CSV files)
- **Recursion**: Disabled (same directory only)
- **Iteration**: Sequential processing of matched files

**Nested Components** (within loop):

##### a) **Copying Processed Files** (Script Task/Copy Operation)
- **Purpose**: Archive processed CSV files to separate folder
- **Destination**: `ProcessedFilesPath_var` (typically `/Processed` subdirectory)
- **Execution**: Post-processing cleanup after successful load

##### b) **Data Flow Task** (Embedded within loop)
- **Structure**: Similar to Package 1 with modifications
- **Components**:
  - **Products Source**: Reads current enumerated CSV file
  - **Trim Dates**: Date standardization (identical to Package 1)
  - **Lookup for New ProductID**: Dimensional lookup (identical to Package 1)
  - **Add IsCurrent Column 1**: Slowly changing dimension flag
  - **Split Based on Changes 1**: Change detection routing
  - **Staging DB Table** (OLEDBDestination): Loads to intermediate staging table
  - **Products DB Table** (OLEDBDestination): Optional direct load to final dimension

#### 2. **Truncate Staging Table** (Execute SQL Task)
- **Execution Order**: Pre-loop initialization
- **SQL Operation**: `TRUNCATE TABLE [StagingTable]`
- **Purpose**: Clear staging layer before first file processing
- **Error Handling**: Ensures fresh state for loop iterations

#### 3. **Staging Table Row Count** (Execute SQL Task)
- **Execution Timing**: Post-loop completion
- **Query**: Row count audit query against staging table
- **Output**: Captures total records processed across all files
- **Logging**: Stored in metrics/audit table or execution log

#### 4. **Slowly Changing Dim Script** (Execute SQL Task)
- **Execution**: Final post-processing step
- **SQL Implementation**: Slowly Changing Dimension Type 2 logic
- **Operations**:
  - Mark existing product versions as inactive (`IsCurrent = 0`)
  - Set old record `ValidityDate_End` to current date
  - Insert new/changed products as current versions
  - Set new record `ValidityDate_Start` to current date
  - Maintain complete historical audit trail
- **Target**: Products warehouse dimension table

### Transformation Logic Summary

```
TRUNCATE Staging Table
        ↓
ForEach CSV File:
├── Read CSV via Products Source
├── Trim Dates
├── Lookup Product ID
├── Add IsCurrent Flag
├── Split Changes
├── Load to Staging Table
└── Copy CSV to Processed Folder
        ↓
Get Staging Table Row Count
        ↓
Execute SCD Type 2 SQL Script
        ↓
Update Products Dimension Table
```

---

## Data Flow Design Patterns

### Pattern 1: Source → Validation → Transformation → Destination

All packages implement this foundational ETL pattern with quality gates at each stage.

### Pattern 2: Conditional Routing (Split Based on Changes)

Instead of processing all rows identically, the solution branches logic:

```
Input Stream
    ↓
Change Detection Logic
    ├→ [NEW] Records Branch → Insert operations
    ├→ [UPDATED] Records Branch → Update commands
    └→ [UNCHANGED] Records Branch → Metrics only
    ↓
Union All (convergence)
    ↓
Final Destination
```

### Pattern 3: Lookup-Based Dimensionality

- **Source**: Incoming product records from CSV
- **Lookup Table**: Existing product dimension
- **Output**: Matched dimension keys and change indicators
- **Function**: Enables efficient change tracking and SCD implementation

### Pattern 4: Multi-Layer Staging Architecture (Package 2)

```
Source Files → Data Flow → Staging Tables → SQL Post-Processing → Final Dimension
```

This separation enables:
- Atomicity: All-or-nothing file processing
- Audit Trail: Historical record of staging states
- Error Recovery: Ability to reprocess failed files
- Performance: Batch SQL operations on staged data

### Pattern 5: Slowly Changing Dimension Type 2 Implementation

Dimensional tables maintain complete history with validity date ranges:

```
New Product Record Example:
{
  ProductID: 123,
  ProductName: "Road Bike",
  IsCurrent: 1,
  ValidityDate_Start: 2025-08-07,
  ValidityDate_End: NULL  (open-ended for current version)
}

Historical Version After Update:
{
  ProductID: 123,
  ProductName: "Road Bike",
  IsCurrent: 0,
  ValidityDate_Start: 2025-08-07,
  ValidityDate_End: 2025-11-05  (marked as inactive)
}

New Version After Update:
{
  ProductID: 123,
  ProductName: "Performance Road Bike",  (changed name)
  IsCurrent: 1,
  ValidityDate_Start: 2025-11-05,
  ValidityDate_End: NULL  (new current version)
}
```

---

## Key Transformations

### 1. Date Trimming (Trim Dates Component)

**Purpose**: Standardize and clean date fields from source data

**Applied Fields**:
- SellStartDate
- SellEndDate
- DiscontinuedDate
- ValidityDate_Start (created)
- ValidityDate_End (created)

**Transformation Logic**:
```
Original: "2012-06-20 00:00:00.000"
Trimmed:  "2012-06-20"
```

### 2. Product Matching (Lookup for New ProductID)

**Purpose**: Identify whether each product is new or existing

**Lookup Configuration**:
- **Input**: ProductNumber from source CSV
- **Dimension Table**: Existing product master
- **Match Key**: ProductNumber (unique identifier)
- **Output**: Existing ProductID or NULL (for new products)

**Implications**:
- New products: Insert into dimension (NULL lookup result)
- Existing products: Check for changes (conditional split)

### 3. Change Detection (Split Based on Changes)

**Purpose**: Route records based on detected modifications

**Conditions Evaluated**:

| Condition | Logic | Destination |
|-----------|-------|-------------|
| ProductName Changed | `CurrentName != LookupName` | UPDATE branch |
| Price Changed | `ListPrice != PreviousPrice` | UPDATE branch |
| No Changes | Both conditions false | No-change counting |

**Output Routing**: Deterministic branching ensures all records flow to appropriate processing logic

### 4. Slowly Changing Dimension Flag (Add IsCurrent Column)

**Purpose**: Mark records as current or historical version

**Expression**: `IsCurrent = 1`

**Interpretation**:
- `IsCurrent = 1`: Latest active version of product
- `IsCurrent = 0`: Historical/superseded version
- Used alongside ValidityDate_End for temporal queries

**Query Example**:
```sql
SELECT ProductID, ProductName, ListPrice
FROM Products
WHERE IsCurrent = 1 AND ValidityDate_End IS NULL;
```

### 5. Dimensional Hierarchy Processing

**Hierarchy Levels**:
```
ProductCategoryID (e.g., 1 = Bikes)
  └── ProductSubcategoryID (e.g., 1 = Mountain Bikes)
        └── ProductID (e.g., 868 = Road-275 Black 58)
              └── Product Attributes (Color, Size, Style, etc.)
```

**Handling**: Includes dimensional keys and descriptive names for degenerate dimensions

### 6. Bulk Operations (OLE DB Command vs. OLE DB Destination)

**OLE DB Command** (Update operations):
```sql
UPDATE Products 
SET ProductName = ? 
WHERE ProductID = ?
```

**OLE DB Destination** (Insert/bulk loads):
- Streaming mode for high-performance bulk inserts
- Parallel execution possible
- Memory-efficient for large datasets

---

## Implementation Highlights

### 1. Enterprise-Grade Error Handling

Both packages implement error paths for common ETL failures:
- Invalid file formats
- Data type mismatches
- Duplicate key violations
- Lookup failures with NULL handling
- Connection timeouts

### 2. Row Count Tracking

The `No changes Row Count` component (Row Sampling transformation) enables:
- Real-time metrics on data flow volumes
- Alerting when row counts exceed thresholds
- Audit trail for compliance reporting
- Performance monitoring of transformation pipeline

### 3. File Processing Efficiency (Package 2)

The Foreach Loop container with file enumerator pattern provides:
- **Scalability**: Process hundreds of CSV files without code changes
- **Manageability**: Simple folder-based source management
- **Automation**: Scheduled execution for recurring data loads
- **Cleanup**: Post-processing of loaded files (archive/delete)

### 4. Modular Component Design

Each transformation is independently configurable:
- Derived Column expressions use business logic-friendly syntax
- Lookup components cache dimension table in memory
- Conditional splits enable complex routing logic
- Union All components handle heterogeneous input schemas

### 5. Data Type Precision

Product catalog includes multiple data types:
- **Strings**: ProductName, ProductColor, ProductStyle
- **Decimals**: StandardCost, ListPrice (pricing precision)
- **Integers**: ProductID, Quantity metrics
- **Dates**: SellStartDate through ValidityDate_End
- **Booleans**: IsCurrent flag
- **Strings (enums)**: ProductLine (R, M, T, S, W)

### 6. Real-World Product Dimensions

Dataset includes:
- Complete product hierarchy (category → subcategory → product)
- Pricing information (StandardCost vs. ListPrice)
- Physical attributes (Size, Weight with unit codes)
- Temporal validity (sell start/end dates)
- Categorical attributes (Color, Style, Line)

---

## Data Processing Workflow

### Package 1 Execution Flow

```
1. Read CSV File
   ├─ Input: Flat file with ~50+ products
   ├─ Fields: ProductNumber, ProductID, Category IDs, Name, Pricing, Sizes, Dates
   └─ Output: Row stream to pipeline

2. Trim Dates
   ├─ Clean datetime fields to date format
   └─ Output: Standardized dates

3. Lookup Product ID
   ├─ Cross-reference each product against dimension
   ├─ Determine: New product vs. Existing product
   └─ Output: Lookup result (ID or NULL)

4. Add IsCurrent Flag
   ├─ Mark all incoming as current version (1)
   └─ Output: Row with IsCurrent = 1

5. Split Based on Changes
   ├─ Evaluate: ProductName changed? ListPrice changed?
   ├─ Route: To UPDATE branch or No-Change branch
   └─ Output: Two parallel streams

6a. [UPDATE Branch]
    ├─ ProductName Change: Execute SQL UPDATE for name
    ├─ UnitPrice Change: Execute SQL UPDATE for price
    └─ Output: Modified records

6b. [No-Change Branch]
    ├─ No changes Row Count: Count unchanged records
    └─ Output: Metrics

7. Union All
   ├─ Merge both branches back to single stream
   └─ Output: Unified result set

8. OLE DB Destination
   ├─ Bulk insert to Products warehouse table
   ├─ Column mapping: Source columns → Destination columns
   └─ Output: Final dimensional table with complete record
```

### Package 2 Execution Flow

```
INITIALIZATION PHASE:
1. Truncate Staging Table
   └─ Clear any residual data from previous runs

FOREACH LOOP PHASE (repeat for each *.csv file):
2. Enumerate CSV Files
   ├─ Iterate: D:\...\Data\*.csv
   └─ Process: Each file in sequence

3. Read CSV File
   ├─ Input: Current enumerated CSV file
   └─ Output: Row stream

4. Trim Dates + Lookup + IsCurrent (identical to Package 1)
   └─ Output: Enhanced row stream

5. Split Based on Changes
   ├─ Route: Changes to UPDATE branch, No-Changes to counting
   └─ Output: Two streams

6. Load to Staging DB Table
   ├─ Bulk insert into intermediate staging table
   ├─ Purpose: Accumulate all files' data before SCD processing
   └─ Output: Aggregated staging table

7. Copy CSV to Processed Folder
   ├─ Archive: Move processed CSV to /Processed subfolder
   ├─ Purpose: Prevent re-processing, maintain history
   └─ Output: File in archive location

[Loop back to step 2 for next CSV file]

POST-PROCESSING PHASE:
8. Staging Table Row Count
   ├─ Query: COUNT(*) from staging table
   ├─ Capture: Total rows from all processed files
   └─ Output: Row count metric

9. Slowly Changing Dim Script
   ├─ SQL Operations:
   │  ├─ UPDATE existing: Set IsCurrent=0, ValidityDate_End=TODAY()
   │  ├─ INSERT new: Add new versions with IsCurrent=1, ValidityDate_End=NULL
   │  └─ MERGE logic: Handle both inserts and updates efficiently
   └─ Output: Updated Products dimension table

COMPLETION:
10. All files processed, dimension table contains complete historical record
    with current versions marked (IsCurrent=1) and ValidityDate_End=NULL
```

---

## Database Schema Reference

### Source Format (CSV Input)

```
ProductNumber, ProductID, ProductCategoryID, ProductSubcategoryID,
ProductName, ProductCategoryName, ProductSubcategoryName,
ProductColor, ProductStyle, ProductLine,
StandardCost, ListPrice,
ProductSize, UnitSizeCode, UnitSizeName,
ProductWeight, UnitWeightCode, UnitWeightName,
SellStartDate, SellEndDate, DiscontinuedDate,
ValidityDate_Start, ValidityDate_End
```

### Sample Data Records

```
BB-3184, 437, (NULL), (NULL), Road Bottom Bracket, (NULL), (NULL),
(NULL), (NULL), (NULL),
0.00, 0.00,
(NULL), (NULL), (NULL),
(NULL), (NULL), (NULL),
2009-03-15, (NULL), (NULL),
2020-06-18, 2030-02-14

SK-C827-M, 828, 3, 23, Racing Bike Socks M, Clothing, Socks,
Black, U, R,
4.1825, 11.50,
M, (NULL), (NULL),
(NULL), (NULL), (NULL),
2012-06-20, 2013-06-19, (NULL),
2021-04-15, 2026-09-11

FR-T42B-62, 847, 2, 14, Performance Road Frame - Black 62, Components, Road Frames,
Black, U, R,
225.8914, 425.75,
62, CM, Centimeter,
2.85, LB, US pound,
2012-06-20, 2014-06-19, (NULL),
2020-11-19, 2028-02-14

BK-T84B-58, 868, 1, 2, Road-275 Black 58, Bikes, Road Bikes,
Black, U, R,
2689.5124, 4295.50,
58, CM, Centimeter,
16.50, LB, US pound,
2012-06-20, 2013-06-19, (NULL),
2021-09-22, 2030-10-18
```

---

## Skills Summary

This project demonstrates proficiency across the full spectrum of enterprise ETL development:

| Domain | Skills Demonstrated |
|--------|---------------------|
| **SSIS Development** | Package design, data flows, control flows, variables, expressions |
| **Data Transformation** | Derived columns, lookups, conditional splits, unions, row operations |
| **SQL Server Integration** | OLE DB connections, staging tables, T-SQL execution, data warehousing |
| **Change Tracking** | SCD Type 2 implementation, temporal dimensions, audit trails |
| **File Processing** | CSV parsing, Foreach loops, file enumerators, batch operations |
| **Data Quality** | Validation, error handling, row counting, metrics collection |
| **Performance Optimization** | Bulk operations, caching, parallelization-ready architecture |
| **Solution Architecture** | Multi-layer staging, separation of concerns, scalable design |

---

## Conclusion

These SSIS packages represent a comprehensive, production-ready ETL solution built on best practices in data integration. The dual-package approach (row-by-row precision vs. bulk loop processing) demonstrates flexibility in architectural choices, while the shared Slowly Changing Dimension Type 2 implementation ensures data integrity and historical accuracy. The project showcases readiness for enterprise data engineering roles and mastery of Microsoft's premier data integration platform.

---

**Created**: August 7, 2025  
**SSIS Version**: 17.0.1008.3 (SQL Server 2022)  
**Development Environment**: SQL Server Data Tools (SSDT)  
**Portfolio Status**: Ready for GitHub publication and professional showcase
