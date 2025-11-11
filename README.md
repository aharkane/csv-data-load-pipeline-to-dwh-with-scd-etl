# CSV Data Load Pipeline to Data Warehouse with Slowly Changed Dimension Using SSIS ETL Tool 

---

## Table of Contents

1. [Overview](#overview)
2. [Technical Skills Demonstrated](#technical-skills-demonstrated)
3. [Package 1: Single File Product.csv RowbyRow ETL](#package-1-single-file-productcsv-rowbyrow-etl)
4. [Package 2: Batch Multi-File Product*.csv Bulk Load ETL](#package-2--batch-multi-file-productcsv-bulk-load-etl)


---
## Overview

ETL solution featuring two processing approaches for product data: row-by-row direct processing and batch multi-file operations. Implements SCD Type 2 for complete historical tracking of product changes (name, price, new products).

---

## Technical Skills Demonstrated

<table>
<tr>
<td>

**SSIS Development**
- Data Flow Task design
- Control Flow Foreach-Loop implementation
- Execute SQL Task
</td>
<td>

**ETL Patterns**
- Staging layer architecture
- Row-by-row vs batch processing
- SCD Implementation
- Data Transformations : Conditional Split, Lookup

</td>
</tr>
</table>

---

## Package 1: Single File Product.csv RowbyRow ETL
ETL pipeline row-by-row ETL pipeline designed for precision processing of product updates with immediate change detection and routing.

**Transformation Logic**

```
Raw CSV → Date Trim → Lookup → IsCurrent Flag → Split Changes
                                                  ├→ UPDATE Changes → Union All → Destination
                                                  └→ Count No-Changes
```

**Execution Flow**

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

## Package 2:  Batch Multi-File Product*.csv Bulk Load ETL

ETL pipeline with foreach looping capability, processing multiple CSV source files sequentially with comprehensive staging and SQL post-processing for slowly changing dimension implementation.


**User Variables**

| Variable Name | Type | Purpose | Example Value |
|---------------|------|---------|---|
| `FileNamePath` | String | Root directory for CSV source files | `...\Data` |
| `RowCountVar` | Integer | Counter for row-level metrics | 0 (initialized) |



**Transformation Logic**

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

**Execution Flow**

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

## Conclusion

These SSIS packages represent a comprehensive, production-ready ETL solution built on best practices in data integration. The dual-package approach (row-by-row precision vs. bulk loop processing) demonstrates flexibility in architectural choices, while the shared Slowly Changing Dimension Type 2 implementation ensures data integrity and historical accuracy. The project showcases readiness for enterprise data engineering roles and mastery of Microsoft's premier data integration platform.
