# Hospital Operations: Claim Cost & Encounter Analysis

An exploratory and inferential analysis of hospital encounter data, built to answer one question:

> **How do total claim costs vary across encounter classes, and is encounter duration associated with total claim cost?**

The project covers data inspection and quality auditing, exploratory data analysis (six visualisations), two rank-based hypothesis tests, and a set of limitations and recommendations for hospital operations and finance teams. It was produced for Week 3 of the Analystlab Africa Data Science internship programme (Track B).

---

## Contents

- [Key results](#key-results)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Figures](#figures)
- [Data-quality decisions](#data-quality-decisions)
- [Limitations](#limitations)
- [Recommendations](#recommendations)
- [Repository structure](#repository-structure)
- [Reproducing the analysis](#reproducing-the-analysis)
- [Privacy](#privacy)

---

## Key results

| Finding | Evidence |
|---|---|
| Claim costs are extremely right-skewed | Median $279 vs mean $3,640; skewness 16.3; the top 10% of encounters account for 66.6% of all cost |
| Typical cost differs sharply by encounter class | Median total claim cost ranges from $143 (urgent care) to $2,356 (inpatient) |
| Cost distributions differ across classes (H1) | Kruskal–Wallis H = 2025.19, p < 0.001 |
| Longer encounters tend to cost more (H2) | Spearman ρ = 0.338, p < 0.001 (weak-to-moderate positive association) |
| Zero recorded payer coverage is ambiguous | 49% of encounters show zero coverage, including 4,779 with a named insurer |

**Median total claim cost by encounter class**

| Encounter class | Encounters | Share of encounters | Median cost | Mean cost | Share of total cost |
|---|---:|---:|---:|---:|---:|
| Ambulatory | 12,537 | 45.0% | $574 | $2,894 | 35.7% |
| Outpatient | 6,300 | 22.6% | $279 | $2,237 | 13.9% |
| Urgent care | 3,666 | 13.1% | $143 | $6,369 | 23.0% |
| Emergency | 2,322 | 8.3% | $146 | $4,630 | 10.6% |
| Wellness | 1,931 | 6.9% | $1,231 | $4,261 | 8.1% |
| Inpatient | 1,135 | 4.1% | $2,356 | $7,761 | 8.7% |

Urgent care illustrates why medians matter here: it has the lowest median cost but the second-highest mean and a disproportionate share of total cost, driven by a small number of very expensive encounters.

> **Important:** these are statistical associations in observational, historical data from a single organisation. They do not show that encounter class or duration *causes* higher costs.

---

## Dataset

The data follows the [Synthea](https://synthetichealth.github.io/synthea/) synthetic-patient format and describes a single organisation (Massachusetts General Hospital) between **2 January 2011 and 5 February 2022**. Costs are assumed to be US dollars (the data dictionary does not state a currency).

| Table | Rows | Columns | Used for |
|---|---:|---:|---|
| `encounters` | 27,891 | 14 | Primary analysis table |
| `procedures` | 47,701 | 9 | Cost-reconciliation check and context |
| `payers` | 10 | 7 | Payer names |
| `organizations` | 1 | 2 | Confirming a single organisation |
| `patients` | 974 | 20 | Key check only (no personal data loaded) |
| `data_dictionary` | 65 | 3 | Field definitions |

**The raw data files are not included in this repository** (see [Privacy](#privacy)). Place your own copies in `data/` to reproduce the analysis.

---

## Methodology

1. **Inspect.** Compare the data dictionary with the files, check foreign-key integrity, look for duplicates and missing values, and confirm data types.
2. **Prepare.** Parse `START` and `STOP` as UTC datetimes, derive encounter duration (hours) and year, and add flags for zero claim cost, extreme duration, zero coverage and uninsured status. Costs are log-transformed *for charts only*.
3. **Explore.** Summarise by encounter class using medians and quartiles alongside means, then produce six visualisations, each with a "shows / cannot show" interpretation.
4. **Test.** Two hypotheses, both using rank-based methods because costs are strongly skewed:
   - **H1:** Kruskal–Wallis test of total claim cost across encounter classes.
   - **H2:** Spearman rank correlation between encounter duration and total claim cost (overall and by class).
5. **Validate.** Confirm the record count (27,891), unique encounter IDs, and no missing values in the analysis fields.

**Design principle:** no rows were deleted and no missing values were imputed. Unusual values are flagged and their influence examined rather than removed.

A fixed random seed (`SEED = 42`) is used so every random step (for example, the 8,000-point scatter sample) is repeatable.

---

## Figures

### 1. Encounters by class
![Encounters by class](outputs/figures/01_encounters_by_class.png)

Ambulatory visits make up about 45% of records; inpatient stays are the smallest group. The groups are very unequal in size.

### 2. Encounters by year
![Encounters by year](outputs/figures/02_encounters_by_year.png)

Volume peaks in 2014 and 2021. The first and last years (2011 and 2022) are partial and should not be compared with full years.

### 3. Distribution of total claim cost
![Cost distribution](outputs/figures/03_cost_distribution.png)

On the original scale almost every record sits near zero, with a very long tail. The log scale reveals several distinct peaks, suggesting different types of encounter with very different typical costs.

### 4. Cost by encounter class
![Cost by class](outputs/figures/04_cost_by_class_boxplot.png)

Classes occupy markedly different cost levels, with some showing wide within-class spread.

### 5. Duration against cost
![Duration vs cost](outputs/figures/05_duration_vs_cost.png)

The vertical stripes are classes with a fixed duration. Where duration varies (ambulatory, outpatient), cost tends to rise with it.

### 6. Duration–cost correlation by class
![Correlation by class](outputs/figures/06_duration_cost_correlation_by_class.png)

Spearman correlation between duration and cost, calculated separately for each class that has variable durations.

### 7. Zero recorded payer coverage
![Zero coverage](outputs/figures/07_zero_coverage.png)

Zero coverage varies by class but far more by payer. It must not be read as "uninsured".

---

## Data-quality decisions

The notebook logs every check, finding and action. The complete table is in [`reports/Data_Quality_Summary.md`](reports/Data_Quality_Summary.md). The decisions with the most influence on interpretation are:

- **Cost fields do not reconcile.** Total claim cost equals base encounter cost plus procedure costs for only 30% of encounters, so total claim cost is analysed as recorded and is not decomposed.
- **89 encounters have a zero total claim cost** despite a positive base cost. They are kept and excluded only from log-scale charts.
- **Duration has limited information.** Several classes have a single fixed duration, every inpatient stay lasts at least 24 hours, and four encounters exceed one year (maximum 44,930 hours, about 5.1 years). Both ">24 hours" (75 encounters) and "≥24 hours" (1,152 encounters) definitions are reported.
- **Zero payer coverage is ambiguous.** It is reported as *recorded coverage* only.
- **Two data-dictionary mismatches** (for example `fipscountycode`, `middle`) do not affect any field used.
- **27 encounters share patient, start time and class** with another record. They are retained as possibly separate services within one visit.

---

## Limitations

- **Repeated encounters.** Patients have a median of 14 encounters each (maximum 1,381), so observations are not fully independent, which can affect test results.
- **Skewed costs.** A small number of expensive encounters strongly affect averages.
- **Extreme and fixed durations.** These limit what duration can reveal, particularly within fixed-duration classes.
- **Confounding.** Procedures, payer and patient characteristics may influence both class and cost; their individual effects are not isolated.
- **Payer coverage ambiguity.** Not-recorded and not-covered cannot be distinguished.
- **Generalisability.** One organisation and one historical period; synthetic data.
- **Association, not causation.**

---

## Recommendations

1. Review encounter classes with higher typical claim costs to understand which services and procedures drive the difference.
2. Report median costs alongside averages so unusually expensive encounters do not distort the picture of typical cost.
3. Investigate high-cost encounters separately, using procedure and patient-level context.
4. Clarify the meaning of zero payer coverage with the data owner before using it to assess insurance status.

These are based on patterns in the supplied data; further information is needed before operational changes are made.

---

## Repository structure

```
.
├── README.md
├── Week3_Hospital_Analysis_Data_Science_Simplified.ipynb   # full analysis
├── Hospital Operations Cost & Encounter Analysis.pdf        # presentation slides
├── data/                                                    # NOT tracked (see Privacy)
│   ├── encounters.csv
│   ├── procedures.csv
│   ├── payers.csv
│   ├── organizations.csv
│   ├── patients.csv
│   └── data_dictionary.csv
├── outputs/
│   ├── figures/                                             # PNG visualisations
│   └── tables/                                              # CSV summary tables (generated)
└── reports/
    └── Data_Quality_Summary.md
```

`outputs/tables/` and `reports/` are created automatically when the notebook runs.

---

## Reproducing the analysis

**Requirements:** Python 3.10+ (developed on 3.13) with `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, and Jupyter.

```bash
# 1. Clone the repository
git clone <your-repository-url>
cd <your-repository-name>

# 2. (Optional) create a virtual environment
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 3. Install dependencies
pip install pandas numpy matplotlib seaborn scipy jupyter

# 4. Add the six CSV files to ./data/

# 5. Run the notebook top to bottom
jupyter notebook Week3_Hospital_Analysis_Data_Science_Simplified.ipynb
```

Run all cells in order (restart the kernel first). The notebook writes figures to `outputs/figures/`, summary tables to `outputs/tables/`, and the data-quality report to `reports/`.

---

## Privacy

`patients.csv` contains names and addresses. Although the dataset is synthetic, the analysis is written as if it were real: only the `Id` column is loaded, and no personal details appear in any output. Raw data should not be committed. Suggested `.gitignore` entries:

```gitignore
data/
*.csv
.ipynb_checkpoints/
.venv/
```

---

## Author

<!-- Add your name, programme and contact/LinkedIn link here -->

## Licence

<!-- Add a licence (for example, MIT) or remove this section -->
