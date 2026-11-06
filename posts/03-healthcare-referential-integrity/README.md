# Post 03: Why Does Power BI Show (Blank)? Understanding Referential Integrity

[Back to the project index](../../README.md)

## Project Overview

Unexpected `(Blank)` categories can appear when a fact table contains keys that have no matching records in a related dimension. This synthetic healthcare example compares the same 24 visits before and after correcting missing department records. It demonstrates one common cause of `(Blank)` values, not every possible cause.

## What You'll Learn

| Technique | Purpose |
| --- | --- |
| Referential integrity checks | Identify fact table keys that are absent from a related dimension. |
| `RELATED()` | Inspect the matching dimension key for each visit and find unmatched rows. |
| `EXCEPT()` | List Department IDs present in `Visits` but missing from `Departments`. |
| Source data correction | Resolve the missing mappings while preserving the total visit count. |

## Dataset and Model

The example uses synthetic healthcare data with **24 visits**.

| Table | Role |
| --- | --- |
| `Visits` | Fact table containing visit records and their Department IDs. |
| `Departments` | Dimension table mapping Department IDs to department names. |

The relationship connects `Visits[DepartmentID]` on the many side to `Departments[DepartmentID]` on the one side. In the Before version, `D05` and `D06` occur in `Visits` but are absent from `Departments`, leaving **10 unmatched visits**.

## Project Files

| File | Description |
| --- | --- |
| [healthcare-referential-integrity-before.pbix](powerbi/healthcare-referential-integrity-before.pbix) | Supplied Before project illustrating missing department keys and the resulting `(Blank)` category. |
| [healthcare-referential-integrity-after.pbix](powerbi/healthcare-referential-integrity-after.pbix) | Supplied After project with the missing department records added. |

Open a file link and select **Download raw file** to save it locally. Alternatively, download the repository as a ZIP from the repository's **Code > Download ZIP** menu and extract it. Both PBIX files are supplied unchanged apart from their filenames. No separate dataset is included.

## How to Use This Project

1. Download both PBIX files using the links above.
2. Open `healthcare-referential-integrity-before.pbix` in Power BI Desktop. Review the department breakdown and the 10 visits grouped under `(Blank)`.
3. Inspect the `Visits` and `Departments` tables and their relationship. Compare the `Matching Department` calculated column and the `Missing Departments` calculated table with the DAX below.
4. Open `healthcare-referential-integrity-after.pbix` in Power BI Desktop. Compare the department breakdown and the diagnostic results with the Before version.
5. Confirm that Dermatology has 6 visits, Radiology has 4 visits, no visits remain unmatched, and the total remains 24.

## Understanding the Referential Integrity Issue

Referential integrity requires each foreign key in the fact table to have a matching key in the related dimension. In the Before version, visits reference `D05` and `D06`, but the dimension has no records for those IDs.

When a visual groups visits by department, Power BI associates those unmatched fact rows with a blank dimension member. The 10 visits therefore appear under `(Blank)` rather than their department names. They remain part of the total; the missing mappings prevent them from being assigned to named departments.

Missing dimension keys are only one possible explanation for `(Blank)`. Blank source values and DAX expressions that return `BLANK()` can also produce blank results. Inspect the data and model before choosing a fix.

## Diagnosing with RELATED()

The calculated column in `Visits` looks up the Department ID through the relationship:

```dax
Matching Department =
RELATED(Departments[DepartmentID])
```

For visits with a matching department record, the column returns that record's Department ID. For unmatched visits, it returns `BLANK()`. Filtering this column to blank in the Before version identifies the 10 visit rows whose Department IDs are `D05` or `D06`.

This is a row-level diagnostic: it shows which fact rows lack a match, rather than listing each missing key only once.

## Diagnosing with EXCEPT()

The calculated table compares the distinct Department IDs in the two tables:

```dax
Missing Departments =
ADDCOLUMNS(
    EXCEPT(
        VALUES(Visits[DepartmentID]),
        VALUES(Departments[DepartmentID])
    ),
    "Missing From", "Departments"
)
```

`EXCEPT()` returns values from its first input that do not occur in its second input. The input order matters: this expression finds keys used by visits that are missing from the department dimension. `ADDCOLUMNS()` labels the missing side.

In the Before version, the result is:

| DepartmentID | Missing From |
| --- | --- |
| D05 | Departments |
| D06 | Departments |

After the dimension records are added, the expression returns no missing Department IDs.

## Before vs. After

The example corrects the dimension data by adding `D05: Dermatology` and `D06: Radiology`.

| Check | Before | After |
| --- | --- | --- |
| Missing dimension keys | D05 and D06 | None |
| Visits without a matching department | 10 | 0 |
| Dermatology (D05) | 6 visits included in `(Blank)` | 6 visits under Dermatology |
| Radiology (D06) | 4 visits included in `(Blank)` | 4 visits under Radiology |
| Total visits | 24 | 24 |
| `EXCEPT()` diagnostic | D05 and D06 | No missing Department IDs |

The correction assigns the previously unmatched visits to their departments without adding or removing visits.

## Key Takeaways

- A valid relationship can still have unmatched fact rows when dimension keys are missing.
- `RELATED()` helps locate affected rows; `EXCEPT()` identifies the missing keys.
- DAX helps diagnose the issue. Correct the underlying data at the source or data preparation layer.
- In real projects, determine whether dimension records are missing or fact table keys are incorrect, then correct the appropriate source data using SQL, Power Query, or the upstream ETL process.
- Check both the department breakdown and total visits after a correction. Missing dimension keys are one possible reason for `(Blank)`, not the only one.

## Related LinkedIn Post

LinkedIn post link will be added after publication.
