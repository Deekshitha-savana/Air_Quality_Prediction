# Power BI Dashboard Guide — Air Quality Index (AQI) Prediction Project

This guide walks you through building a professional Power BI dashboard on top of the
`AQI_Predictions.csv` file exported by the Colab notebook.

---

## 1. Import the Data

1. Open Power BI Desktop → **Home → Get Data → Text/CSV**
2. Select `AQI_Predictions.csv`
3. Click **Transform Data** (Power Query Editor) to verify column types:
   - `PM2.5, PM10, NO, NO2, NOx, NH3, CO, SO2, O3` → Decimal Number
   - `Actual_AQI, Predicted_AQI` → Decimal Number
   - `AQI_Category` → Text
   - `Model_Used` → Text
4. Click **Close & Apply**

## 2. Create Helper Columns / Measures (DAX)

In the **Data** view or Power Query, add these calculated columns/measures:

```DAX
Prediction Error = ABS('AQI_Predictions'[Actual_AQI] - 'AQI_Predictions'[Predicted_AQI])

Avg Actual AQI = AVERAGE('AQI_Predictions'[Actual_AQI])

Avg Predicted AQI = AVERAGE('AQI_Predictions'[Predicted_AQI])

Model Accuracy % =
1 - (AVERAGE('AQI_Predictions'[Prediction Error]) / AVERAGE('AQI_Predictions'[Actual_AQI]))
```

Also create a **color mapping table** (New Table) for consistent AQI category colors:

```DAX
AQI_Colors =
DATATABLE(
    "AQI_Category", STRING,
    "ColorHex", STRING,
    {
        {"Good", "#2ECC71"},
        {"Satisfactory", "#A9DFBF"},
        {"Moderate", "#F4D03F"},
        {"Poor", "#E67E22"},
        {"Very Poor", "#E74C3C"},
        {"Severe", "#7B241C"}
    }
)
```
Relate this table to `AQI_Category` and use it to drive conditional formatting across all pages.

---

## 3. Dashboard Pages

### Page 1 — AQI Overview
| Visual | Fields |
|---|---|
| Card | Avg Actual AQI |
| Card | Max(Actual_AQI) |
| Card | Min(Actual_AQI) |
| Card | Count of rows (Total Records) |
| Donut Chart | AQI_Category (legend), Count of records (values) |
| Gauge | Avg Actual AQI, with target/min/max set to category thresholds (0–500) |

### Page 2 — Actual vs Predicted AQI
| Visual | Fields |
|---|---|
| Line/Scatter Chart | X = Actual_AQI, Y = Predicted_AQI (add a diagonal reference line via Analytics pane) |
| Line Chart | Index/row vs Actual_AQI & Predicted_AQI (two series) to compare trends |
| Table | Top 10 rows sorted by `Prediction Error` descending |
| Card | Model Accuracy % |

### Page 3 — Pollutant Comparison
| Visual | Fields |
|---|---|
| Clustered Column Chart | Average of each pollutant (PM2.5, PM10, NO, NO2, NOx, NH3, CO, SO2, O3) |
| Matrix | AQI_Category (rows) × average pollutant values (columns) |
| Scatter Chart | X = PM2.5, Y = Predicted_AQI, size = PM10 (explore relationships) |

### Page 4 — AQI Category Distribution
| Visual | Fields |
|---|---|
| Stacked Column Chart | AQI_Category (axis), Count of records (values) — colored via `AQI_Colors` table |
| Slicer | AQI_Category (lets users filter every other visual/page via sync slicers) |
| Slicer | Model_Used |

### Page 5 — Feature Importance
| Visual | Fields |
|---|---|
| Horizontal Bar Chart | Feature (axis), Importance (values) — import `feature_importance.csv` if you export it separately from the notebook's `feat_imp_df` |
| Card/Text box | Short explanation of which pollutants matter most (e.g., PM2.5/PM10 usually dominate) |

*(Tip: If you want Feature Importance in Power BI, add one more cell in the notebook: `feat_imp_df.to_csv('feature_importance.csv', index=False)` and import that CSV as a second table.)*

---

## 4. Polish & Interactivity

- **Theme**: Apply a custom theme (View → Themes) with AQI-category colors as the accent palette.
- **Navigation**: Add buttons/bookmarks on each page to jump between the 5 pages.
- **Sync Slicers**: Sync the `AQI_Category` and `Model_Used` slicers across all pages (Slicer → Sync Slicers pane).
- **Tooltips**: Enable report page tooltips showing exact pollutant values on hover over scatter points.
- **Titles & Subtitles**: Add clear titles (e.g., "AQI Prediction Dashboard — Actual vs Predicted") and a subtitle noting the best model and its R² score.

---

## 5. Suggested Report Flow for Viva/Presentation

1. Start on **AQI Overview** — set the context (what AQI is, how bad/good things are overall)
2. Move to **Actual vs Predicted** — demonstrate the model's accuracy visually
3. Show **Pollutant Comparison** — explain which pollutants are worst and how they vary by category
4. Show **AQI Category Distribution** — highlight how many days/records fall into each severity band
5. End with **Feature Importance** — tie back to the ML model and explain what drives predictions
