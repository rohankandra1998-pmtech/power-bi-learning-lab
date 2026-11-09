# Post 04: Documenting a Semantic Model with INFO.VIEW

[Back to the project index](../../README.md)

## Project Overview

A report can show a no-show rate without explaining which appointments count toward it. This project uses a synthetic outpatient appointment model to show how a searchable data dictionary makes those definitions, calculations, and model relationships easier to find inside Power BI.

## What You'll Learn

| Technique | Purpose |
| --- | --- |
| `INFO.VIEW.MEASURES()` | Inspect measure names, home tables, descriptions, and DAX expressions in `Model Measures`. |
| `INFO.VIEW.COLUMNS()` | Inspect column names, data types, and other properties in `Model Columns`. |
| `INFO.VIEW.TABLES()` | Inspect table names, descriptions, and storage modes in `Model Tables`. |
| `INFO.VIEW.RELATIONSHIPS()` | Inspect relationship endpoints, cardinality, active state, and cross-filter behavior in `Model Relationships`. |
| Model descriptions | Record business definitions alongside the technical metadata. |
| `FILTER()`, `SELECTCOLUMNS()`, and `UNION()` | Shape metadata from the four object types into one dictionary. |

## Dataset and Model

The saved import contains **5,000 synthetic outpatient appointments** for calendar year 2025.

| Table | Role |
| --- | --- |
| `FactAppointments` | One row per scheduled appointment, with its status and any recorded wait time, service duration, and satisfaction score. |
| `DimDate` | Calendar dates used to group appointments over time. |
| `DimPatients` | Synthetic patient IDs and age bands. |
| `DimProviders` | Synthetic provider IDs, labels, and department assignments. |
| `DimDepartments` | Eight outpatient departments. |
| `Appointment Metrics` | Disconnected home table for the ten measures. |

Four active, single-direction, one-to-many relationships filter `FactAppointments` from the date, patient, provider, and department dimensions. The four metadata helper tables and the consolidated `Data Dictionary` calculated table sit alongside that model.

## Project Files

| File | Description |
| --- | --- |
| [healthcare-data-dictionary.pbix](powerbi/healthcare-data-dictionary.pbix) | Supplied Power BI report with its semantic model and saved imported data. |

Open the file link and choose **Download raw file** to save it locally. The PBIX is supplied unchanged apart from its filename. No separate Excel workbook is provided.

## How to Use This Project

1. Download the PBIX and open it in a recent version of Power BI Desktop for Windows that supports the four `INFO.VIEW` functions. Use the saved import to explore the report without refreshing.
2. Start on **Outpatient Appointment Overview**. Review the appointment KPIs, completed-visit trend, status breakdown, and department performance.
3. Open **Data Dictionary** and select **Measure** in the **Type** slicer. Open the **Name** dropdown, use its search box to find `No-Show Rate`, then select the matching item. The table shows its name, description, location, and expression. Clear the slicers to browse the other entries.
4. In Model view, find **No-Show Rate** under **Appointment Metrics** and read its **Description** property. In Table view, inspect the four helper tables and compare their raw metadata with the consolidated dictionary.

## Giving Measures Business Context

The description for **No-Show Rate** explains that the denominator includes completed and no-show appointments and excludes cancellations. The measure applies that definition in DAX:

```dax
DIVIDE ( [No-Show Appointments], [Completed Appointments] + [No-Show Appointments] )
```

For the full year, the saved data gives 514 no-shows out of 4,610 eligible appointments, or **11.15%** to two decimal places.

## Building the Data Dictionary

The four helper tables show the outputs of the individual functions. The `Data Dictionary` calculated table calls those functions directly and combines their results:

1. `FILTER()` excludes hidden objects and the documentation tables themselves from the column, measure, and table lists.
2. `SELECTCOLUMNS()` projects each result to the same five fields in the same order: `Type`, `Name`, `Description`, `Location`, and `Expression`. Relationship rows use active state and cross-filter behavior as their description.
3. `UNION()` stacks the four results by column position. Blank descriptions reflect missing model descriptions, and blank expressions are expected for imported objects that have no DAX expression.

## Validation and Refresh Limitations

- The published PBIX matches the supplied file byte for byte. The package integrity check passed, and the saved import was read directly from the file.
- The same PBIX was previously reopened in Power BI Desktop with its imported data. Desktop validation confirmed four model relationships and **33 dictionary entries**: 13 columns, 10 measures, 6 tables, and 4 relationships. Searching for and selecting **No-Show Rate** returned one row.
- Refresh requires the original synthetic Excel workbook or a compatible workbook with the same five named tables. The Power Query parameter `SourceWorkbook` points to a local path on the author's computer. Set it to your workbook's full path before refreshing. Refresh will fail if the workbook is unavailable.
- The dictionary reflects model metadata when the calculated table is processed. If a description changes, the dictionary needs to be recalculated before the report page displays the update. Review the expressions included in the dictionary before distributing a model that contains sensitive business logic.

## Related LinkedIn Post

LinkedIn post link will be added after publication.
