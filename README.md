# Power BI DAX - Complete Date Operations Guide

## 📅 Table of Contents
1. [Date Table Creation](#date-table-creation)
2. [Extract Date Components](#extract-date-components)
3. [Date Calculations](#date-calculations)
4. [Time Intelligence](#time-intelligence)
5. [Comparison Calculations](#comparison-calculations)
6. [Custom Date Columns](#custom-date-columns)

---

## 1. DATE TABLE CREATION

### Create Complete Date Table
```dax
DateTable = 
ADDCOLUMNS(
    CALENDAR(DATE(2018, 1, 1), DATE(2025, 12, 31)),
    "Year", YEAR([Date]),
    "Month Number", MONTH([Date]),
    "Month Name", FORMAT([Date], "MMMM"),
    "Month Short", FORMAT([Date], "MMM"),
    "Quarter", "Q" & QUARTER([Date]),
    "Quarter Number", QUARTER([Date]),
    "Day", DAY([Date]),
    "Day Name", FORMAT([Date], "DDDD"),
    "Day Short", FORMAT([Date], "DDD"),
    "Day of Week", WEEKDAY([Date]),
    "Day of Year", DATEDIFF(DATE(YEAR([Date]), 1, 1), [Date], DAY) + 1,
    "Week Number", WEEKNUM([Date]),
    "Is Weekend", IF(WEEKDAY([Date]) IN {1, 7}, "Yes", "No"),
    "Year-Month", FORMAT([Date], "YYYY-MM"),
    "Year-Quarter", YEAR([Date]) & "-Q" & QUARTER([Date])
)
```

### Simple Date Table (Auto-generate from existing data)
```dax
DateTable = 
CALENDARAUTO()
```

### Date Table from Min/Max Dates in Data
```dax
DateTable = 
CALENDAR(
    MIN(Sales[Order Date]),
    MAX(Sales[Order Date])
)
```

---

## 2. EXTRACT DATE COMPONENTS

### Year Operations
```dax
// Extract Year
Year = YEAR(Sales[Order Date])

// Year as Text
Year Text = FORMAT(Sales[Order Date], "YYYY")

// Fiscal Year (April start)
Fiscal Year = 
IF(
    MONTH(Sales[Order Date]) >= 4,
    YEAR(Sales[Order Date]),
    YEAR(Sales[Order Date]) - 1
)

// Year-Month
Year-Month = FORMAT(Sales[Order Date], "YYYY-MM")

// Current Year Flag
Is Current Year = 
IF(YEAR(Sales[Order Date]) = YEAR(TODAY()), "Yes", "No")
```

### Month Operations
```dax
// Month Number
Month Number = MONTH(Sales[Order Date])

// Month Name (Full)
Month Name = FORMAT(Sales[Order Date], "MMMM")

// Month Name (Short)
Month Short = FORMAT(Sales[Order Date], "MMM")

// Month-Year
Month-Year = FORMAT(Sales[Order Date], "MMM YYYY")

// Start of Month
Start of Month = STARTOFMONTH(Sales[Order Date])

// End of Month
End of Month = ENDOFMONTH(Sales[Order Date])

// Current Month Flag
Is Current Month = 
IF(
    YEAR(Sales[Order Date]) = YEAR(TODAY()) &&
    MONTH(Sales[Order Date]) = MONTH(TODAY()),
    "Yes", "No"
)
```

### Quarter Operations
```dax
// Quarter Number
Quarter Number = QUARTER(Sales[Order Date])

// Quarter Label
Quarter = "Q" & QUARTER(Sales[Order Date])

// Year-Quarter
Year-Quarter = 
YEAR(Sales[Order Date]) & "-Q" & QUARTER(Sales[Order Date])

// Start of Quarter
Start of Quarter = STARTOFQUARTER(Sales[Order Date])

// End of Quarter
End of Quarter = ENDOFQUARTER(Sales[Order Date])

// Fiscal Quarter (April start)
Fiscal Quarter = 
VAR MonthNum = MONTH(Sales[Order Date])
RETURN
    SWITCH(
        TRUE(),
        MonthNum >= 4 && MonthNum <= 6, "Q1",
        MonthNum >= 7 && MonthNum <= 9, "Q2",
        MonthNum >= 10 && MonthNum <= 12, "Q3",
        "Q4"
    )
```

### Day Operations
```dax
// Day of Month
Day = DAY(Sales[Order Date])

// Day Name (Full)
Day Name = FORMAT(Sales[Order Date], "DDDD")

// Day Name (Short)
Day Short = FORMAT(Sales[Order Date], "DDD")

// Day of Week (1=Sunday, 7=Saturday)
Day of Week = WEEKDAY(Sales[Order Date])

// Day of Week (1=Monday, 7=Sunday)
Day of Week Monday = WEEKDAY(Sales[Order Date], 2)

// Day of Year
Day of Year = 
DATEDIFF(
    DATE(YEAR(Sales[Order Date]), 1, 1),
    Sales[Order Date],
    DAY
) + 1

// Is Weekend
Is Weekend = 
IF(WEEKDAY(Sales[Order Date]) IN {1, 7}, "Weekend", "Weekday")

// Is Business Day (Mon-Fri)
Is Business Day = 
IF(WEEKDAY(Sales[Order Date]) IN {2, 3, 4, 5, 6}, "Yes", "No")
```

### Week Operations
```dax
// Week Number (Calendar Year)
Week Number = WEEKNUM(Sales[Order Date])

// Week Number (ISO 8601 - Week starting Monday)
Week Number ISO = WEEKNUM(Sales[Order Date], 21)

// Start of Week (Sunday)
Start of Week = Sales[Order Date] - WEEKDAY(Sales[Order Date]) + 1

// Start of Week (Monday)
Start of Week Monday = 
Sales[Order Date] - WEEKDAY(Sales[Order Date], 2) + 1

// Week Label
Week Label = 
"Week " & WEEKNUM(Sales[Order Date])
```

---

## 3. DATE CALCULATIONS

### Date Difference Calculations
```dax
// Days Between Dates
Days Between = 
DATEDIFF(Sales[Order Date], Sales[Ship Date], DAY)

// Months Between Dates
Months Between = 
DATEDIFF(Sales[Order Date], Sales[Ship Date], MONTH)

// Years Between Dates
Years Between = 
DATEDIFF(Sales[Order Date], Sales[Ship Date], YEAR)

// Age Calculation (Years)
Age in Years = 
DATEDIFF(Customer[Birth Date], TODAY(), YEAR)

// Days Since Order
Days Since Order = 
DATEDIFF(Sales[Order Date], TODAY(), DAY)

// Business Days Between (excluding weekends)
Business Days = 
VAR StartDate = Sales[Order Date]
VAR EndDate = Sales[Ship Date]
VAR TotalDays = DATEDIFF(StartDate, EndDate, DAY)
VAR Weeks = INT(TotalDays / 7)
VAR RemainderDays = MOD(TotalDays, 7)
VAR WeekendDays = Weeks * 2
VAR StartDayOfWeek = WEEKDAY(StartDate)
VAR AdditionalWeekendDays = 
    COUNTROWS(
        FILTER(
            GENERATESERIES(0, RemainderDays - 1),
            WEEKDAY(StartDate + [Value]) IN {1, 7}
        )
    )
RETURN
    TotalDays - WeekendDays - AdditionalWeekendDays
```

### Date Addition/Subtraction
```dax
// Add Days
Date Plus 7 Days = Sales[Order Date] + 7

// Add Months
Date Plus 1 Month = 
EDATE(Sales[Order Date], 1)

// Add Years
Date Plus 1 Year = 
EDATE(Sales[Order Date], 12)

// Subtract Days
Date Minus 30 Days = Sales[Order Date] - 30

// Next Month Same Day
Next Month = 
DATE(
    YEAR(Sales[Order Date]),
    MONTH(Sales[Order Date]) + 1,
    DAY(Sales[Order Date])
)

// Last Day of Previous Month
Last Day Previous Month = 
EOMONTH(Sales[Order Date], -1)
```

---

## 4. TIME INTELLIGENCE FUNCTIONS

### Year-to-Date (YTD)
```dax
// YTD Total
Sales YTD = 
TOTALYTD(
    SUM(Sales[Amount]),
    'DateTable'[Date]
)

// YTD with Custom Year End (e.g., June 30)
Sales YTD Fiscal = 
TOTALYTD(
    SUM(Sales[Amount]),
    'DateTable'[Date],
    "6/30"
)

// YTD Average
Avg Sales YTD = 
AVERAGEX(
    DATESYTD('DateTable'[Date]),
    [Total Sales]
)
```

### Quarter-to-Date (QTD)
```dax
// QTD Total
Sales QTD = 
TOTALQTD(
    SUM(Sales[Amount]),
    'DateTable'[Date]
)

// QTD Count
Orders QTD = 
CALCULATE(
    COUNTROWS(Sales),
    DATESQTD('DateTable'[Date])
)
```

### Month-to-Date (MTD)
```dax
// MTD Total
Sales MTD = 
TOTALMTD(
    SUM(Sales[Amount]),
    'DateTable'[Date]
)

// MTD vs Target
MTD vs Target = 
[Sales MTD] - [Target MTD]
```

### Previous Period Calculations
```dax
// Previous Year Same Period
Sales PY = 
CALCULATE(
    SUM(Sales[Amount]),
    SAMEPERIODLASTYEAR('DateTable'[Date])
)

// Previous Month
Sales Previous Month = 
CALCULATE(
    SUM(Sales[Amount]),
    PREVIOUSMONTH('DateTable'[Date])
)

// Previous Quarter
Sales Previous Quarter = 
CALCULATE(
    SUM(Sales[Amount]),
    PREVIOUSQUARTER('DateTable'[Date])
)

// Previous Year
Sales Previous Year = 
CALCULATE(
    SUM(Sales[Amount]),
    PREVIOUSYEAR('DateTable'[Date])
)
```

### Date Offset Calculations
```dax
// Last 7 Days
Sales Last 7 Days = 
CALCULATE(
    SUM(Sales[Amount]),
    DATESINPERIOD(
        'DateTable'[Date],
        LASTDATE('DateTable'[Date]),
        -7,
        DAY
    )
)

// Last 30 Days
Sales Last 30 Days = 
CALCULATE(
    SUM(Sales[Amount]),
    DATESINPERIOD(
        'DateTable'[Date],
        LASTDATE('DateTable'[Date]),
        -30,
        DAY
    )
)

// Last 3 Months
Sales Last 3 Months = 
CALCULATE(
    SUM(Sales[Amount]),
    DATESINPERIOD(
        'DateTable'[Date],
        LASTDATE('DateTable'[Date]),
        -3,
        MONTH
    )
)

// Last 12 Months
Sales Last 12 Months = 
CALCULATE(
    SUM(Sales[Amount]),
    DATESINPERIOD(
        'DateTable'[Date],
        LASTDATE('DateTable'[Date]),
        -12,
        MONTH
    )
)

// Rolling 12 Months
Rolling 12 Months Sales = 
CALCULATE(
    SUM(Sales[Amount]),
    DATESBETWEEN(
        'DateTable'[Date],
        EDATE(MAX('DateTable'[Date]), -12),
        MAX('DateTable'[Date])
    )
)
```

---

## 5. COMPARISON CALCULATIONS

### Year-over-Year (YoY)
```dax
// YoY Growth
Sales YoY Growth = 
VAR CurrentYear = SUM(Sales[Amount])
VAR PreviousYear = CALCULATE(
    SUM(Sales[Amount]),
    SAMEPERIODLASTYEAR('DateTable'[Date])
)
RETURN
    CurrentYear - PreviousYear

// YoY Growth %
Sales YoY Growth % = 
VAR CurrentYear = SUM(Sales[Amount])
VAR PreviousYear = CALCULATE(
    SUM(Sales[Amount]),
    SAMEPERIODLASTYEAR('DateTable'[Date])
)
RETURN
    DIVIDE(CurrentYear - PreviousYear, PreviousYear, 0)

// YoY Growth % Formatted
YoY Growth % Text = 
FORMAT([Sales YoY Growth %], "0.0%")
```

### Month-over-Month (MoM)
```dax
// MoM Growth
Sales MoM Growth = 
VAR CurrentMonth = SUM(Sales[Amount])
VAR PreviousMonth = CALCULATE(
    SUM(Sales[Amount]),
    PREVIOUSMONTH('DateTable'[Date])
)
RETURN
    CurrentMonth - PreviousMonth

// MoM Growth %
Sales MoM Growth % = 
VAR CurrentMonth = SUM(Sales[Amount])
VAR PreviousMonth = CALCULATE(
    SUM(Sales[Amount]),
    PREVIOUSMONTH('DateTable'[Date])
)
RETURN
    DIVIDE(CurrentMonth - PreviousMonth, PreviousMonth, 0)
```

### Quarter-over-Quarter (QoQ)
```dax
// QoQ Growth
Sales QoQ Growth = 
VAR CurrentQuarter = SUM(Sales[Amount])
VAR PreviousQuarter = CALCULATE(
    SUM(Sales[Amount]),
    PREVIOUSQUARTER('DateTable'[Date])
)
RETURN
    CurrentQuarter - PreviousQuarter

// QoQ Growth %
Sales QoQ Growth % = 
VAR CurrentQuarter = SUM(Sales[Amount])
VAR PreviousQuarter = CALCULATE(
    SUM(Sales[Amount]),
    PREVIOUSQUARTER('DateTable'[Date])
)
RETURN
    DIVIDE(CurrentQuarter - PreviousQuarter, PreviousQuarter, 0)
```

### Variance Analysis
```dax
// Variance vs Previous Year
Variance vs PY = [Total Sales] - [Sales PY]

// Variance vs Target
Variance vs Target = [Total Sales] - [Target Sales]

// Variance % vs Previous Year
Variance % vs PY = 
DIVIDE([Total Sales] - [Sales PY], [Sales PY], 0)

// Variance Status
Variance Status = 
SWITCH(
    TRUE(),
    [Variance vs Target] > 0, "Above Target",
    [Variance vs Target] < 0, "Below Target",
    "On Target"
)
```

---

## 6. CUSTOM DATE COLUMNS

### Dynamic Date Filters
```dax
// Current Date Flag
Is Current Date = 
IF('DateTable'[Date] = TODAY(), "Today", "Other")

// Date Range Categories
Date Category = 
SWITCH(
    TRUE(),
    'DateTable'[Date] = TODAY(), "Today",
    'DateTable'[Date] = TODAY() - 1, "Yesterday",
    'DateTable'[Date] >= TODAY() - 7, "Last 7 Days",
    'DateTable'[Date] >= TODAY() - 30, "Last 30 Days",
    'DateTable'[Date] >= TODAY() - 90, "Last 90 Days",
    "Older"
)

// Relative Date
Relative Date = 
VAR DaysDiff = DATEDIFF('DateTable'[Date], TODAY(), DAY)
RETURN
    SWITCH(
        TRUE(),
        DaysDiff = 0, "Today",
        DaysDiff = -1, "Yesterday",
        DaysDiff = 1, "Tomorrow",
        DaysDiff > 1, DaysDiff & " days ahead",
        DaysDiff < -1, ABS(DaysDiff) & " days ago"
    )
```

### Age Calculations
```dax
// Customer Age
Customer Age = 
DATEDIFF(Customer[Birth Date], TODAY(), YEAR)

// Age Group
Age Group = 
SWITCH(
    TRUE(),
    [Customer Age] < 18, "Under 18",
    [Customer Age] < 25, "18-24",
    [Customer Age] < 35, "25-34",
    [Customer Age] < 45, "35-44",
    [Customer Age] < 55, "45-54",
    [Customer Age] < 65, "55-64",
    "65+"
)

// Tenure (Years with Company)
Tenure Years = 
DATEDIFF(Employee[Hire Date], TODAY(), YEAR)

// Tenure Group
Tenure Group = 
SWITCH(
    TRUE(),
    [Tenure Years] < 1, "Less than 1 year",
    [Tenure Years] < 3, "1-3 years",
    [Tenure Years] < 5, "3-5 years",
    [Tenure Years] < 10, "5-10 years",
    "10+ years"
)
```

### Seasonal Columns
```dax
// Season
Season = 
VAR MonthNum = MONTH('DateTable'[Date])
RETURN
    SWITCH(
        TRUE(),
        MonthNum IN {12, 1, 2}, "Winter",
        MonthNum IN {3, 4, 5}, "Spring",
        MonthNum IN {6, 7, 8}, "Summer",
        "Fall"
    )

// Holiday Flag (US)
Is Holiday = 
VAR DateValue = 'DateTable'[Date]
VAR MonthValue = MONTH(DateValue)
VAR DayValue = DAY(DateValue)
RETURN
    SWITCH(
        TRUE(),
        MonthValue = 1 && DayValue = 1, "New Year's Day",
        MonthValue = 7 && DayValue = 4, "Independence Day",
        MonthValue = 12 && DayValue = 25, "Christmas",
        "No"
    )

// Business Quarter (Fiscal)
Business Quarter = 
VAR MonthNum = MONTH('DateTable'[Date])
VAR FiscalMonth = IF(MonthNum >= 4, MonthNum - 3, MonthNum + 9)
RETURN
    "Q" & ROUNDUP(FiscalMonth / 3, 0)
```

### Working Days Calculations
```dax
// Working Days in Month
Working Days in Month = 
VAR StartDate = STARTOFMONTH('DateTable'[Date])
VAR EndDate = ENDOFMONTH('DateTable'[Date])
RETURN
    COUNTROWS(
        FILTER(
            CALENDAR(StartDate, EndDate),
            WEEKDAY([Date]) NOT IN {1, 7}
        )
    )

// Remaining Working Days in Month
Remaining Working Days = 
VAR CurrentDate = TODAY()
VAR EndDate = ENDOFMONTH(CurrentDate)
RETURN
    COUNTROWS(
        FILTER(
            CALENDAR(CurrentDate, EndDate),
            WEEKDAY([Date]) NOT IN {1, 7}
        )
    )

// Elapsed Working Days in Month
Elapsed Working Days = 
VAR StartDate = STARTOFMONTH(TODAY())
VAR CurrentDate = TODAY()
RETURN
    COUNTROWS(
        FILTER(
            CALENDAR(StartDate, CurrentDate),
            WEEKDAY([Date]) NOT IN {1, 7}
        )
    )
```

---

## 📌 QUICK REFERENCE CHEAT SHEET

### Common Date Functions
```dax
TODAY()                          // Current date
NOW()                            // Current date and time
DATE(2024, 12, 31)              // Create specific date
YEAR([Date])                     // Extract year
MONTH([Date])                    // Extract month (1-12)
DAY([Date])                      // Extract day (1-31)
WEEKDAY([Date])                  // Day of week (1-7)
WEEKNUM([Date])                  // Week number
QUARTER([Date])                  // Quarter (1-4)
EOMONTH([Date], 0)              // End of month
EDATE([Date], 1)                // Add/subtract months
DATEDIFF([Date1], [Date2], DAY) // Difference between dates
FORMAT([Date], "YYYY-MM-DD")    // Format date
```

### Time Intelligence Functions
```dax
TOTALYTD(expression, dates)
TOTALMTD(expression, dates)
TOTALQTD(expression, dates)
SAMEPERIODLASTYEAR(dates)
PREVIOUSMONTH(dates)
PREVIOUSQUARTER(dates)
PREVIOUSYEAR(dates)
DATESINPERIOD(dates, start, number, interval)
DATESBETWEEN(dates, start, end)
STARTOFMONTH(dates)
ENDOFMONTH(dates)
STARTOFQUARTER(dates)
ENDOFQUARTER(dates)
STARTOFYEAR(dates)
ENDOFYEAR(dates)
```

---

## 💡 BEST PRACTICES

1. **Always create a Date Table** - Don't rely on auto-date tables
2. **Mark as Date Table** - Right-click → Mark as Date Table
3. **Use CALCULATE** - Wrap time intelligence in CALCULATE when needed
4. **Handle Blanks** - Use ISBLANK() or DIVIDE() with alternate result
5. **Optimize Performance** - Use variables (VAR) for repeated calculations
6. **Name Consistently** - Use clear, descriptive measure names
7. **Format Results** - Apply appropriate number/date formats
8. **Test Edge Cases** - Verify with leap years, month-end dates
9. **Document Measures** - Add descriptions for complex DAX
10. **Use Date Hierarchies** - Year > Quarter > Month > Day

---

## 🎯 Common Use Cases

### Sales Dashboard
```dax
// Current Month Sales
Current Month Sales = 
CALCULATE(
    SUM(Sales[Amount]),
    DATESMTD('DateTable'[Date])
)

// vs Last Month
vs Last Month = 
[Current Month Sales] - 
CALCULATE(
    SUM(Sales[Amount]),
    PREVIOUSMONTH('DateTable'[Date])
)

// vs Same Month Last Year
vs Same Month LY = 
[Current Month Sales] - 
CALCULATE(
    SUM(Sales[Amount]),
    SAMEPERIODLASTYEAR('DateTable'[Date])
)
```

### Performance Metrics
```dax
// Daily Average (MTD)
Daily Avg MTD = 
DIVIDE(
    [Sales MTD],
    COUNTROWS(DATESMTD('DateTable'[Date])),
    0
)

// Run Rate (Annual projection)
Annual Run Rate = 
[Sales YTD] * 
DIVIDE(365, [Days Elapsed in Year], 1)

// Days Elapsed in Year
Days Elapsed in Year = 
DATEDIFF(
    DATE(YEAR(TODAY()), 1, 1),
    TODAY(),
    DAY
) + 1
```
