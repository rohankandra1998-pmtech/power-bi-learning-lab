# Post 05: Active vs. Inactive Relationships in Power BI | USERELATIONSHIP()

[Back to the project index](../../README.md)

## Project Overview

A retail sale recorded in August may be refunded in October. Each row in `Sales` represents one transaction line with a `SaleDate` and an optional `RefundDate`. This project uses the same `Calendar` table to report units sold by sale date and units refunded by refund date.

## What You'll Learn

| Technique | Purpose |
| --- | --- |
| Active relationship | See how a month filter on `Calendar` reaches `Sales` through `SaleDate` by default. |
| Refund filter | See why filtering to rows with a refund date still groups them by sale month when the default relationship applies. |
| Inactive relationship | Set up a second date path between the same tables. |
| `USERELATIONSHIP()` in `CALCULATE()` | Use the refund-date relationship for one measure. |
| Month-by-month comparison | Trace a refund across months and compare the two measures with the full date range selected. |

## Dataset and Semantic Model

| Table | Role |
| --- | --- |
| `Sales` | 210 retail transaction lines with sale and refund dates, product details, and units. A populated `RefundDate` means the full quantity on that line was refunded. |
| `Calendar` | Date dimension covering 2026, with `YearMonth` for grouping transactions by month. |

| Filter path | State in Model view | Cardinality and direction |
| --- | --- | --- |
| `Calendar[Date]` → `Sales[SaleDate]` | Active (solid line) | One-to-many, single direction from Calendar to Sales |
| `Calendar[Date]` → `Sales[RefundDate]` | Inactive (dashed line) | One-to-many, single direction from Calendar to Sales |

To recreate the second relationship in another model, select **Manage relationships > New**, choose `Calendar[Date]` and `Sales[RefundDate]`, set the cardinality to one-to-many with Calendar on the one side, set cross-filter direction to **Single**, and clear **Make this relationship active**.

## Project Files

| File | Description |
| --- | --- |
| [userelationship-demo.pbix](powerbi/userelationship-demo.pbix) | Supplied Power BI report with its saved semantic model and embedded data. |

## How to Use This Project

1. Open the PBIX link above on GitHub and choose **Download raw file**.
2. Open the downloaded PBIX in Power BI Desktop.
3. Inspect `Sales` and `Calendar` in Table view. On **Understanding the Data Model**, find the sample transaction lines.
4. Open Model view and trace both relationships from `Calendar[Date]` to `Sales`.
5. On **The Problem**, compare `Units Refunded (Incorrect)` and `Units Refunded` in the monthly matrix. Use the sale and refund dates for `T1001` to trace where each calculation counts its two units.
6. On **The Solution**, use the month slicer to see how the cards respond to the date selection. Clear the month and product category slicers to compare the full-period totals.

## DAX Measures

### Units Sold

```dax
Units Sold =
SUM('Sales'[Units])
```

A Calendar month filters this measure through the active `SaleDate` relationship.

### Units Refunded (Incorrect)

```dax
Units Refunded (Incorrect) =
CALCULATE(
    [Units Sold],
    NOT ISBLANK('Sales'[RefundDate])
)
```

The blank check keeps refunded transaction lines. Calendar still filters through `SaleDate`, so the refunded units appear in the month of the original sale.

### Units Refunded (Correct)

```dax
Units Refunded =
CALCULATE(
    [Units Sold],
    USERELATIONSHIP(
        'Calendar'[Date],
        'Sales'[RefundDate]
    ),
    NOT ISBLANK('Sales'[RefundDate])
)
```

For this evaluation, `USERELATIONSHIP()` uses the existing `RefundDate` relationship to carry the Calendar filter. The model keeps `SaleDate` as its default filter path. [Microsoft Learn documentation](https://learn.microsoft.com/en-us/dax/userelationship-function-dax)

## Before vs. After

| TransactionID | SaleDate | RefundDate | Units |
| --- | --- | --- | --- |
| `T1001` | August 11, 2026 | October 27, 2026 | 2 |

Both units count toward August sales. The incorrect refund measure also counts them in August, while the correct measure counts them in October. Across the full saved dataset, summing the refunded lines yields **105 units** under either grouping. The monthly distribution changes even when the total is the same.

## Key Takeaways

1. The active relationship provides the default filter path.
2. An inactive relationship provides an alternative date path.
3. `USERELATIONSHIP()` enables that path for a specific calculation.

## Validation and Limitations

- The PBIX passed an archive integrity check and matches the supplied file by SHA-256. The saved report definitions, measure expressions, relationship metadata, and transaction `T1001` were inspected directly. The refund total was calculated from the embedded rows.
- The PBIX contains both the saved import and the CSV text used by Power Query. With the supplied blank `SalesCsvPath` parameter, refresh uses that bundled data. Setting the parameter to a file path makes refresh depend on that CSV.
- Power BI Desktop rendering, DAX execution, and refresh were outside the archive validation.

## Related LinkedIn Post

LinkedIn post link will be added after publication.
