# Sales & Finance Dashboard

## 📊 DASHBOARD ARCHITECTURE

```
Sales & Finance Dashboard
│
├─ Page 1: Executive Overview
│  ├─ 3 KPI Cards (Revenue, Profit, Margin)
│  ├─ Revenue Trend (12-month line chart)
│  ├─ Revenue by Region (bar chart)
│  ├─ Top Products (horizontal bar)
│  └─ Global Filters (Date, Region, Category)
│
├─ Page 2: Sales Performance
│  ├─ 3 KPI Cards (Revenue, Qty, AOV)
│  ├─ Revenue vs Target (gauge chart)
│  ├─ Regional Breakdown (matrix table)
│  ├─ Product Performance (detail table)
│  └─ Regional Donut & Category Pie Charts
│
├─ Page 3: Financial Analysis
│  ├─ 6 KPI Cards (Revenue through Net Margin)
│  ├─ Profit Waterfall (breakdown chart)
│  ├─ Margin Trend (area chart)
│  ├─ Monthly Profit (stacked column)
│  └─ Margin by Category (clustered column)
│
├─ Page 4: Detailed Analysis
│  ├─ Sales Rep Scorecard (matrix)
│  ├─ Rep Performance Rankings (bar charts)
│  ├─ Customer Analysis (table)
│  ├─ Transaction Details (100 rows table)
│  └─ Filter for Sales Rep & Date Range
│
└─ Drill-Through Pages
   ├─ Region Detail
   ├─ Product Detail
   └─ Sales Rep Detail
```

---

## 📈 KEY METRICS (50+ Measures)

### Revenue Measures
- Total Revenue
- Revenue MTD / YTD
- Revenue Growth %
- vs Previous Month
- Revenue Target Achievement

### Profit & Margin Measures
- Gross Profit & Margin %
- Net Profit & Margin %
- Total COGS
- Operating Expenses
- Margin by Product/Category

### Volume Measures
- Total Quantity Sold
- Transaction Count
- Average Order Value
- Unique Customers
- Customer Purchase Frequency

### Performance Measures
- Regional Rankings
- Product Rankings
- Sales Rep Rankings
- YoY Growth %
- 3-Month Moving Average

---

## 💾 DATA MODEL

### Tables
```
Sales (Main Fact Table)
├─ TransactionID (PK)
├─ Date (FK to Calendar)
├─ ProductID (FK to Finance)
├─ Quantity, UnitPrice, SalesAmount
├─ Region, SalesRep, CustomerID
└─ Derived: Month, Year, Quarter, MonthYear

Finance (Product Dimension)
├─ ProductID (PK)
├─ ProductName, Category
├─ CostPrice
├─ Supplier, CostCenter
└─ Derived: StandardMargin

Calendar (Date Dimension - auto-created)
├─ Date (PK)
├─ Year, Month, Day
├─ Quarter, WeekNumber
├─ MonthName, DayName
└─ YearMonth, MonthYear Formats
```

### Relationships

Sales.ProductID → Finance.ProductID (1:M)
Sales.Date → Calendar.Date (1:M)


## 📊 SAMPLE DATA INCLUDED

### Sales Data (sales_data.csv)
- 30 sample transactions from Jan-Apr 2024
- 5 products across 3 categories
- 4 regions and 3 sales reps
- Realistic transaction amounts ($6K-$25K per transaction)


### Finance Data (finance_data.csv)
- Cost price for each of 5 products
- Product categories and cost centers
- Used for profit and margin calculations
- Realistic cost structures


### DATA MODELING

Step 1: Import Sales Data

1. Open Power BI Desktop
2. Home → Get Data → Select your source (CSV, SQL, Excel)
3. Browse to sales_data.csv file
4. Load → Apply transformations in Power Query
5. Use Power_Query_Scripts.md for transformation logic
6. Click Load


Step 2: Import Finance Data

1. Home → Get Data → Select Finance data source
2. Load finance_data.csv
3. Apply transformations (check for duplicates, validate costs)
4. Click Load


Step 3: Create Calendar Table

1. Home → New Table
2. Paste Calendar Query from Power_Query_Scripts.md
3. Set minimum and maximum date based on data range
4. Load calendar with all required date fields


### Define Table Relationships

In the Model view:

1. Click on Sales table
2. Drag ProductID to Finance ProductID → Create relationship
   (Sales.ProductID → Finance.ProductID, One-to-Many, Single)

3. Drag Date (from Sales) to Calendar Date
   (Sales.Date → Calendar.Date, One-to-Many, Single)

4. Mark Calendar as Date Table:
   - Calendar table → Mark as Date Table → Date column: Date


