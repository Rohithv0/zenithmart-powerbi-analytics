# Day 8 — SUMX, RANKX, and Top N (with real-world data debugging)

## SUMX vs. a Stored Column

Built `Total Net Amount` as a `SUMX`-based iterator measure and found it
disagreed with the existing stored `Net Amount` column (311.20K vs.
225.80K).

**Root cause:** The stored column's formula —
`[Amount] - ([ReturnedQuantity] * [UnitPrice])` — was calculated in Power
Query *before* the 2024 data existed in the model. Appending new rows
later does not retroactively recalculate old computed columns.

**Conclusion:** The `SUMX` measure is correct; the stored column is stale.
Going forward, the SUMX measure (`Total Net Amount (SUMX)`) is the
authoritative source, not the original column.

```dax
Total Net Amount (SUMX) =
SUMX(
    Sales_AllYears,
    Sales_AllYears[Amount] -
        (Sales_AllYears[ReturnedQuantity] * Sales_AllYears[UnitPrice])
)
```

## RANKX and a Missing Dimension Row

Built `Product Sales Rank` with `RANKX` and found a **blank product**
ranked #2 by revenue.

**Root cause:** `ProductMaster` was missing 6 product codes (P101–P106)
that existed in the 2024 sales data.

**Fix:**
1. Created `ProductMaster_Additions` (Enter Data, 6 new rows)
2. Appended it to the original `ProductMaster` → renamed the result
   `productMaster_Full`
3. Along the way, resolved: join key trim/type mismatches, a stale merge
   dialog cache, an accidentally deleted merge step, and a column-name
   collision on `ProductName` (fixed with a coalesce pattern written
   directly in the Advanced Editor)

**Also discovered:** `SalesRep` is null for all 2024 rows because the
2024 source file never had that column — a genuine schema gap, documented
as a known limitation rather than "fixed."

```dax
Product Sales Rank =
RANKX(
    ALL(productMaster_Full[ProductName]),
    [Total Net Amount (SUMX)]
)
```

## Top N — Two Approaches Compared

1. **Visual-level filter** on the rank measure — basic filtering (checkbox
   list) worked more reliably than the advanced filtering dropdown, which
   had UI issues.
2. **Built-in Top N filter type** — no measure required, but not reusable
   across other visuals.

## Key Takeaways
- Merge/join keys must match exactly on both **trim** and **type**.
- Power Query Editor previews can go stale mid-session after renaming
  queries — editing M code directly in the Advanced Editor is a reliable
  fallback when UI dialogs misbehave.
- A blank or duplicated value surfacing in a `RANKX` or `SUMX` result is
  usually a signal to investigate upstream data quality, not a DAX bug.
