# ToyStore Data Warehouse

A Data Warehousing and Business Intelligence (DWBI) project that transforms raw online toy store transactional data (2012–2015) into a star-schema data warehouse, with SSIS ETL pipelines and a Power BI dashboard.

---

## 📋 Overview

| Item | Detail |
|---|---|
| **Database** | SQL Server — `ToyStore_DWH`, `ToyStore_OLTP` |
| **Schema** | Star Schema with SCD Type 2 |
| **ETL Tool** | SQL Server Integration Services (SSIS) |
| **Visualization** | Power BI |
| **Data Period** | 2012 – 2015 |

---

## 🗂️ Project Structure

```
ToyStore/
├── README.md                                   # This file
├── Documentation/
│   ├── DWBI Assignment 1.pdf                   # Schema design & data modelling
│   └── DWBI Assignment 2.pdf                   # ETL implementation & dashboard
├── SchemaDesign/
│   └── StarSchema.png                          # Star schema ERD
├── Dashboard/
│   ├── Slicers & Charts.png                    # Main overview page
│   ├── MatrixVisual.png                        # Sales matrix visual
│   ├── DrillDown.png                           # Drill-down chart
│   └── DrillThrough/
│       ├── DrillThrough1.png                   # Order-level detail page
│       └── DrillThrough2.png                   # Customer & channel detail page
├── SourceFiles/
│   ├── orders.csv                              # Order header records
│   ├── orders_fixed.csv                        # Cleaned orders (used by ETL)
│   ├── order_items.csv                         # Line items per order
│   ├── order_item_refunds.csv                  # Refund transactions
│   ├── order_completions.csv                   # Accumulating completion timestamps
│   ├── users.csv                               # Registered customer accounts
│   ├── users_fixed.csv                         # Cleaned users (used by ETL)
│   ├── website_sessions.csv                    # Web sessions with UTM data
│   ├── website_pageviews.csv                   # Page-level clickstream events
│   └── Excel/
│       ├── products.xlsx                       # Product master data
│       └── geography.xlsx                      # Geography lookup data
├── SQL Queries/
│   ├── Database_setup.sql.txt                  # Create ToyStore_DWH & OLTP databases
│   ├── DimensionTables.sql.txt                 # Create all dimension tables
│   ├── FactTable.sql.txt                       # Create FactSales & error log table
│   ├── StaggingTable.sql.txt                   # Create staging tables
│   ├── Populated_dimdate.sql.txt               # Populate DimDate (2012–2015)
│   ├── SCD_type2Queries.txt                    # SCD Type 2 history updates
│   ├── FixDuplicateRows.sql.txt                # Remove duplicate dimension rows
│   ├── VerifyRowCount.sql.txt                  # Verify row counts across all tables
│   ├── AccumulatingFact.sql.txt                # Validate completion timestamp updates
│   └── VerifyFactable.sql.txt                  # Final fact table validation
└── ToyStore_ETL/                               # SSIS Project
    ├── ToyStore_ETL-IT23725010.dtproj          # Visual Studio project file
    ├── ToyStore_ETL/
    │   ├── Load_Dimensions.dtsx                # Load all dimension tables
    │   ├── Load_FactSales.dtsx                 # Load fact table
    │   ├── Update_AccumFact.dtsx               # Update accumulating fact timestamps
    │   ├── DWH_Conn.conmgr                     # Connection: ToyStore_DWH database
    │   ├── OLTP_Conn.conmgr                    # Connection: ToyStore_OLTP database
    │   ├── CSV_Users.conmgr                    # Connection: users_fixed.csv
    │   ├── CSV_Sessions.conmgr                 # Connection: website_sessions.csv
    │   ├── CSV_Pageviews.conmgr                # Connection: website_pageviews.csv
    │   ├── XL_Products.conmgr                  # Connection: products.xlsx
    │   ├── XL_Geography.conmgr                 # Connection: geography.xlsx
    │   └── Project.params                      # Project-level parameters
    └── bin/Development/                        # Compiled SSIS packages
        └── ToyStore_ETL-IT23725010.ispac
```

---

## 🏗️ Schema Design

**Fact Table:** `FactSales` — order line items with revenue, COGS, discount, refund, and order processing timestamps.

**Dimension Tables:**

| Table | SCD | Key Attributes |
|---|---|---|
| `DimDate` | Type 0 | Date, Day, Month, Quarter, Year, IsWeekend |
| `DimProduct` | Type 2 | Name, Category, Brand, Cost, AgeGroup |
| `DimCustomer` | Type 2 | Email, Device, Country, HasPurchased |
| `DimGeography` | Type 1 | City, Country, Region, Continent |
| `DimMarketingChannel` | Type 1 | UTM Source, Campaign, Content, DeviceType |
| `DimShipping` | Type 0 | Standard, Express, Free Shipping |

---

## 📂 Data Sources

| File | Description | Rows |
|---|---|---|
| `orders.csv` | Order headers | ~32,000 |
| `order_items.csv` | Line items per order | ~40,000 |
| `order_item_refunds.csv` | Refund records | ~1,700 |
| `order_completions.csv` | Accumulating completion timestamps | ~100 |
| `users.csv` | Customer accounts | ~394,000 |
| `website_sessions.csv` | Sessions with UTM data | ~472,000 |
| `website_pageviews.csv` | Page-level clickstream | ~1,188,000 |
| `products.xlsx` | Product master | — |
| `geography.xlsx` | Geography lookup | — |

---

## 🚀 Getting Started

### Prerequisites
- SQL Server 2019+
- Visual Studio 2019+ with SSIS extension
- Power BI Desktop

### 1. Create Databases
```sql
-- Run: SQL Queries/Database_setup.sql.txt
-- Creates: ToyStore_DWH and ToyStore_OLTP
```

### 2. Build Schema (run in order)
```
1. DimensionTables.sql.txt
2. FactTable.sql.txt
3. StaggingTable.sql.txt
4. Populated_dimdate.sql.txt
```

### 3. Configure SSIS
Open `ToyStore_ETL/ToyStore_ETL-IT23725010.dtproj` and update:
- `DWH_Conn.conmgr` / `OLTP_Conn.conmgr` → your SQL Server instance
- `CSV_*.conmgr` / `XL_*.conmgr` → local path to `SourceFiles/`

### 4. Run ETL Packages (in order)
| Package | Purpose |
|---|---|
| `Load_Dimensions.dtsx` | Load all 6 dimension tables |
| `Load_FactSales.dtsx` | Load order line items into FactSales |
| `Update_AccumFact.dtsx` | Update completion timestamps |

### 5. Post-Load
```
- FixDuplicateRows.sql.txt   → deduplicate dimensions
- SCD_type2Queries.txt       → apply SCD Type 2 history
- VerifyRowCount.sql.txt     → confirm table row counts
- VerifyFactable.sql.txt     → validate joined fact data
```

---

## 📊 Dashboard

Power BI visuals are in `Dashboard/`:

| Screenshot | Description |
|---|---|
| `Slicers & Charts.png` | Main overview with filters and trend charts |
| `MatrixVisual.png` | Sales breakdown by category and geography |
| `DrillDown.png` | Time-series drill-down (year → month) |
| `DrillThrough1/2.png` | Order and customer detail pages |

---

## 📄 Documentation

Full design and implementation details are in `Documentation/`:
- `DWBI Assignment 1.pdf` — Schema design, data modelling, SCD strategy
- `DWBI Assignment 2.pdf` — ETL implementation, accumulating facts, dashboard

---

## 📄 License
This project was created for academic purposes as part of IT3021 coursework.

## 👤 Author
Created for Data Warehouse design and ETL implementation assignment.