### Hide Unnecessary Fields
- Hide all ID fields (they're only for relationships)
- Hide duplicate columns after merging
- Keep only business-relevant columns visible to users

### Verify Data Integrity
```DAX
// Create a quick measure to verify row counts
Row Count = COUNTROWS(Sales)
// Should match source system
```

---

##  DAX MEASURES & CALCULATIONS 

### 3.1 Create Measure Table
```
1. In Power BI, right-click on Sales table
2. New Measure
3. Create a dedicated "Measures" table for organization:
   - Home → New Table
   - Measures = SELECTCOLUMNS(Sales, "Dummy", 1)
   - Hide Dummy column
```

### Add Measures

**Revenue Measures**
```DAX
Copy all measures from DAX_Formulas.md section "REVENUE MEASURES"
- Total Revenue
- Revenue MTD
- Revenue YTD
- Revenue Growth %
```

**Profit Measures**
```DAX
Copy all measures from DAX_Formulas.md section "COST & PROFIT MEASURES"
- Total COGS
- Gross Profit
- Gross Profit Margin %
- Net Profit
- Net Profit Margin %
```

**Volume & KPI Measures**
```DAX
Copy from section "VOLUME & QUANTITY MEASURES" and "KPI & THRESHOLD MEASURES"
- Total Quantity
- AOV
- Unique Customers
- Revenue Target Achievement %
```

### Test Measures
Create a test table visual:
1. Drag Date (Month level) to Rows
2. Drag each measure to Values
3. Verify calculations match expected results
4. Compare with source system reports
5. Delete test table after validation

### Create Comparison Measures
```DAX
// YoY Growth
YoY Growth % = 
IFERROR(
    ([Total Revenue] - CALCULATE([Total Revenue], DATEADD(Calendar[Date], -1, YEAR))) 
    / CALCULATE([Total Revenue], DATEADD(Calendar[Date], -1, YEAR)),
    BLANK()
)

// vs Previous Month
vs Previous Month = 
[Total Revenue] - CALCULATE([Total Revenue], DATEADD(DATESMTD(Calendar[Date]), -1, MONTH))
```

---

## VISUALIZATION

### Create Executive Overview

**Create Cards/KPIs**
```
1. Insert → Card (for Total Revenue)
   - Fields: Drag Total Revenue measure
   - Format: Set to Currency with 0 decimals
   - Add conditional formatting for trends

2. Repeat for Gross Profit, Net Profit, Profit Margin %
   - Adjust formatting for each measure type
   - Add trend indicators and growth percentages
```

**Create Line Chart (Revenue Trend)**
```
1. Insert → Line Chart
2. X-axis: Calendar[MonthYear]
3. Y-axis: Total Revenue
4. Add additional lines:
   - Revenue Target (constant)
   - 3-Month Moving Average
5. Format:
   - Title: "Revenue Trend (Last 12 Months)"
   - Legend: Top
   - Data colors: Blue for actual, Red dashed for target
```

**Create Bar Chart (Revenue by Region)**
```
1. Insert → Bar Chart
2. Category: Sales[Region]
3. Values: Total Revenue (sort descending)
4. Format: Enable data labels, use blue color
5. Enable drill-through to region details page
```

**Create Donut Chart (Revenue by Category)**
```
1. Insert → Donut Chart
2. Legend: Sales[Category]
3. Values: Total Revenue
4. Format: Display percentages, custom colors
5. Add tooltip showing exact values
```

### Create Sales Performance

**Regional Breakdown Table**
```
1. Insert → Matrix/Table
2. Rows: Sales[Region]
3. Values: 
   - Total Revenue
   - Total Quantity
   - Transaction Count
   - Average Order Value
4. Add conditional formatting for highlights
5. Enable expand/collapse for drill-down
```

**Product Performance Table**
```
1. Insert → Table
2. Columns:
   - ProductName
   - Total Revenue (sorted descending)
   - Total Quantity
   - Gross Profit
   - Gross Profit Margin %
   - Product Rank
3. Format: Enable sorting, highlight top 3 products
4. Add data bars for visual comparison
```

**Revenue vs Target Gauge**
```
1. Insert → Gauge Chart
2. Value: Total Revenue
3. Min/Max: Set based on target range
4. Target: Revenue Target measure
5. Format: Set to 0-150% scale, add threshold colors
```

### Financial Analysis

**Waterfall Chart (Profit Breakdown)**
```
1. Insert → Waterfall Chart
2. Columns (in order):
   - Revenue (actual, blue)
   - COGS (decrease, red)
   - Operating Expense (decrease, orange)
   - Net Profit (total, green)
3. Format: Show values on columns
4. Title: "Profit Waterfall Analysis"
```

**Margin Trend (Area Chart)**
```
1. Insert → Area Chart
2. X-axis: Calendar[MonthYear]
3. Y-axis: 
   - Gross Profit Margin %
   - Net Profit Margin %
4. Format: Use green for gross, blue for net
5. Title: "Profit Margin Trend"
```

**Margin by Category (Clustered Column)**
```
1. Insert → Clustered Column Chart
2. X-axis: Sales[Category]
3. Y-axis: Gross Profit Margin %, Net Profit Margin %
4. Format: Show data labels, conditional colors
5. Add reference line for target margin (30%)
```

### Create Detailed Analysis

**Sales Rep Scorecard (Matrix)**
```
1. Insert → Matrix
2. Rows: Sales[SalesRep]
3. Values:
   - Total Revenue
   - Total Quantity
   - Gross Profit
   - Gross Profit Margin %
   - Unique Customers
   - Sales Rep Rank
4. Format: 
   - Sort by revenue (descending)
   - Add conditional formatting (color scale)
   - Enable drill-down
```

**Transaction Details Table**
```
1. Insert → Table
2. Columns:
   - Date
   - ProductName
   - Region
   - SalesRep
   - Quantity
   - UnitPrice
   - SalesAmount
   - (Calculated Profit)
   - Gross Profit Margin %
3. Format: 
   - Sort by date (descending)
   - Show top 100 transactions
   - Add data bars for amounts
4. Optional: Add search/filter on Product names
```

---

## FILTERS 

### Add Global Slicers

On each page (especially Page 1), add:

**Date Slicer**
```
1. Insert → Slicer → Calendar[MonthYear]
2. Format: Dropdown or List
3. Default: Last 12 months
4. Position: Top of page
5. Sync across: All pages using this slicer
```

**Region Filter**
```
1. Insert → Slicer → Sales[Region]
2. Format: Checkbox list (multi-select)
3. Default: All selected
4. Ctrl+Click to toggle pages that use it
```

**Category Filter**
```
1. Insert → Slicer → Sales[Category]
2. Format: Dropdown (multi-select)
3. Default: All
4. Link to all charts showing category breakdown
```

**Sales Rep Filter** 
```
1. Insert → Slicer → Sales[SalesRep]
2. Format: Checkbox list
3. Link to transaction table and rep scorecard
```

### Configure Slicer Interactions

For each chart:
```
1. Click chart
2. Format → Interaction → Set slicer interaction
3. Relationship:
   - Date slicer: Filter all
   - Region slicer: Filter all
   - Category slicer: Filter sales/finance charts
   - Rep slicer: Filter detailed tables only
4. Check/uncheck boxes to control which visuals are affected
```

### Enable Drill-Through

**Setup Drill-Through Page:**
```
1. Create new blank page: "Region Detail"
2. Add Page Level Filter: Sales[Region]
3. Insert charts filtered by this region:
   - Revenue by Product (in this region)
   - Revenue by Sales Rep (in this region)
   - Sales by Month (in this region)
```

**Enable Drill-Through on Region Chart:**
```
1. Click "Revenue by Region" bar chart
2. Format → Interaction → Drill-through
3. Select "Region Detail" page as target
4. Users can right-click region → "Drill through" to see details
```

---

## PHASE 6: FORMATTING & POLISH (Week 4-5)

### 6.1 Apply Color Scheme
```
1. View → Themes → Choose professional theme (or create custom)
2. Set theme colors:
   - Primary (Revenue): #0078D4 (Blue)
   - Success (Profit): #107C10 (Green)
   - Alert (Costs): #D13438 (Red)
3. Apply to all cards and charts
```

### 6.2 Add Titles & Descriptions
```
1. Each page → Add text box with page title
2. Each visualization → Add title/description
3. Cards → Add small description (e.g., "vs Previous Month")
4. Use consistent font: Segoe UI, 14pt for titles
```

### Format Numbers Consistently
```
On each Card/Visual:
1. Select visual
2. Format → Data Labels
3. Set display units:
   - Revenue/Profit: Thousands (K) or Auto
   - Percentages: 0.0% format
   - Quantities: Whole numbers
4. Example: $100K instead of $100,000
```

### Add Conditional Formatting

On Profit Margin Card:
```
1. Select Card visual
2. Format → Data Colors → Conditional formatting
3. Rules:
   - >= 30%: Green
   - 20-30%: Yellow
   - < 20%: Red
```

On Revenue Target Gauge:
```
1. Select Gauge visual
2. Format → Gauges → Set colors:
   - Success (>= 100%): Green
   - Warning (80-100%): Yellow
   - Danger (< 80%): Red
```

---



