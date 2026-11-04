# Post 02: Power BI Native Matrix Calendar

[Back to the project index](../../README.md)

## Project Overview

A calendar can make daily supply chain exceptions easier to scan before investigating individual incidents. This project builds that calendar with Power BI's native Matrix visual, a date dimension, DAX display measures, and conditional formatting. No custom calendar visual is required.

The report contains an `Exception Calendar` page and an `Incident Register` drill-through page. A disconnected category selector spotlights relevant days while keeping the full calendar context visible.

## What You'll Learn

| Technique | Purpose |
| --- | --- |
| Native Matrix calendar | Arrange months and weeks into Monday-to-Sunday calendar cells. |
| DAX cell display | Combine a date, incident summaries, overflow count, and daily estimated exposure. |
| Conditional formatting | Color cells by daily severity and dim days outside selected categories. |
| Disconnected category slicer | Highlight matches without filtering incidents out of the calendar. |
| Multi-month selection | Display separate month sections without combining dates from different months. |
| Drill-through | Move from a calendar cell to the underlying incident records. |

## Dataset and Model

The Excel workbook contains **408 synthetic supply chain exception records** for 2026 in the `Exceptions` worksheet and named Excel table. Each row represents one fictional incident. This is learning data, not actual company operations. Estimated value at risk is potential incident exposure in USD, not realized loss.

The 13 source fields are `ExceptionID`, `ExceptionDate`, `ExceptionType`, `Description`, `Warehouse`, `Region`, `Severity`, `EstimatedValueAtRiskUSD`, `Status`, `ResolutionDate`, `SeverityRank`, `CalendarLabel`, and `Location`. `SeverityRank` and `CalendarLabel` are imported source fields.

Seven exception categories cover Shipment Delays, Stockout Risks, Inventory Discrepancies, Near-Expiry Inventory, Warehouse Capacity Alerts, Supplier Quality Issues, and Fulfillment SLA Breaches.

| Model table | Role |
| --- | --- |
| `FactExceptions` | Imports the Excel `Exceptions` table and holds the four calendar measures. |
| `DimDate` | DAX date table covering January 1 through December 31, 2026, with month, weekday, and week-of-month fields. |
| `ExceptionCategorySelector` | Independent category list with `ExceptionType` and `CategoryOrder`; intentionally has no relationships. |

The active relationship connects `DimDate[Date]` on the one side to `FactExceptions[ExceptionDate]` on the many side, with date filters flowing to the incidents. The selector remains disconnected.

## Project Files

| File | Description |
| --- | --- |
| [supply-chain-calendar.pbix](powerbi/supply-chain-calendar.pbix) | Supplied Power BI report with its embedded model, native Matrix calendar, slicers, and drill-through page. |
| [supply-chain-exceptions-2026.xlsx](data/supply-chain-exceptions-2026.xlsx) | Supplied 408-record synthetic dataset. |

Open a file link and select **Download raw file** to save it locally. Alternatively, download the repository as a ZIP from the repository's **Code > Download ZIP** menu and extract it.

## How to Use This Project

1. Download both files using the links above.
2. Open `supply-chain-calendar.pbix` in Power BI Desktop. The embedded imported data allows initial exploration without refreshing.
3. If refresh is required, open **Transform data**, then **Manage Parameters** (or **Edit parameters**, depending on the Desktop version). Set the text parameter `DataSourcePath` to the full local path of your downloaded `supply-chain-exceptions-2026.xlsx`, for example `C:/PowerBI/data/supply-chain-exceptions-2026.xlsx`.
4. Apply the parameter change, choose **Close & Apply** if in Power Query Editor, and refresh. The import reads `Excel.Workbook(File.Contents(DataSourcePath), null, true)` and selects the named table `Exceptions`. Preserve that table name and its columns. Renaming the distributed workbook does not update the absolute path stored inside the PBIX.
5. Open `Exception Calendar`. Use `Month-Year` to select one or more months; use Ctrl+click if required by your Desktop slicer settings. The saved report selects February and March 2026. Scroll the Matrix to review additional month sections.
6. Choose one or more entries in `Spotlight categories`. Matching days retain their severity colors; other days dim. Clear the category selection to restore the normal calendar view. The selector does not remove dates or incidents, and the daily exposure still includes every incident on that date.
7. Right-click a calendar value cell for a real date, then choose **Drill through > Incident Register**. The register displays incident-level date, ID, category, severity, description, warehouse, estimated exposure, status, and resolution date. Use **Back to calendar** to return; in Desktop authoring mode, Ctrl+click activates the button.

The PBIX and workbook are supplied unchanged apart from their filenames. The stored source path belongs to the original environment and must be updated locally before refresh.

## Native Matrix Setup

| Field well | Fields, in order |
| --- | --- |
| Rows | `DimDate[Month-Year]`, then `DimDate[Week of Month]` |
| Columns | `DimDate[Day Name]` |
| Values | `FactExceptions[Calendar Cell Display]` |

