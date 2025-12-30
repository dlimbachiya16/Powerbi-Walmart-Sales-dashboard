Walmart Sales Analysis Dashboard - Power BI Project
Project Overview
This Power BI dashboard analyzes Walmart weekly store sales data (2010-2012) across three comprehensive pages:

Page 1: Store-Level Overview - Total sales, holiday vs non-holiday breakdown, trends by year/store
Page 2: Store Performance & Holiday Impact - Top/bottom store rankings, holiday sales contribution, quarterly trends
Page 3: Sales Driver Sensitivity - Economic drivers (Fuel Price, CPI, Unemployment, Temperature) impact analysis
​

Key Business Insights Discovered
High Fuel Prices → +28% average weekly sales ($1.13B vs $881M baseline)

High Unemployment → +19% sales (necessity shopping during recessions)

Medium Fuel + Low Unemployment = +15% optimal scenario

Holiday weeks contribute 7.5% of total sales volume despite being few weeks

Weather sensitivity: Hot temperatures correlate with higher sales

Data Model
Single Fact Table: Walmart_Sales

text
Key Columns:
- Date
- Store (45 stores)
- Weekly_Sales ($ target)
- Holiday_Flag (0/1)
- Fuel_Price ($/gallon)
- CPI (Consumer Price Index)
- Unemployment (%)
- Temperature (°F)
Calculated Columns (Modeling → New Column)
All columns use manual tertile/median logic based on Walmart dataset distributions:

text
-- Fuel Band: Fixed thresholds matching dataset quartiles
Fuel Band = 
SWITCH (
    TRUE(),
    Walmart_Sales[Fuel_Price] < 2.5, "Low",
    Walmart_Sales[Fuel_Price] < 3.0, "Medium",
    "High"
)
text
-- Unemployment Band: Median split at 8.5% (2010-2012 recession data)
Unemp Band = 
SWITCH (
    TRUE(),
    Walmart_Sales[Unemployment] <= 8.5, "Low Unemployment",
    "High Unemployment"
)
text
-- CPI Band: Matches typical CPI range 115-170
CPI Band = 
SWITCH (
    TRUE(),
    Walmart_Sales[CPI] < 130, "Low CPI",
    Walmart_Sales[CPI] < 140, "Medium CPI",
    "High CPI"
)
text
-- Temperature Band: Weather impact analysis
Temp Band = 
SWITCH (
    TRUE(),
    Walmart_Sales[Temperature] < 50, "Cold",
    Walmart_Sales[Temperature] < 70, "Mild",
    "Hot"
)
Core DAX Measures (Modeling → New Measure)
Driver KPI Cards (Page 3)
text
Average Fuel Price = AVERAGE(Walmart_Sales[Fuel_Price])
Average CPI = AVERAGE(Walmart_Sales[CPI])
Average Unemployment = AVERAGE(Walmart_Sales[Unemployment])
Average Temperature = AVERAGE(Walmart_Sales[Temperature])
Band Sensitivity Tables (Page 3)
text

Pattern repeated for all 4 drivers
Avg Sales by Fuel Band = 
AVERAGEX(
    VALUES(Walmart_Sales[Fuel Band]),
    CALCULATE(AVERAGE(Walmart_Sales[Weekly_Sales]))
)

Avg Sales by Unemp Band = 
AVERAGEX(
    VALUES(Walmart_Sales[Unemp Band]),
    CALCULATE(AVERAGE(Walmart_Sales[Weekly_Sales]))
)

Avg Sales by CPI Band = 
AVERAGEX(
    VALUES(Walmart_Sales[CPI Band]),
    CALCULATE(AVERAGE(Walmart_Sales[Weekly_Sales]))
)

Avg Sales by Temp Band = 
AVERAGEX(
    VALUES(Walmart_Sales[Temp Band]),
    CALCULATE(AVERAGE(Walmart_Sales[Weekly_Sales]))
)
Sensitivity Matrix (Page 3 Hero Visual)
text
Sales % vs Total = 
DIVIDE(
    AVERAGE(Walmart_Sales[Weekly_Sales]),
    CALCULATE(
        AVERAGE(Walmart_Sales[Weekly_Sales]), 
        ALL(Walmart_Sales)
    ),
    0
) - 1
Matrix Configuration: Rows=Fuel Band, Columns=Unemp Band, Values=Sales % vs Total

Holiday Analysis (Page 1 & 2)
text
Holiday Sales % = 
DIVIDE(
    CALCULATE(
        SUM(Walmart_Sales[Weekly_Sales]),
        Walmart_Sales[Holiday_Flag] = 1
    ),
    SUM(Walmart_Sales[Weekly_Sales])
)

Visual Implementation
Page 1: Store Overview
​

Cards: Total Sales $6.74B | Avg Weekly $1.05B | Store Count 45
Donut: Holiday 7.5% vs Non-Holiday 92.5%
Line: Weekly Sales trend by Year
Bar: Total Sales by Store (top performers)
Page 2: Store Performance
​

Cards: Total $2.0B | Avg $1.03B | Holiday Sales % 4.9%
Slicer: Year/Quarter
Line: Sales trend by week
Bar: Holiday vs Non-Holiday split
Table: Top 10 stores by total sales
Page 3: Driver Sensitivity
​

Cards: Avg Temp 60.66° | Fuel $3.36 | CPI 171.58 | Unemp 8.0%
4 Tables: Fuel/CPI/Unemp/Temp band analysis
2 Pie Charts: % by Month/Year
Manual Logic Behind Thresholds
Fuel Price: 2.5/3.0 split dataset into ~33% each (Low:2.2-2.5, Med:2.5-3.0, High:3.0+)
Unemployment: 8.5% = median of 2010-2012 recession data (6-12% range)
CPI: 130/140 splits inflation cycles (115 low, 170 high)
Temperature: 50/70°F reflects seasonal weather patterns

Results Summary
Driver	Low	Medium	High	Key Insight
Fuel	$470M (-47%)	$1.05B (+19%)	$1.13B (+28%)	High gas → Walmart traffic
Unemp	$1.07B (+21%)	-	$1.05B (+19%)	Recession = necessity shopping
CPI	$1.05B	$1.04B	$1.09B	Inflation drives value shopping
Temp	$1.07B	$1.06B	$1.10B (+25%)	Hot weather boosts sales


Reproduction Steps

Load Walmart weekly sales CSV (standard Kaggle format)

Create 4 calculated columns (copy-paste above)

Create 9+ measures (copy-paste above)

Build visuals per 3-page structure

Apply conditional formatting to matrices (Green/Red by Sales % vs Total)
