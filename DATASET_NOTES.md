# DSA 401 Project 2 — Dataset Notes (instructor copy)

All five datasets are from the UCI Machine Learning Repository (CC BY 4.0). Students should cite the dataset using the citation on its UCI page.

| File | UCI ID | Rows | Features | Target |
|---|---|---|---|---|
| `superconductivity.csv` | 464 | 21,263 | 81 numeric | `critical_temp` (K) |
| `seoul_bike.csv` | 560 | 8,465 | 12 (8 numeric, 4 categorical incl. `hour`) | `rented_bike_count` |
| `steel_energy.csv` | 851 | 35,040 | 6 (3 numeric, 3 categorical) | `usage_kwh` |
| `wine_quality.csv` | 186 | 6,497 | 12 (11 numeric + `color`) | `quality` (3–9) |
| `abalone.csv` | 1 | 4,175 | 8 (7 numeric + `sex`) | `rings` (age ≈ rings + 1.5 yrs) |

Every file has the target in the first column, snake_case names, and no missing values.

## Cleaning applied

**Superconductivity:** used `train.csv` only (`unique_m.csv`, the element formulas, is not needed). There are 66 duplicate rows, left in because the effect is negligible.

**Seoul Bike:**
- The original file is Korean-encoded (cp949), which breaks a plain `read_csv`, so the columns were renamed.
- Removed the 295 "non-functioning day" rows, where the count is always 0 (the system was closed).
- Replaced `date` with `day_of_week`.
- `hour` is stored as a number. Treating it as categorical instead is a good modeling choice for students to discuss.

**Steel Energy:**
- Dropped `date`.
- Dropped `CO2(tCO2)`, which is calculated from energy use (r = 0.99).
- Dropped both power-factor columns. Lagging power factor plus lagging reactive power let you compute usage exactly: usage = PF·Q / √(1 − PF²) matches the target to within about 0.0005 kWh. Leaving them in gives a tree R² of 0.999.
- `seconds_from_midnight` is the time of day.
- A stricter version also drops the two reactive-power columns, leaving only time, day, and load type. That version gives Linear 0.42, Tree 0.70, RF 0.70, HistGB 0.71.

**Wine Quality:** combined the red and white files and added a `color` column. There are 1,179 exact duplicate rows in the original data. They were left in, but they inflate cross-validation scores slightly, which is worth a sentence on the poster.

**Abalone:** added column names and removed 2 rows with height = 0. Measurements were converted back to real units (mm and g); UCI stores them divided by 200.

## Baseline results (5-fold CV R², default settings, no tuning)

| Dataset | Linear | Tree | Random Forest | HistGB | Story |
|---|---|---|---|---|---|
| Superconductivity | 0.735 | 0.877 | 0.925 | 0.909 | Large gain from nonlinear models |
| Seoul Bike | 0.545 | 0.846 | 0.921 | 0.933 | Largest gain; demand is very nonlinear in hour and temperature |
| Steel Energy | 0.872 | 0.876 | 0.930 | 0.938 | Linear is solid; ensembles add a clear but modest gain |
| Wine Quality | 0.292 | 0.088 | 0.517 | 0.456 | Noisy, subjective target; single tree overfits badly |
| Abalone | 0.528 | 0.073 | 0.541 | 0.541 | Everything ties except the single tree |

Default single trees reach training R² = 1.0 on every dataset, so pruning or depth limits should be part of the assignment. Random forest beats untuned HistGB on two datasets, so "boosting always wins" will not hold.

## Runtime notes (2-CPU machine, about what Colab gives)

- Five-fold CV takes a few seconds for every model on every dataset, with one exception.
- The exception is random forest on Superconductivity: about 190 seconds with default settings. With `max_features="sqrt"` it takes about 27 seconds at the same R² (0.926). Recommend that setting, or tell students to keep tuning grids small.
- HistGB's automatic early stopping only turns on above 10,000 rows (Superconductivity and Steel).