Expand the row hierarchy through `Week of Month` while retaining `Month-Year`. Enable **Show items with no data** for the date row and column fields. Sort `Day Name` by `Day of Week Number` (Monday = 1), and `Month-Year` by `Month-Year Sort`. The first partial week belongs to week 1.

Keeping the month in the row context is essential: month, week of month, and weekday together identify at most one date. Omitting the month can combine dates across selected months and cause `SELECTEDVALUE(DimDate[Date])` to return blank. Turn off row and column subtotals and enable value word wrap so each cell reads as a compact daily summary.

## DAX Measures and Conditional Formatting

The model stores four measures in `FactExceptions`:

| Measure | Behavior |
| --- | --- |
| `Calendar Cell Display` | Uses `SELECTEDVALUE` to identify the date, `TOPN(2)` and `CONCATENATEX` to show up to two incidents ordered by ascending `SeverityRank`, then `ExceptionID`. Adds severity symbols, line breaks with `UNICHAR(10)`, a `+N more` line for extra incidents, and the sum of estimated exposure for all incidents on that date. Empty days show the date; invalid date intersections return blank. |
| `Category Match` | Uses `ISFILTERED`, `VALUES`, and `INTERSECT` to test whether any incident category on the date matches the disconnected selection. Returns 1 when no category filter is active. |
| `Calendar Background Color` | Uses the minimum daily severity rank for the highest severity: Critical (1), High (2), Medium (3), or Low (4). Returns a muted background for nonmatching days, white for empty days, and the canvas color for invalid date intersections. |
| `Calendar Font Color` | Returns muted text for nonmatching days and the normal text color otherwise. |

For `Calendar Cell Display`, configure **Conditional formatting > Background color**, choose **Field value**, and select `Calendar Background Color`. Repeat for **Font color**, selecting `Calendar Font Color`. These are color-returning measures, not numeric color scales. Menu locations can vary by Desktop version.

## Disconnected Selector Setup

The supplied selector is a Power Query table, not a DAX calculated table. Its source is:

```powerquery
#table(
    type table [ExceptionType = text, CategoryOrder = Int64.Type],
    {
        {"Shipment Delays", 1},
        {"Stockout Risks", 2},
        {"Inventory Discrepancies", 3},
        {"Near-Expiry Inventory", 4},
        {"Warehouse Capacity Alerts", 5},
        {"Supplier Quality Issues", 6},
        {"Fulfillment SLA Breaches", 7}
    }
)
```

To reproduce it, create a blank query with this source, name it `ExceptionCategorySelector`, and load it. Sort `ExceptionType` by `CategoryOrder`. Do not create a relationship to either model table. Add `ExceptionCategorySelector[ExceptionType]` to the category slicer and allow multiple selections.

The measure reads the independent selection explicitly:

```dax
Category Match =
VAR SelectionActive =
    ISFILTERED ( ExceptionCategorySelector[ExceptionType] )
VAR MatchingTypes =
    INTERSECT (
        VALUES ( FactExceptions[ExceptionType] ),
        VALUES ( ExceptionCategorySelector[ExceptionType] )
    )
RETURN
    IF ( NOT SelectionActive, 1, INT ( COUNTROWS ( MatchingTypes ) > 0 ) )
```

Use this result in the color measures, rather than as a visual filter. Relating the selector to the fact table, or filtering the Matrix to `Category Match = 1`, would change the intended spotlight behavior. A match checks all incidents on the date, including those beyond the two displayed summaries. Daily severity and exposure remain based on the full day.

## Drill-through Setup

`Incident Register` is configured as a drill-through destination using `DimDate[Month-Year]`, `DimDate[Week of Month]`, and `DimDate[Day Name]`. These are the calendar's grouping fields; together they identify the clicked date. Keep all three in the destination's drill-through setup.

The register page is hidden in reading view and contains a Back action button. A category spotlight does not restrict its incident rows: the selector is disconnected, so the register retains the clicked day's full incident detail.

## Key Takeaways

- A native Matrix can serve as a calendar when date context and sorting are designed carefully.
- Month context keeps multi-month selections distinct.
- DAX can combine compact incident text with daily totals while drill-through exposes the full records.
- A disconnected selector can guide attention through formatting while preserving data context.
- Conditional colors reflect the day's highest severity, not only the selected category's severity.

## Validation Notes

Both supplied archives passed integrity checks. The workbook contains 408 data rows, 13 fields, and an `Exceptions` named table. Report definitions were inspected directly for Matrix fields, conditional formatting references, slicer settings, saved month selection, and drill-through bindings. Model documentation was cross-checked against the existing local model extraction associated with a byte-identical copy of the supplied PBIX.

SHA-256 checks confirmed that the renamed PBIX and XLSX match their supplied originals byte for byte. Relative Markdown file links were checked, and existing repository files other than the main README index were preserved.

This publication validation did not open or refresh the project in Power BI Desktop or run interactive slicer and drill-through tests. Archive and metadata inspection do not establish refresh success or compatibility with every Desktop version.

## Related LinkedIn Post

LinkedIn post link will be added after publication.
