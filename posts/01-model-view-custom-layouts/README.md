# Post 01: Power BI Model View Custom Layouts

[Back to the project index](../../README.md)

## Project Overview

A semantic model with several fact tables and shared dimensions can be difficult to navigate when tables are scattered and relationship lines cross. This lab uses three saved Model View layouts to compare ways of organizing the same Northstar Retail model for inspection, troubleshooting, and handover.

## What You'll Learn

The PBIX contains these exact saved layout names, in addition to the default `All tables` diagram:

| Saved layout | Arrangement | Useful for |
| --- | --- | --- |
| `01 Unstructured` | Tables are scattered across the canvas, making relationship lines harder to follow. | Seeing how an unstructured arrangement affects readability. |
| `02 Bus` | Dimensions run across the top, fact tables are stacked on the left, and `Metrics` sits in the upper-left corner. | Finding shared dimensions and following relationships from each fact table. |
| `03 Double Decker` | Fact tables remain on the left, with dimensions split between the top and bottom. | Using the canvas width more efficiently while inspecting a larger model. |

Compare which arrangement makes it easiest to locate a table, trace a relationship, and explain the model to another developer.

## Dataset and Model

The Excel workbook's `Start here` sheet identifies the source as synthetic data generated with fixed seed 31. Customer, employee, and supplier labels are generic sample identifiers.

The saved PBIX diagram definitions contain 14 tables:

| Role | Tables |
| --- | --- |
| 5 fact tables | `FactSales`, `FactReturns`, `FactInventory`, `FactPurchases`, `FactTargets` |
| 8 dimension tables | `DimDate`, `DimProduct`, `DimCustomer`, `DimStore`, `DimChannel`, `DimEmployee`, `DimSupplier`, `DimPromotion` |
| Measures table | `Metrics` |

The workbook contains the 13 corresponding fact and dimension worksheets and Excel tables, plus the `Start here` guide. `Metrics` belongs to the Power BI model rather than the Excel source.

## Project Files

| File | Description |
| --- | --- |
| [northstar-layout-lab.pbix](powerbi/northstar-layout-lab.pbix) | Existing Power BI project with the embedded model and three saved custom layouts. |
| [northstar-retail.xlsx](data/northstar-retail.xlsx) | Northstar Retail source workbook and dataset guide. |

Open a file link and select **Download raw file** to save it locally. Alternatively, download the repository as a ZIP from the repository's **Code** menu.

## How to Use This Project

1. Download both the PBIX and Excel workbook using the links above.
2. Open `northstar-layout-lab.pbix` in Power BI Desktop.
3. Update the `SourceWorkbook` Power Query parameter to the full path of your downloaded `northstar-retail.xlsx`. In Power BI Desktop, open **Transform data**, then **Manage Parameters**, select `SourceWorkbook`, and set its current value. Apply the change. This parameter supplies the Excel file path for all 13 source queries.
4. Refresh the model if required after updating the source.
5. Open **Model view**.
6. Select `01 Unstructured`, `02 Bus`, and `03 Double Decker` using the layout tabs.
7. Compare table positions and relationship lines. Try locating the same fact table and its shared dimensions in each layout.

The supplied PBIX has been preserved without rebuilding the model or editing its relationships. Any stored local source path may need to be updated on your computer.

## Key Takeaways

- All three layouts show the same underlying semantic model.
- Moving a table on the canvas changes its visual position. It does not change the data, relationships, or calculations.
- Editing an actual relationship from any layout changes the shared semantic model and affects all layouts.
- Thoughtful arrangements make navigation, relationship inspection, troubleshooting, and knowledge transfer easier.
- A visual layout change does not inherently improve query performance.

## Validation Notes

The PBIX and XLSX archives passed integrity checks. The saved layout names, table membership, and table positions were inspected directly in the PBIX diagram definitions. The workbook's worksheets and table definitions were readable.

The embedded model was also decoded for inspection: it contains 22 relationships and 14 measures assigned to `Metrics`. All 13 data tables were readable. The Excel source is referenced through the `SourceWorkbook` parameter, which currently stores an absolute Windows path. No credential or personal-contact patterns were detected in the inspected data and readable metadata. This is a scoped inspection, not a guarantee that every possible sensitive value can be detected.

The project has not been opened and refreshed in Power BI Desktop as part of this validation. Archive inspection does not establish refresh success or compatibility with every Desktop version.

## Related LinkedIn Post

LinkedIn post link will be added after publication.
