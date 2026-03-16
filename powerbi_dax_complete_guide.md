# Power BI DAX - Complete General Reference Guide

## 📑 Table of Contents
1. [Basic Aggregations](#basic-aggregations)
2. [Conditional Logic](#conditional-logic)
3. [Text Functions](#text-functions)
4. [Mathematical Operations](#mathematical-operations)
5. [Statistical Functions](#statistical-functions)
6. [Filter & Context Functions](#filter-context-functions)
7. [Table Functions](#table-functions)
8. [Relationship Functions](#relationship-functions)
9. [Ranking & Comparison](#ranking-comparison)
10. [Variables & Optimization](#variables-optimization)

---

## 1. BASIC AGGREGATIONS

### Simple Aggregations
```dax
// Sum
Total Sales = SUM(Sales[Amount])

// Average
Average Sales = AVERAGE(Sales[Amount])

// Count (rows)
Row Count = COUNTROWS(Sales)

// Count (non-blank values)
Count Values = COUNT(Sales[Order ID])

// Count (distinct values)
Distinct Customers = DISTINCTCOUNT(Sales[Customer ID])

// Minimum
Min Price = MIN(Sales[Price])

// Maximum
Max Price = MAX(Sales[Price])

// Median
Median Sales = MEDIAN(Sales[Amount])

// Standard Deviation
StdDev Sales = STDEV.P(Sales[Amount])

// Variance
Variance Sales = VAR.P(Sales[Amount])
```

### Advanced Aggregations
```dax
// Sum with filter
Sales USA = 
CALCULATE(
    SUM(Sales[Amount]),
    Geography[Country] = "USA"
)

// Weighted Average
Weighted Avg Price = 
SUMX(Sales, Sales[Quantity] * Sales[Price]) / 
SUM(Sales[Quantity])

// Count Blank Values
Blank Count = COUNTBLANK(Sales[Ship Date])

// Count All (including blanks)
Count All = COUNTA(Sales[Product])

// Distinct Count with Calculate
Distinct Products Sold = 
CALCULATE(DISTINCTCOUNT(Sales[Product ID]))
```

---

## 2. CONDITIONAL LOGIC

### IF Statements
```dax
// Simple IF
Sales Category = 
IF(Sales[Amount] > 1000, "High", "Low")

// Nested IF
Performance Rating = 
IF(Sales[Amount] > 10000, "Excellent",
    IF(Sales[Amount] > 5000, "Good",
        IF(Sales[Amount] > 1000, "Average", "Poor")
    )
)

// IF with AND
High Value USA = 
IF(
    AND(Sales[Amount] > 1000, Geography[Country] = "USA"),
    "Yes",
    "No"
)

// IF with OR
Weekend or Holiday = 
IF(
    OR([Is Weekend] = "Yes", [Is Holiday] = "Yes"),
    "Non-Working Day",
    "Working Day"
)

// IF with Multiple Conditions (&&)
Premium Customer = 
IF(
    Sales[Amount] > 5000 && Customer[Type] = "VIP",
    "Premium",
    "Standard"
)
```

### SWITCH Function
```dax
// SWITCH (Better than nested IF)
Region Category = 
SWITCH(
    Geography[Country],
    "USA", "North America",
    "Canada", "North America",
    "Mexico", "North America",
    "UK", "Europe",
    "Germany", "Europe",
    "France", "Europe",
    "Other"  // Default value
)

// SWITCH TRUE (for ranges)
Sales Tier = 
SWITCH(
    TRUE(),
    Sales[Amount] >= 10000, "Tier 1",
    Sales[Amount] >= 5000, "Tier 2",
    Sales[Amount] >= 1000, "Tier 3",
    "Tier 4"
)

// SWITCH with calculation
Discount Rate = 
SWITCH(
    Customer[Segment],
    "Corporate", 0.15,
    "Home Office", 0.10,
    "Consumer", 0.05,
    0  // Default
)
```

### IFERROR & ISBLANK
```dax
// Handle Division by Zero
Profit Margin = 
IFERROR(
    DIVIDE(Sales[Profit], Sales[Revenue]),
    0
)

// Better: Use DIVIDE with alternate result
Profit Margin Safe = DIVIDE(Sales[Profit], Sales[Revenue], 0)

// Check for Blank
Has Ship Date = 
IF(ISBLANK(Sales[Ship Date]), "Not Shipped", "Shipped")

// Replace Blank with Value
Ship Date Display = 
IF(ISBLANK(Sales[Ship Date]), "Pending", Sales[Ship Date])

// Check if value exists
Value Exists = 
IF(NOT(ISBLANK(Sales[Product])), "Has Product", "No Product")

// COALESCE (return first non-blank)
Best Contact = 
COALESCE(Customer[Email], Customer[Phone], "No Contact")
```

---

## 3. TEXT FUNCTIONS

### Text Manipulation
```dax
// Concatenate
Full Name = Customer[First Name] & " " & Customer[Last Name]

// CONCATENATE (alternative)
Full Address = 
CONCATENATE(
    CONCATENATE(Customer[Street], ", "),
    Customer[City]
)

// CONCATENATEX (for tables)
Product List = 
CONCATENATEX(
    VALUES(Sales[Product]),
    Sales[Product],
    ", ",
    Sales[Product], ASC
)

// Upper Case
Upper Name = UPPER(Customer[Name])

// Lower Case
Lower Name = LOWER(Customer[Name])

// Proper Case (Title Case)
Proper Name = PROPER(Customer[Name])

// LEFT, RIGHT, MID
First 3 Chars = LEFT(Customer[Name], 3)
Last 3 Chars = RIGHT(Customer[Name], 3)
Middle Chars = MID(Customer[Name], 2, 5)  // Start at pos 2, length 5

// LEN (Length)
Name Length = LEN(Customer[Name])

// TRIM (Remove extra spaces)
Clean Name = TRIM(Customer[Name])

// SUBSTITUTE (Replace text)
Clean Phone = 
SUBSTITUTE(
    SUBSTITUTE(Customer[Phone], "-", ""),
    " ", ""
)

// REPLACE (Replace by position)
Masked Card = 
REPLACE(Customer[Card Number], 1, 12, "************")
```

### Text Search & Extraction
```dax
// FIND (case-sensitive, returns position)
Position = FIND("@", Customer[Email])

// SEARCH (case-insensitive, returns position)
Position Insensitive = SEARCH("gmail", Customer[Email])

// Check if text contains
Has Gmail = 
IF(
    IFERROR(SEARCH("gmail", Customer[Email]), 0) > 0,
    "Gmail User",
    "Other"
)

// CONTAINSSTRING
Contains Word = 
IF(
    CONTAINSSTRING(Product[Name], "Premium"),
    "Premium Product",
    "Standard Product"
)

// Extract domain from email
Email Domain = 
RIGHT(
    Customer[Email],
    LEN(Customer[Email]) - FIND("@", Customer[Email])
)

// Extract first word
First Word = 
LEFT(Product[Name], FIND(" ", Product[Name]) - 1)
```

### Text Formatting
```dax
// FORMAT (Number to Text)
Price Text = FORMAT(Product[Price], "$#,##0.00")

// FORMAT (Date to Text)
Date Text = FORMAT(Sales[Date], "MMMM DD, YYYY")

// FORMAT (Percentage)
Discount Pct = FORMAT([Discount Rate], "0.0%")

// FORMAT (Custom)
Custom Format = FORMAT(Sales[Amount], "#,##0.00 USD")

// FIXED (Number to text with decimals)
Fixed Price = FIXED(Product[Price], 2)

// VALUE (Text to Number)
Number Value = VALUE(Sales[Amount Text])

// REPT (Repeat character)
Star Rating = REPT("⭐", Product[Rating])
```

---

## 4. MATHEMATICAL OPERATIONS

### Basic Math
```dax
// Addition, Subtraction, Multiplication, Division
Profit = Sales[Revenue] - Sales[Cost]
Total Price = Sales[Quantity] * Sales[Unit Price]
Unit Cost = Sales[Total Cost] / Sales[Quantity]

// Power
Squared = POWER(Sales[Amount], 2)
Square Root = SQRT(Sales[Amount])

// Absolute Value
Abs Difference = ABS(Sales[Actual] - Sales[Target])

// SIGN (Returns -1, 0, or 1)
Direction = SIGN(Sales[Change])

// MOD (Remainder)
Remainder = MOD(Sales[Quantity], 10)

// QUOTIENT (Integer division)
Full Boxes = QUOTIENT(Sales[Quantity], 12)
```

### Rounding Functions
```dax
// ROUND (to nearest)
Rounded = ROUND(Sales[Amount], 2)

// ROUNDUP (always up)
Rounded Up = ROUNDUP(Sales[Amount], 0)

// ROUNDDOWN (always down)
Rounded Down = ROUNDDOWN(Sales[Amount], 0)

// INT (Remove decimals)
Integer Part = INT(Sales[Amount])

// TRUNC (Truncate)
Truncated = TRUNC(Sales[Amount], 1)

// CEILING (Round up to nearest multiple)
Ceil to 100 = CEILING(Sales[Amount], 100)

// FLOOR (Round down to nearest multiple)
Floor to 100 = FLOOR(Sales[Amount], 100)

// MROUND (Round to nearest multiple)
Round to 5 = MROUND(Sales[Amount], 5)
```

### Advanced Math
```dax
// EXP (e^x)
Exponential = EXP(2)

// LN (Natural log)
Natural Log = LN(Sales[Amount])

// LOG (Log base 10)
Log10 = LOG(Sales[Amount], 10)

// LOG10 (Log base 10)
Log Base 10 = LOG10(Sales[Amount])

// PI
Circle Area = PI() * POWER([Radius], 2)

// RAND (Random number 0-1)
Random = RAND()

// RANDBETWEEN (Random integer)
Random Num = RANDBETWEEN(1, 100)

// DEGREES / RADIANS
Degrees = DEGREES(PI())
Radians = RADIANS(180)
```

---

## 5. STATISTICAL FUNCTIONS

### Central Tendency
```dax
// Mean (Average)
Mean Sales = AVERAGE(Sales[Amount])

// AVERAGEA (includes text/logical as 0)
Average All = AVERAGEA(Sales[Amount])

// AVERAGEX (Iterate and average expression)
Avg Order Value = 
AVERAGEX(
    Sales,
    Sales[Quantity] * Sales[Price]
)

// Median
Median Price = MEDIAN(Sales[Price])

// Mode (Most frequent - need workaround)
// Power BI doesn't have built-in MODE
```

### Dispersion
```dax
// Standard Deviation (Sample)
StdDev Sample = STDEV.S(Sales[Amount])

// Standard Deviation (Population)
StdDev Pop = STDEV.P(Sales[Amount])

// Variance (Sample)
Variance Sample = VAR.S(Sales[Amount])

// Variance (Population)
Variance Pop = VAR.P(Sales[Amount])

// Range
Range = MAX(Sales[Amount]) - MIN(Sales[Amount])

// Coefficient of Variation
CV = DIVIDE([StdDev Sample], [Mean Sales], 0)
```

### Percentiles & Quartiles
```dax
// PERCENTILE.INC (Inclusive)
P50 = PERCENTILE.INC(Sales[Amount], 0.5)
P75 = PERCENTILE.INC(Sales[Amount], 0.75)
P90 = PERCENTILE.INC(Sales[Amount], 0.90)

// PERCENTILE.EXC (Exclusive)
P50 Exc = PERCENTILE.EXC(Sales[Amount], 0.5)

// PERCENTILEX.INC (with expression)
P75 Revenue = 
PERCENTILEX.INC(
    Sales,
    Sales[Quantity] * Sales[Price],
    0.75
)

// Quartiles
Q1 = PERCENTILE.INC(Sales[Amount], 0.25)
Q2 = PERCENTILE.INC(Sales[Amount], 0.50)  // Median
Q3 = PERCENTILE.INC(Sales[Amount], 0.75)

// IQR (Interquartile Range)
IQR = [Q3] - [Q1]

// Outlier Detection (below Q1-1.5*IQR or above Q3+1.5*IQR)
Is Outlier = 
IF(
    OR(
        Sales[Amount] < [Q1] - 1.5 * [IQR],
        Sales[Amount] > [Q3] + 1.5 * [IQR]
    ),
    "Outlier",
    "Normal"
)
```

---

## 6. FILTER & CONTEXT FUNCTIONS

### CALCULATE (Core Function)
```dax
// Basic CALCULATE
Sales 2024 = 
CALCULATE(
    SUM(Sales[Amount]),
    YEAR(Sales[Date]) = 2024
)

// Multiple Filters
High Value USA Sales = 
CALCULATE(
    SUM(Sales[Amount]),
    Sales[Amount] > 1000,
    Geography[Country] = "USA"
)

// Remove Filters
Total All Products = 
CALCULATE(
    SUM(Sales[Amount]),
    ALL(Product)
)

// Remove All Filters
Grand Total = 
CALCULATE(
    SUM(Sales[Amount]),
    ALL(Sales)
)

// Keep Filters on Specific Columns
Total Keep Region = 
CALCULATE(
    SUM(Sales[Amount]),
    ALL(Sales),
    VALUES(Geography[Region])
)
```

### ALL Functions
```dax
// ALL (Remove all filters from table/column)
All Products Total = 
CALCULATE(SUM(Sales[Amount]), ALL(Product))

// ALLEXCEPT (Remove all except specified)
Total Except Category = 
CALCULATE(
    SUM(Sales[Amount]),
    ALLEXCEPT(Sales, Product[Category])
)

// ALLSELECTED (Respect slicer selections)
Pct of Visible Total = 
DIVIDE(
    SUM(Sales[Amount]),
    CALCULATE(SUM(Sales[Amount]), ALLSELECTED(Sales))
)

// ALLNOBLANKROW
All No Blank = 
CALCULATE(SUM(Sales[Amount]), ALLNOBLANKROW(Product))
```

### FILTER Functions
```dax
// FILTER (Return filtered table)
High Value Orders = 
CALCULATE(
    COUNTROWS(Sales),
    FILTER(Sales, Sales[Amount] > 1000)
)

// Multiple conditions in FILTER
Complex Filter = 
CALCULATE(
    SUM(Sales[Amount]),
    FILTER(
        Sales,
        Sales[Amount] > 1000 && 
        YEAR(Sales[Date]) = 2024
    )
)

// KEEPFILTERS (Combine with existing filters)
Keep Filter Sales = 
CALCULATE(
    SUM(Sales[Amount]),
    KEEPFILTERS(Product[Category] = "Electronics")
)

// REMOVEFILTERS (Clear filters)
Clear Category Filter = 
CALCULATE(
    SUM(Sales[Amount]),
    REMOVEFILTERS(Product[Category])
)
```

### VALUES & DISTINCT
```dax
// VALUES (Unique values respecting filters)
Unique Products = 
COUNTROWS(VALUES(Product[Name]))

// DISTINCT (Unique values ignoring blank row)
Distinct Products = 
COUNTROWS(DISTINCT(Product[Name]))

// HASONEVALUE (Check if single value selected)
Selected Product = 
IF(
    HASONEVALUE(Product[Name]),
    VALUES(Product[Name]),
    "Multiple Products"
)

// HASONEFILTER (Check if single filter)
Has Single Filter = HASONEFILTER(Product[Category])

// ISCROSSFILTERED (Check if filtered by related table)
Is Filtered = ISCROSSFILTERED(Product[Category])

// ISFILTERED (Check if direct filter applied)
Is Direct Filtered = ISFILTERED(Product[Category])
```

### SELECTEDVALUE
```dax
// Get single selected value (returns BLANK if multiple)
Selected Category = SELECTEDVALUE(Product[Category])

// With alternate result
Selected or Default = 
SELECTEDVALUE(Product[Category], "All Categories")

// Use in conditional formatting
Color by Selection = 
IF(
    SELECTEDVALUE(Product[Category]) = "Electronics",
    "Blue",
    "Gray"
)
```

---

## 7. TABLE FUNCTIONS

### Creating Tables
```dax
// SUMMARIZE (Group by)
Summary Table = 
SUMMARIZE(
    Sales,
    Product[Category],
    Product[SubCategory],
    "Total Sales", SUM(Sales[Amount]),
    "Order Count", COUNTROWS(Sales)
)

// ADDCOLUMNS (Add calculated columns to table)
Enhanced Table = 
ADDCOLUMNS(
    VALUES(Product[Name]),
    "Total Sales", CALCULATE(SUM(Sales[Amount])),
    "Avg Price", CALCULATE(AVERAGE(Sales[Price]))
)

// SELECTCOLUMNS (Select specific columns)
Selected Cols = 
SELECTCOLUMNS(
    Sales,
    "Date", Sales[Order Date],
    "Amount", Sales[Amount]
)

// GENERATESERIES (Create number sequence)
Number Series = GENERATESERIES(1, 100, 1)

// GENERATEALL (Cartesian product)
All Combinations = 
GENERATESERIES(Product[Category], Geography[Region])
```

### Table Manipulation
```dax
// TOPN (Top N rows)
Top 10 Products = 
TOPN(
    10,
    ALL(Product[Name]),
    CALCULATE(SUM(Sales[Amount])),
    DESC
)

// SAMPLE (Random sample)
Random Sample = SAMPLE(100, Sales, Sales[Order ID])

// UNION (Combine tables)
Combined = 
UNION(
    SELECTCOLUMNS(Sales2023, "Year", 2023, "Amount", Sales2023[Amount]),
    SELECTCOLUMNS(Sales2024, "Year", 2024, "Amount", Sales2024[Amount])
)

// EXCEPT (Rows in first but not in second)
Difference = EXCEPT(AllProducts, SoldProducts)

// INTERSECT (Common rows)
Common = INTERSECT(Table1, Table2)

// CROSSJOIN
Cross = CROSSJOIN(Product[Category], Geography[Region])
```

### Table Filtering
```dax
// CALCULATETABLE (CALCULATE for tables)
Filtered Products = 
CALCULATETABLE(
    Product,
    Product[Category] = "Electronics",
    Product[Price] > 100
)

// ALL variations with tables
All Products = ALL(Product)
All Except Price = ALLEXCEPT(Product, Product[Price])

// FILTERS (Current filter on column)
Current Filters = FILTERS(Product[Category])
```

---

## 8. RELATIONSHIP FUNCTIONS

### RELATED & RELATEDTABLE
```dax
// RELATED (Many to One - Get value from "One" side)
Product Category = RELATED(Product[Category])

// Multiple levels
Manufacturer Country = 
RELATED(Product[Manufacturer Country])

// RELATEDTABLE (One to Many - Get related rows)
Related Sales Count = 
COUNTROWS(RELATEDTABLE(Sales))

// Sum related values
Total Order Value = 
SUMX(RELATEDTABLE(OrderDetails), OrderDetails[Amount])
```

### USERELATIONSHIP
```dax
// Activate inactive relationship
Sales by Ship Date = 
CALCULATE(
    SUM(Sales[Amount]),
    USERELATIONSHIP(Sales[Ship Date], 'Date'[Date])
)

// Multiple inactive relationships
Delivered Sales = 
CALCULATE(
    SUM(Sales[Amount]),
    USERELATIONSHIP(Sales[Delivery Date], 'Date'[Date])
)
```

### CROSSFILTER
```dax
// Change filter direction temporarily
Reverse Filter = 
CALCULATE(
    COUNTROWS(Sales),
    CROSSFILTER(Product[ID], Sales[Product ID], Both)
)

// Disable relationship
No Filter = 
CALCULATE(
    SUM(Sales[Amount]),
    CROSSFILTER(Product[ID], Sales[Product ID], None)
)
```

---

## 9. RANKING & COMPARISON

### RANKX
```dax
// Basic Rank
Product Rank = 
RANKX(
    ALL(Product[Name]),
    CALCULATE(SUM(Sales[Amount])),
    ,
    DESC,
    Dense
)

// Rank within Category
Rank in Category = 
RANKX(
    ALLSELECTED(Product[Name]),
    CALCULATE(SUM(Sales[Amount])),
    ,
    DESC
)

// Rank with ties
Rank Dense = 
RANKX(
    ALL(Product[Name]),
    [Total Sales],
    ,
    DESC,
    Dense  // 1,2,2,3 instead of 1,2,2,4
)

// Rank Skip
Rank Skip = 
RANKX(
    ALL(Product[Name]),
    [Total Sales],
    ,
    DESC,
    Skip  // 1,2,2,4
)
```

### Top/Bottom N
```dax
// Is in Top 10
Is Top 10 = 
IF([Product Rank] <= 10, "Top 10", "Other")

// Top N by Category
Top 5 in Category = 
VAR CurrentCategory = SELECTEDVALUE(Product[Category])
VAR RankInCat = 
    RANKX(
        FILTER(ALL(Product), Product[Category] = CurrentCategory),
        [Total Sales],
        ,
        DESC
    )
RETURN
    IF(RankInCat <= 5, "Top 5", "Other")

// Dynamic Top N
Top N Products = 
VAR N = [Selected N Value]  // From slicer
RETURN
    IF([Product Rank] <= N, "Top " & N, "Other")
```

### Comparison Functions
```dax
// Earlier (Previous row in table)
Running Total = 
SUMX(
    FILTER(
        ALL(Sales),
        Sales[Date] <= EARLIER(Sales[Date])
    ),
    Sales[Amount]
)

// EARLIER with multiple levels
Nested Earlier = 
SUMX(
    Product,
    SUMX(
        FILTER(
            Sales,
            Sales[Product ID] = EARLIER(Product[ID])
        ),
        Sales[Amount]
    )
)
```

---

## 10. VARIABLES & OPTIMIZATION

### Using Variables
```dax
// Basic VAR
Profit Margin = 
VAR Revenue = SUM(Sales[Revenue])
VAR Cost = SUM(Sales[Cost])
VAR Profit = Revenue - Cost
RETURN
    DIVIDE(Profit, Revenue, 0)

// Multiple calculations with VAR
Customer Metrics = 
VAR TotalCustomers = DISTINCTCOUNT(Sales[Customer ID])
VAR TotalOrders = COUNTROWS(Sales)
VAR AvgOrdersPerCustomer = DIVIDE(TotalOrders, TotalCustomers, 0)
RETURN
    AvgOrdersPerCustomer

// VAR with table
Top Products List = 
VAR TopProducts = 
    TOPN(
        10,
        ALL(Product[Name]),
        [Total Sales],
        DESC
    )
RETURN
    CONCATENATEX(TopProducts, Product[Name], ", ")
```

### Performance Optimization
```dax
// Use variables to avoid recalculation
Optimized Measure = 
VAR BaseValue = CALCULATE(SUM(Sales[Amount]))
VAR LastYear = CALCULATE(SUM(Sales[Amount]), SAMEPERIODLASTYEAR('Date'[Date]))
VAR Growth = DIVIDE(BaseValue - LastYear, LastYear, 0)
RETURN
    IF(Growth > 0.1, "High Growth", "Normal")

// Use KEEPFILTERS instead of AND
Better Filter = 
CALCULATE(
    SUM(Sales[Amount]),
    KEEPFILTERS(Product[Category] = "Electronics")
)

// Avoid nested CALCULATE
// Bad:
Nested Calc = 
CALCULATE(
    CALCULATE(SUM(Sales[Amount]), Product[Category] = "A"),
    Geography[Region] = "West"
)

// Good:
Flat Calc = 
CALCULATE(
    SUM(Sales[Amount]),
    Product[Category] = "A",
    Geography[Region] = "West"
)
```

---

## 📌 QUICK REFERENCE - Common Patterns

### 1. Cumulative Total
```dax
Cumulative Sales = 
CALCULATE(
    SUM(Sales[Amount]),
    FILTER(
        ALL('Date'[Date]),
        'Date'[Date] <= MAX('Date'[Date])
    )
)
```

### 2. Percentage of Total
```dax
% of Total = 
DIVIDE(
    SUM(Sales[Amount]),
    CALCULATE(SUM(Sales[Amount]), ALL(Product)),
    0
)
```

### 3. Same Period Last Year
```dax
Sales LY = 
CALCULATE(
    SUM(Sales[Amount]),
    SAMEPERIODLASTYEAR('Date'[Date])
)
```

### 4. Year-over-Year Growth %
```dax
YoY Growth % = 
VAR CurrentYear = SUM(Sales[Amount])
VAR LastYear = [Sales LY]
RETURN
    DIVIDE(CurrentYear - LastYear, LastYear, 0)
```

### 5. Moving Average
```dax
Moving Avg 3 Months = 
CALCULATE(
    AVERAGE(Sales[Amount]),
    DATESINPERIOD('Date'[Date], LASTDATE('Date'[Date]), -3, MONTH)
)
```

### 6. Conditional Formatting Value
```dax
Status Color = 
SWITCH(
    TRUE(),
    [YoY Growth %] > 0.1, "Green",
    [YoY Growth %] < -0.1, "Red",
    "Yellow"
)
```

### 7. Dynamic Measure Selector
```dax
Selected Measure = 
SWITCH(
    SELECTEDVALUE(MeasureSelector[Measure]),
    "Sales", [Total Sales],
    "Profit", [Total Profit],
    "Quantity", [Total Quantity],
    [Total Sales]  // Default
)
```

### 8. ABC Analysis
```dax
ABC Category = 
VAR CumulativePct = 
    DIVIDE(
        [Cumulative Sales],
        CALCULATE(SUM(Sales[Amount]), ALL(Product)),
        0
    )
RETURN
    SWITCH(
        TRUE(),
        CumulativePct <= 0.8, "A",
        CumulativePct <= 0.95, "B",
        "C"
    )
```

### 9. Pareto 80/20
```dax
Is Top 80% = 
IF([ABC Category] = "A", "Top 80%", "Bottom 20%")
```

### 10. Custom Sorting
```dax
Month Sort = 
SWITCH(
    [Month Name],
    "January", 1, "February", 2, "March", 3,
    "April", 4, "May", 5, "June", 6,
    "July", 7, "August", 8, "September", 9,
    "October", 10, "November", 11, "December", 12
)
```

---

## 💡 BEST PRACTICES

### ✅ DO's
1. **Use Variables** - Store intermediate calculations
2. **Name Clearly** - Use descriptive measure names
3. **Format Results** - Apply appropriate number formats
4. **Comment Complex DAX** - Add explanations
5. **Test Edge Cases** - Verify with empty data, nulls
6. **Avoid Nested CALCULATE** - Flatten when possible
7. **Use DIVIDE** - Instead of division operator
8. **Document Dependencies** - Note related measures
9. **Optimize Filters** - Put most restrictive filters first
10. **Use Measure Groups** - Organize related measures

### ❌ DON'Ts
1. **Don't Use Calculated Columns** - When measures work
2. **Avoid Complex Nested IFs** - Use SWITCH instead
3. **Don't Repeat Calculations** - Use variables
4. **Avoid Implicit Measures** - Create explicit ones
5. **Don't Ignore Blanks** - Handle with ISBLANK or DIVIDE
6. **Avoid Cross-Filtering Issues** - Be explicit
7. **Don't Ignore Relationships** - Leverage data model
8. **Avoid Text in Calculations** - Use IDs instead
9. **Don't Over-Use ALL()** - Can impact performance
10. **Avoid Magic Numbers** - Use parameters/variables

---

## 🎯 CHEAT SHEET

| Category | Common Functions |
|----------|-----------------|
| **Aggregation** | SUM, AVERAGE, COUNT, DISTINCTCOUNT, MIN, MAX |
| **Logical** | IF, SWITCH, AND, OR, NOT, IFERROR |
| **Text** | CONCATENATE, LEFT, RIGHT, MID, UPPER, LOWER, FORMAT |
| **Math** | ROUND, ABS, POWER, SQRT, MOD, DIVIDE |
| **Filter** | CALCULATE, FILTER, ALL, ALLEXCEPT, VALUES |
| **Time** | DATESYTD, SAMEPERIODLASTYEAR, DATEADD, TOTALYTD |
| **Table** | SUMMARIZE, ADDCOLUMNS, TOPN, UNION, CROSSJOIN |
| **Relationship** | RELATED, RELATEDTABLE, USERELATIONSHIP |
| **Ranking** | RANKX, TOPN, EARLIER |
| **Info** | ISBLANK, HASONEVALUE, SELECTEDVALUE, ISFILTERED |

---

**Happy DAX Coding! 🚀📊**
