# EDA Portfolio — Stack Overflow Developer Survey
**Date:** 2026-06-16
**Status:** Approved

---

## Objective

Profile the Stack Overflow Developer Survey dataset and surface 6 insights across two correlation themes:
- **Theme 1:** AI adoption × Compensation (3 charts)
- **Theme 2:** AI adoption × Tech stack trends (3 charts)

---

## Success Metrics

- [ ] Clear data-quality summary (nulls, dupes, dtypes)
- [ ] 6 visual insights with interpretation (3 per theme)
- [ ] Reproducible Jupyter notebook (run top-to-bottom, no errors)
- [ ] All figures saved as PNG to `img/`
- [ ] README.md containing all insights and figure references

---

## Dataset

- **File:** `dataset/so_dev_survey.csv`
- **Source:** Stack Overflow Developer Survey (2024/2025)
- **Size:** ~2,037 respondents, ~170 columns
- **Key columns used:**
  - Demographics: `Age`, `Country`, `EdLevel`, `Employment`
  - Compensation: `ConvertedCompYearly`
  - AI adoption: `AISelect`, `AISent`, `AIAcc`, `AIModelsHaveWorkedWith`
  - Tech stack: `LanguageHaveWorkedWith`, `LanguageWantToWorkWith`, `DevType`
- **Notable characteristic:** Many columns are semicolon-separated multi-values (e.g., `LanguageHaveWorkedWith`). These require splitting before analysis.

---

## Project Structure

```
EDA/
├── dataset/
│   └── so_dev_survey.csv
├── notebooks/
│   └── eda.ipynb
├── img/
│   └── (6 chart PNGs auto-saved here)
├── docs/
│   └── superpowers/specs/2026-06-16-eda-portfolio-design.md
└── README.md
```

---

## Notebook Structure (`notebooks/eda.ipynb`)

### Section 0 — Setup & Imports
- Libraries: `pandas`, `numpy`, `matplotlib`, `seaborn`
- Global style config: one consistent color palette, font, DPI
- Path constants for `dataset/` and `img/` relative to the notebook

### Section 1 — Data Quality Summary
Satisfies the "clear data-quality summary" success metric.

- **Null heatmap:** `seaborn.heatmap` of `df.isnull()` — visual scan of missingness across all columns
- **Dtype + null count table:** DataFrame printed/displayed showing column name, dtype, null count, null %
- **Duplicate check:** Count of exact-duplicate rows
- **Key stats:** `.describe()` on numeric columns (`ConvertedCompYearly`, `YearsCode`, `WorkExp`)

### Section 2 — AI Adoption Overview (Context)
Brief profiling to establish baseline before the two correlation analyses:
- Distribution of `AISelect` (usage frequency)
- Distribution of `AISent` (sentiment toward AI)
- Top AI models from `AIModelsHaveWorkedWith` (after semicolon-splitting)

---

## Part 1 — AI Adoption × Compensation (3 Charts)

### Chart 1 — Violin + Strip Plot
**File:** `img/01_violin_ai_comp.png`

- X-axis: `AISelect` categories (Never / Monthly / Weekly / Daily)
- Y-axis: `ConvertedCompYearly` (log-scaled, outliers capped at 99th percentile)
- Overlay: `stripplot` with low alpha for individual data points
- **Why:** Shows full salary distribution shape per AI usage level — richer than a boxplot, reveals density + individual outlier clusters

### Chart 2 — Bubble Chart (Country-level)
**File:** `img/02_bubble_country_ai_comp.png`

- X-axis: % of respondents in each country who use AI tools (AISelect != "Never")
- Y-axis: Median `ConvertedCompYearly` for that country
- Bubble size: Respondent count for that country
- Annotate top 5 countries by respondent count
- Filter: Countries with >= 10 respondents
- **Why:** Surfaces the global relationship between AI adoption rate and pay in one view

### Chart 3 — Annotated Diverging Bar
**File:** `img/03_diverging_devtype_ai_premium.png`

- Y-axis: Developer type (`DevType`, top 10 by count)
- X-axis: Salary premium = median comp of AI users minus median comp of non-users (USD)
- Color: Positive premium -> teal, negative -> coral
- Annotate bars with exact dollar values
- **Why:** Immediately answers "which roles benefit most financially from AI adoption"

---

## Part 2 — AI Adoption × Tech Stack (3 Charts)

### Chart 4 — Lollipop Chart (Language AI Adoption Rate)
**File:** `img/04_lollipop_lang_ai_adoption.png`

- Y-axis: Top 15 programming languages (from `LanguageHaveWorkedWith`, semicolon-split)
- X-axis: AI adoption rate (% of that language's users who use AI tools)
- Dashed vertical line at overall average adoption rate
- Sorted descending by adoption rate
- **Why:** Cleaner than a bar chart; shows ranking and deviation from average clearly

### Chart 5 — Annotated Heatmap (Language x AI Model)
**File:** `img/05_heatmap_lang_aimodel.png`

- Rows: Top 10 programming languages
- Columns: Top 5 AI models (from `AIModelsHaveWorkedWith`)
- Values: % of that language's users who have worked with each AI model
- Annotate cells with percentage values
- **Why:** Shows which language communities gravitate toward which AI tools — reveals cross-ecosystem adoption patterns

### Chart 6 — Slope Chart (AI Momentum by Language)
**File:** `img/06_slope_lang_ai_momentum.png`

- Two vertical axes for top 10 languages:
  - Left: % of current users (from `LanguageHaveWorkedWith`) who use AI tools (AISelect != "Never")
  - Right: % of aspiring users (from `LanguageWantToWorkWith`) who use AI tools
- Lines connect the two points per language; colored by slope direction (rising vs. falling)
- **Why:** A rising slope means the future adopters of a language are more AI-forward than its current users — signals which ecosystems are attracting AI-native developers

---

## Visualization Standards

- Library: `seaborn` + `matplotlib` (static PNG)
- DPI: 150
- Save: `fig.savefig('../img/<name>.png', dpi=150, bbox_inches='tight')`
- Color palette: Consistent across all charts (e.g., `sns.color_palette("muted")`)
- Each chart cell followed by a Markdown cell with 2-3 sentence interpretation

---

## README.md Structure

```
# Stack Overflow Developer Survey — EDA Portfolio

## Dataset
## Data Quality Summary
## Theme 1: AI Adoption x Compensation
  - Insight 1 (+ embedded img)
  - Insight 2 (+ embedded img)
  - Insight 3 (+ embedded img)
## Theme 2: AI Adoption x Tech Stack
  - Insight 4 (+ embedded img)
  - Insight 5 (+ embedded img)
  - Insight 6 (+ embedded img)
## Key Takeaways
## How to Reproduce
```

---

## Reproduction Instructions

```bash
pip install pandas numpy matplotlib seaborn jupyter
cd notebooks
jupyter notebook eda.ipynb
# Run All Cells
```

No external API calls. No random seeds needed (no ML models). Fully deterministic output.
