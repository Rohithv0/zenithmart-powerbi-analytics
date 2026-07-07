# Day 7 — Multi-Year Data & Time Intelligence

## What Was Built
- Imported `SalesData_2024.csv` and combined it with the existing 2025 data
  via **Append Queries** to form `Sales_AllYears`
- Updated core measures (`Total Sales`, `Total Profit`, `Running Total
  Sales`) to reference the new combined table, and corrected each
  measure's **Home Table** property (updating a measure's DAX formula does
  not automatically update its Home Table or any visual's field bindings —
  both must be changed manually)
- Built and verified:
  - **Previous Year Sales** using `SAMEPERIODLASTYEAR`
  - **YoY Growth %**
  - Attempted **Previous Quarter Sales / QoQ Growth %**

## Key Lessons

**1. Time intelligence needs the Calendar table's date column**
Functions like `TOTALYTD`, `DATESYTD`, and `SAMEPERIODLASTYEAR` must be
driven by the **Calendar table's** date column on the visual axis — never
a fact table's date column — or they silently return blank/incorrect
values.

**2. Two fact tables sharing only a dimension table don't filter each
other**
If `Sales_AllYears` and another fact table both connect to `Calendar` but
not to each other, filtering one does not automatically filter the other.
This caused flat, repeated totals in visuals until the relationship was
corrected.

**3. Blanks aren't always bugs**
- 2024 showed blank Previous Year Sales — correct, since there's no 2023
  data to compare against.
- Previous Quarter Sales / QoQ Growth % returned 0% everywhere — correct,
  because the dataset has no two consecutive complete quarters (gaps in
  Q4 2023 and Q4 2024).

**4. `DIVIDE` vs `/`**
`DIVIDE`'s third argument gracefully handles blank denominators, avoiding
divide-by-zero errors that a raw `/` would throw.

## Example Measures

```dax
Previous Year Sales =
CALCULATE(
    [Total Sales],
    SAMEPERIODLASTYEAR('Calendar'[Date])
)

YoY Growth % =
DIVIDE(
    [Total Sales] - [Previous Year Sales],
    [Previous Year Sales]
)
```
