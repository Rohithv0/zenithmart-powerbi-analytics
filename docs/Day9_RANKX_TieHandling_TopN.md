# Day 9 — RANKX Tie Handling & Reusable Top-N via TOPN/CALCULATE

## RANKX Tie Handling: Skip vs. Dense

`RANKX` has a 4th argument, **Ties**, controlling how tied values are
ranked:

- **Skip** (default): a tie at rank 3 means the next distinct value jumps
  to rank 5 — matches SQL's `RANK()`
- **Dense**: a tie at rank 3 means the next distinct value gets rank 4, no
  gap — matches SQL's `DENSE_RANK()`

```dax
Product Sales Rank (Dense) =
RANKX(
    ALL(productMaster_Full[ProductName]),
    [Total Net Amount (SUMX)],
    ,
    DESC,
    Dense
)
```

Built alongside the original `Product Sales Rank` measure to compare both
ranking styles side by side in a table visual.

## Reusable Top-N via TOPN + CALCULATE

Both Day 8 Top-N approaches (visual filter, built-in Top N) shared a
limitation: neither produces a standalone, reusable measure. The
`TOPN` + `CALCULATE` pattern solves this — it's just a measure, so it can
be dropped into any card or combined with other calculations.

```dax
Top 3 Product Sales =
VAR TopProducts =
    TOPN(
        3,
        ALL(productMaster_Full[ProductName]),
        [Total Net Amount (SUMX)],
        DESC
    )
VAR Result =
    CALCULATE(
        [Total Net Amount (SUMX)],
        TopProducts
    )
RETURN
    Result
```

`TOPN` returns a **table** (the top N rows by the given measure), which
`CALCULATE` then uses as a filter argument — turning "top N rows" into
"an aggregate over just those rows."

**Tested and confirmed:** changing `3` to `5` correctly increased the
card's returned value, confirming the measure is dynamic and filter-driven
rather than hardcoded.

## Key Takeaways
- Skip ranking = SQL `RANK()`; Dense ranking = SQL `DENSE_RANK()`.
- `TOPN` returns a table, not a scalar — it's almost always paired with
  `CALCULATE` to turn "top N rows" into a usable aggregate.
- The `VAR`/`TOPN` pattern is preferable to UI-based Top N filters
  whenever the result needs to be reused across multiple visuals or
  combined with other measures (e.g., "Top 3 vs. Overall" comparisons on
  the same dashboard).
