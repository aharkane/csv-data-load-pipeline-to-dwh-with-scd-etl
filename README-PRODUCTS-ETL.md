# 🏭 AdventureWorks Products ETL Pipeline
### Automated SSIS Package for Product Dimension Management

[![SQL Server](https://img.shields.io/badge/SQL%20Server-2022-CC2927?logo=microsoft-sql-server&logoColor=white)](https://www.microsoft.com/sql-server)
[![SSIS](https://img.shields.io/badge/SSIS-ETL%20Pipeline-0078D4?logo=microsoft&logoColor=white)](https://docs.microsoft.com/sql/integration-services)
[![Data Integration](https://img.shields.io/badge/ETL-Automation-4CAF50?logo=microsoft&logoColor=white)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Production-ready SSIS ETL pipeline for automated product dimension loading, transformation, and management from AdventureWorks OLTP into the DWH.**

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [Technical Architecture](#technical-architecture)
- [Getting Started](#getting-started)
- [Package Structure](#package-structure)
- [Data Flow](#data-flow)
- [Sample Data](#sample-data)
- [Related Projects](#related-projects)
- [Author](#author)
- [License](#license)

---

## Project Overview

This SSIS ETL pipeline is part of the larger **AdventureWorks Sales Analytics** data platform. It automates the extraction, transformation, and loading of product dimension data from the OLTP source system into the enterprise data warehouse.

### What This Project Does

- 🔄 **Automated ETL**: Extracts product updates from OLTP
- 🔀 **Data Transformation**: Enriches product attributes (categories, subcategories, pricing)
- 📊 **Dimension Loading**: Loads DimProduct with SCD Type 1 & 2 logic
- ✅ **Data Quality**: Validates data integrity and completeness
- 📈 **Monitoring**: Logs execution metrics and error handling
- 🔗 **Integration**: Works seamlessly with the enterprise DWH

### Business Context

Product dimension is critical for sales analytics:
- 📦 Product catalog management (504 active products)
- 💰 Pricing strategy tracking (historical and current)
- 📂 Category and subcategory organization
- 🎨 Product attributes (color, size, style, line)

### Project Statistics

- **2 SSIS Packages** (Single File & Multi-File variants)
- **11 Data Flows** for extraction and transformation
- **504 Products** processed from OLTP
- **4 Categories** and **37 Subcategories**
- **SCD Type 1 & 2** implementation
- **<30 seconds** execution time

---

## Features

### 1. Dual Package Architecture

#### Single File Package
- **Purpose**: Consolidated ETL in one SSIS package
- **Use Case**: Small loads, quick testing, simple deployments
- **File**: `1.-ETL_Products_single_File.xml`
- **Best For**: Proof of concept, development environments

#### Multi-File Package
- **Purpose**: Modular design with separate extract, transform, and load stages
- **Use Case**: Production environments, complex workflows, parallel processing
- **File**: `2.-ETL_Products_Multi_Files.xml`
- **Best For**: Enterprise deployments, scalability, maintainability

### 2. Comprehensive Data Transformation

**Input Data** (from OLTP):
```
ProductID, ProductNumber, ProductName, ProductColor, ProductSize,
StandardCost, ListPrice, SellStartDate, SellEndDate, ProductCategoryID,
ProductSubcategoryID, UnitMeasureCode, ProductWeight
```

**Output Data** (to DWH):
```
ProductDWKey (Surrogate Key), ProductID (Natural Key), ProductNumber,
ProductName, ProductCategoryID, ProductCategoryName, ProductSubcategoryID,
ProductSubcategoryName, ProductColor, ProductStyle, ProductLine,
StandardCost, ListPrice, ProductSize, UnitSizeCode, UnitSizeName,
ProductWeight, UnitWeightCode, UnitWeightName, SellStartDate, SellEndDate,
DiscontinuedDate, ValidityDate_Start, ValidityDate_End, IsCurrent
```

### 3. SCD Type 1 & 2 Implementation

| Attribute | SCD Type | Change Handling | History |
|-----------|----------|-----------------|---------|
| **StandardCost** | Type 1 | Overwrite current value | No |
| **ListPrice** | Type 2 | Create new version | Yes ⭐ |
| **ProductName** | Type 0 | Fixed (never changes) | No |
| **ProductColor** | Type 1 | Overwrite | No |
| **DiscontinuedDate** | Type 1 | Overwrite | No |

### 4. Error Handling & Logging

- ✅ Data validation checks
- ✅ Primary key constraint validation
- ✅ NULL value handling
- ✅ Referential integrity checks
- ✅ Exception logging
- ✅ Execution metrics tracking

### 5. Performance Optimization

- 🚀 Parallel data flows
- 🎯 Incremental loading capability
- 📦 Batch processing
- 💾 Memory-optimized data paths
- ⚡ <30 second execution

---

## Technical Architecture

### SSIS Package Components

```
┌─────────────────────────────────────────────────────┐
│         AdventureWorks2022 OLTP Database            │
│              Production.Product Table               │
│  ┌─────────────────────────────────────────────┐   │
│  │  504 Products with full attributes          │   │
│  │  • Categories (4)                           │   │
│  │  • Subcategories (37)                       │   │
│  │  • Pricing and costs                        │   │
│  │  • Physical attributes                      │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
                        │
                        ▼ Data Flow Task
        ┌───────────────────────────────────┐
        │      EXTRACT                      │
        │  • Retrieve product records       │
        │  • Filter by updated dates        │
        │  • Cache for transformation       │
        └───────────────────────────────────┘
                        │
                        ▼ Lookups + Joins
        ┌───────────────────────────────────┐
        │      TRANSFORM                    │
        │  • Join Category data             │
        │  • Join Subcategory data          │
        │  • Join UnitMeasure data          │
        │  • Apply business logic           │
        │  • Add derived columns            │
        └───────────────────────────────────┘
                        │
                        ▼ Data Flow Task
        ┌───────────────────────────────────┐
        │      LOAD                         │
        │  • Insert/Update DimProduct       │
        │  • SCD Type 1 overwrites          │
        │  • SCD Type 2 versioning          │
        │  • Update ETL metadata            │
        └───────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│       AdventureWorks2022DWH Database               │
│  ┌────────────────────────────────────────────┐   │
│  │  prod.DimProduct                           │   │
│  │  ┌──────────────────────────────────────┐  │   │
│  │  │ ProductDWKey (PK, IDENTITY)          │  │   │
│  │  │ ProductID (Business Key)             │  │   │
│  │  │ ProductName, Category, Subcategory   │  │   │
│  │  │ Pricing (Type 1 & 2)                 │  │   │
│  │  │ Physical attributes                  │  │   │
│  │  │ SCD tracking columns                 │  │   │
│  │  │ (IsCurrent, ValidityDate_Start/End)  │  │   │
│  │  └──────────────────────────────────────┘  │   │
│  └────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

### Data Flow Stages

**Stage 1: Extract**
```
Source: Production.Product (OLTP)
├── Connection: AdventureWorks2022OLTP
├── Query: SELECT * WHERE ModifiedDate > LastCutoff
├── Columns: 13 product attributes
└── Rows: 504 products
```

**Stage 2: Transform**
```
Transformations Applied:
├── Lookup: ProductCategory (4 categories)
├── Lookup: ProductSubcategory (37 subcategories)
├── Lookup: UnitMeasure (Weight & Size units)
├── Derived Columns:
│   ├── ProductCategoryName (from lookup)
│   ├── ProductSubcategoryName (from lookup)
│   ├── UnitSizeCode & UnitSizeName
│   ├── UnitWeightCode & UnitWeightName
│   ├── ValidityDate_Start (GETDATE())
│   ├── ValidityDate_End (9999-12-31)
│   └── IsCurrent (1 for new records)
└── Data Type Conversions
```

**Stage 3: Load**
```
Destination: prod.DimProduct (DWH)
├── Connection: AdventureWorks2022DWH
├── Operation: MERGE (Insert/Update)
├── SCD Logic:
│   ├── Type 1 Columns:
│   │   ├── StandardCost (UPDATE)
│   │   ├── ProductColor (UPDATE)
│   │   └── DiscontinuedDate (UPDATE)
│   │
│   └── Type 2 Columns:
│       ├── ListPrice → New Version if Changed
│       ├── IsCurrent = 0 (mark old as historical)
│       └── IsCurrent = 1 (new record)
└── Error Output: Capture any failures
```

---

## Getting Started

### Prerequisites

- **SQL Server 2019+** (Express/Developer Edition works)
- **SQL Server Integration Services (SSIS)**
- **SQL Server Data Tools (SSDT)** for editing
- **AdventureWorks2022 OLTP Database** (source)
- **AdventureWorks2022DWH Database** (target - from main project)

### Installation

#### 1. Extract SSIS Package

```bash
# From GitHub repository
git clone https://github.com/aharkane/adventureworks-products-etl.git
cd adventureworks-products-etl

# Choose your package variant
# Option A: Single File (recommended for first-time)
# Option B: Multi-File (recommended for production)
```

#### 2. Import Package into SSIS

**Using SQL Server Data Tools (SSDT)**:
```
1. Open SSDT
2. File → New → Integration Services Project
3. Right-click Project → Add Existing Package
4. Select: 1.-ETL_Products_single_File.xml
5. Rename if desired
```

**Using SSIS Catalogs**:
```sql
-- Deploy to SSIS Catalog
USE [SSISDB]
GO

EXEC [catalog].[create_folder] @folder_name = N'AdventureWorks'
EXEC [catalog].[deploy_project] 
    @folder_name = N'AdventureWorks',
    @project_name = N'ProductsETL',
    @ProjectStream = <your_ispac_file>
```

#### 3. Configure Connection Strings

**Source Connection** (OLTP):
```
Server: YOUR_SQL_SERVER
Database: AdventureWorks2022OLTP
Authentication: Windows or SQL Server
```

**Destination Connection** (DWH):
```
Server: YOUR_SQL_SERVER
Database: AdventureWorks2022DWH
Authentication: Windows or SQL Server
```

#### 4. Execute Package

**From SSDT**:
```
1. Right-click package
2. Execute Package
3. Monitor execution in progress window
4. Verify output in DWH
```

**From SQL Agent** (for scheduling):
```sql
EXEC msdb.dbo.sp_add_job 
    @job_name = 'ETL_Products_Daily'

EXEC msdb.dbo.sp_add_jobstep
    @job_name = 'ETL_Products_Daily',
    @step_name = 'ExecuteProductsETL',
    @subsystem = 'SSIS',
    @command = '/ISSERVER "\SSISDB\AdventureWorks\ProductsETL\ETL_Products" /SERVER "YOUR_SERVER"'

EXEC msdb.dbo.sp_add_schedule
    @schedule_name = 'Daily_Midnight',
    @freq_type = 4,  -- Daily
    @freq_interval = 1,
    @active_start_time = '000000'
```

---

## Package Structure

### Single File Package
```
1.-ETL_Products_single_File.xml
├── Control Flow
│   ├── Execute SQL Task: Initialize
│   ├── Data Flow Task: Extract & Transform
│   ├── Execute SQL Task: Load
│   └── Execute SQL Task: Logging
│
└── Data Flow
    ├── OLE DB Source (OLTP)
    ├── Lookup: Categories
    ├── Lookup: Subcategories
    ├── Lookup: UnitMeasure
    ├── Derived Column Transform
    ├── Data Conversion
    └── OLE DB Destination (DWH)
```

### Multi-File Package
```
2.-ETL_Products_Multi_Files.xml
├── Control Flow - Extract Phase
│   ├── Execute SQL Task: Get Cutoff Date
│   └── Data Flow: Extract Products
│
├── Control Flow - Transform Phase
│   ├── Execute SQL Task: Staging Prep
│   └── Data Flow: Apply Transformations
│
├── Control Flow - Load Phase
│   ├── Execute SQL Task: SCD Type 1
│   ├── Execute SQL Task: SCD Type 2
│   └── Execute SQL Task: Update ETL Log
│
└── Control Flow - Cleanup
    └── Execute SQL Task: Clear Staging
```

---

## Data Flow

### Sample Input Data (ProductsUpdates.csv)

```
ProductNumber,ProductID,ProductName,ProductColor,ListPrice,StandardCost,SellStartDate,SellEndDate
BK-T84B-58,868,Road-275 Black 58,Black,4295.50,2689.5124,2012-06-20,2013-06-19
BK-T53B-54,873,Road-550 Black 54,Black,1795.75,1095.8291,2012-06-20,2013-06-19
FR-T42B-62,847,Performance Road Frame - Black 62,Black,425.75,225.8914,2012-06-20,2014-06-19
FE-N76G-52,864,Elite Mountain Frame - Green 52,Green,1564.80,818.1705,2012-06-20,2013-06-19
HS-3762,925,Performance Headset,,128.95,56.2485,2013-07-10,2014-07-09
```

### Sample Output Data (DimProduct)

```
ProductDWKey | ProductID | ProductName | ProductCategoryName | ListPrice | IsCurrent | ValidityDate_Start | ValidityDate_End
1            | 868       | Road-275    | Bikes               | 4295.50   | 1         | 2024-11-05        | 9999-12-31
2            | 873       | Road-550    | Bikes               | 1795.75   | 1         | 2024-11-05        | 9999-12-31
3            | 847       | Perf Frame  | Components          | 425.75    | 1         | 2024-11-05        | 9999-12-31
```

---

## Related Projects

This package is part of the **AdventureWorks Sales Analytics Platform**:

### Core Data Warehouse
- 📦 **Repository**: [adventureworks-sales-dwh](https://github.com/aharkane/adventureworks-sales-dwh)
- ⭐ **Description**: Enterprise DWH with metadata-driven ETL, SCD implementation, 19 stored procedures
- 🔗 **Connection**: This package loads into the prod.DimProduct table

### Analytics Layer (Coming Soon)
- 📊 **Future**: Power BI dashboards for product performance analytics
- 📈 **Features**: Interactive visualizations, DAX measures, drill-down analysis

### Ecosystem Diagram
```
OLTP Source
    ↓
[adventureworks-sales-dwh] ← Main DWH (metadata-driven ETL)
    ↓
├── prod.DimAddress
├── prod.DimCustomer
├── prod.DimSalesPerson
├── [prod.DimProduct] ← THIS PROJECT loads this table
└── prod.FactSales
    ↓
[Future: adventureworks-powerbi-analytics]
```

---

## Key Skills Demonstrated

✅ **SSIS Development**
- Data Flow tasks
- Control Flow orchestration
- Error handling & logging
- Package deployment

✅ **ETL Best Practices**
- Incremental loading
- SCD Type 1 & 2 logic
- Data validation
- Performance optimization

✅ **SQL Server Integration**
- Connection management
- Transaction handling
- Stored procedure integration
- ETL metadata tracking

✅ **Data Engineering**
- Dimensional modeling
- Data transformation
- Quality assurance
- Production-ready code

---

## Monitoring & Troubleshooting

### Check Execution Logs
```sql
-- Query SSIS execution history
SELECT 
    execution_id,
    project_name,
    package_name,
    execution_status,
    start_time,
    end_time,
    DATEDIFF(SECOND, start_time, end_time) AS duration_seconds
FROM [SSISDB].[catalog].[executions]
WHERE package_name = 'ETL_Products'
ORDER BY execution_id DESC
```

### Validate Data Load
```sql
-- Check DimProduct record count
SELECT COUNT(*) as total_products FROM prod.DimProduct WHERE IsCurrent = 1
-- Expected: 504 active products

-- Check SCD Type 2 versions
SELECT ProductID, COUNT(*) as versions
FROM prod.DimProduct
GROUP BY ProductID
HAVING COUNT(*) > 1
-- Shows products with historical versions
```

### Common Issues

| Issue | Solution |
|-------|----------|
| Connection timeout | Check server name, firewall rules |
| Data type mismatch | Verify column data types in source and destination |
| Lookup failure | Ensure reference tables exist in destination |
| SCD logic not applied | Verify SQL merge statement in load task |

---

## Author

**Harkane Amine**
- 💼 LinkedIn: [Harkane Amine](https://www.linkedin.com/in/aharkane/)
- 🐙 GitHub: [@aharkane](https://github.com/aharkane)
- 📧 Email: [harkaneamine@gmail.com](mailto:harkaneamine@gmail.com)

### Project Evolution

This SSIS project evolved from the metadata-driven DWH framework, demonstrating practical ETL automation skills:
- Phase 1 ✅: Built enterprise data warehouse with T-SQL procedures
- Phase 2 ✅: Automated product loading with SSIS (this project)
- Phase 3 🔜: Power BI analytics dashboards
- Phase 4 🔜: Real-time streaming analytics

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

- **Microsoft** - AdventureWorks sample database
- **Kimball Group** - Dimensional modeling methodology
- **SSIS Documentation** - Integration Services reference

---

<div align="center">

**⭐ If you found this project helpful, please give it a star! ⭐**

Made with ❤️ and ☕ by [Harkane Amine](https://github.com/aharkane)

*Last Updated: November 2025*

</div>
