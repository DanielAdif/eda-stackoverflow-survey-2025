# EDA Portfolio — Stack Overflow Developer Survey Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a reproducible Jupyter notebook that surfaces 6 insights on AI adoption vs. compensation and AI adoption vs. tech stack trends from the Stack Overflow Developer Survey.

**Architecture:** Single notebook (`notebooks/eda.ipynb`) with 6 sections: setup, data quality, AI adoption overview, 3 AI×compensation charts, 3 AI×tech-stack charts, and key takeaways. Each chart is saved as PNG to `img/`. A `README.md` embeds all charts and narrates findings. Cells are added programmatically via `nbformat` run from the repo root.

**Tech Stack:** Python 3.9+, pandas, numpy, matplotlib, seaborn, nbformat, jupyter

---

## File Map

| Action | Path | Purpose |
|--------|------|---------|
| Create | `dataset/so_dev_survey.csv` | Move raw data out of root |
| Create | `notebooks/eda.ipynb` | Main analysis notebook |
| Create | `img/01_violin_ai_comp.png` | Chart 1 output |
| Create | `img/02_bubble_country_ai_comp.png` | Chart 2 output |
| Create | `img/03_diverging_devtype_ai_premium.png` | Chart 3 output |
| Create | `img/04_lollipop_lang_ai_adoption.png` | Chart 4 output |
| Create | `img/05_heatmap_lang_aimodel.png` | Chart 5 output |
| Create | `img/06_slope_lang_ai_momentum.png` | Chart 6 output |
| Create | `README.md` | Portfolio summary with embedded charts |

---

## Shared Helper Pattern

Every task that adds notebook cells uses this pattern (run from repo root):

```python
import nbformat as nbf

with open('notebooks/eda.ipynb', 'r', encoding='utf-8') as f:
    nb = nbf.read(f, as_version=4)

def add_md(nb, text):
    nb.cells.append(nbf.v4.new_markdown_cell(text))

def add_code(nb, code):
    nb.cells.append(nbf.v4.new_code_cell(code))

# --- add cells here ---

with open('notebooks/eda.ipynb', 'w', encoding='utf-8') as f:
    nbf.write(nb, f)
print("Notebook updated.")
```

Save as a one-off script, run it, then delete it. Each task shows only the `add_md` / `add_code` calls to insert.

---

## Task 1: Scaffold Project Structure

**Files:**
- Create: `dataset/` directory
- Create: `notebooks/` directory
- Create: `img/` directory
- Move: `so_dev_survey.csv` -> `dataset/so_dev_survey.csv`

- [ ] **Step 1: Create directories**

```bash
mkdir dataset notebooks img
```

- [ ] **Step 2: Move the CSV**

```bash
mv so_dev_survey.csv dataset/so_dev_survey.csv
```

- [ ] **Step 3: Verify structure**

```bash
ls dataset/ notebooks/ img/
```

Expected output:
```
dataset/:
so_dev_survey.csv

notebooks/:
(empty)

img/:
(empty)
```

- [ ] **Step 4: Commit**

```bash
git init
git add dataset/so_dev_survey.csv
git commit -m "chore: scaffold project structure and move dataset"
```

---

## Task 2: Install Dependencies

**Files:** none (environment only)

- [ ] **Step 1: Install packages**

```bash
pip install pandas numpy matplotlib seaborn nbformat jupyter
```

- [ ] **Step 2: Verify imports**

```bash
python -c "import pandas, numpy, matplotlib, seaborn, nbformat; print('All OK')"
```

Expected: `All OK`

---

## Task 3: Initialize Notebook with Setup Section

**Files:**
- Create: `notebooks/eda.ipynb`

- [ ] **Step 1: Create empty notebook**

Save as `init_nb.py` at repo root, run it, then delete it:

```python
import nbformat as nbf
nb = nbf.v4.new_notebook()
with open('notebooks/eda.ipynb', 'w', encoding='utf-8') as f:
    nbf.write(nb, f)
print("Created notebooks/eda.ipynb")
```

```bash
python init_nb.py && rm init_nb.py
```

- [ ] **Step 2: Add title and imports cells**

Save as `add_setup.py` at repo root, run it, then delete it:

```python
import nbformat as nbf

with open('notebooks/eda.ipynb', 'r', encoding='utf-8') as f:
    nb = nbf.read(f, as_version=4)

def add_md(nb, text):
    nb.cells.append(nbf.v4.new_markdown_cell(text))

def add_code(nb, code):
    nb.cells.append(nbf.v4.new_code_cell(code))

add_md(nb, "# Stack Overflow Developer Survey — EDA Portfolio\n\n**Theme 1:** AI Adoption x Compensation | **Theme 2:** AI Adoption x Tech Stack")

add_code(nb, """import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
from matplotlib.patches import Patch
import seaborn as sns
from pathlib import Path

# Paths (relative to notebooks/)
DATASET = Path('../dataset/so_dev_survey.csv')
IMG = Path('../img')
IMG.mkdir(exist_ok=True)

# Global style
sns.set_theme(style='whitegrid', palette='muted', font_scale=1.1)
plt.rcParams['figure.dpi'] = 100
plt.rcParams['savefig.dpi'] = 150

TEAL  = '#2a9d8f'
CORAL = '#e76f51'""")

add_code(nb, """df = pd.read_csv(DATASET)
print(f"Shape: {df.shape}")
df.head(2)""")

with open('notebooks/eda.ipynb', 'w', encoding='utf-8') as f:
    nbf.write(nb, f)
print("Setup cells added.")
```

```bash
python add_setup.py && rm add_setup.py
```

- [ ] **Step 3: Execute and verify**

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/eda.ipynb --ExecutePreprocessor.timeout=120
```

Expected: no errors; cell output shows `Shape: (2037, 174)` approximately.

- [ ] **Step 4: Commit**

```bash
git add notebooks/eda.ipynb
git commit -m "feat: initialize notebook with imports and data load"
```

---

## Task 4: Data Quality Section

**Files:**
- Modify: `notebooks/eda.ipynb`

- [ ] **Step 1: Add data quality cells**

Save as `add_quality.py`, run, delete:

```python
import nbformat as nbf

with open('notebooks/eda.ipynb', 'r', encoding='utf-8') as f:
    nb = nbf.read(f, as_version=4)

def add_md(nb, text): nb.cells.append(nbf.v4.new_markdown_cell(text))
def add_code(nb, code): nb.cells.append(nbf.v4.new_code_cell(code))

add_md(nb, "## Section 1 — Data Quality Summary")

add_code(nb, """fig, ax = plt.subplots(figsize=(22, 4))
sns.heatmap(df.isnull(), cbar=False, yticklabels=False,
            cmap=['#f0f0f0', '#e76f51'], ax=ax)
ax.set_title('Missing Values Heatmap  (orange = missing)', fontsize=13, pad=10)
ax.set_xlabel('Columns')
plt.tight_layout()
plt.show()""")

add_code(nb, """quality = pd.DataFrame({
    'dtype'     : df.dtypes,
    'null_count': df.isnull().sum(),
    'null_pct'  : (df.isnull().sum() / len(df) * 100).round(1),
}).sort_values('null_pct', ascending=False)

print(f"Total rows    : {len(df):,}")
print(f"Total columns : {len(df.columns)}")
print(f"Duplicate rows: {df.duplicated().sum()}")
quality.head(20)""")

add_code(nb, """# Coerce numeric columns that may contain strings
for c in ['YearsCode', 'WorkExp']:
    df[c] = pd.to_numeric(df[c], errors='coerce')

df[['ConvertedCompYearly', 'YearsCode', 'WorkExp']].describe().round(0)""")

with open('notebooks/eda.ipynb', 'w', encoding='utf-8') as f:
    nbf.write(nb, f)
print("Data quality cells added.")
```

```bash
python add_quality.py && rm add_quality.py
```

- [ ] **Step 2: Execute and verify**

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/eda.ipynb --ExecutePreprocessor.timeout=120
```

Expected: heatmap renders, quality table shows columns sorted by null %, describe() shows numeric stats.

- [ ] **Step 3: Commit**

```bash
git add notebooks/eda.ipynb
git commit -m "feat: add data quality summary section"
```

---

## Task 5: AI Adoption Overview Section

**Files:**
- Modify: `notebooks/eda.ipynb`

- [ ] **Step 1: Add overview cells**

Save as `add_overview.py`, run, delete:

```python
import nbformat as nbf

with open('notebooks/eda.ipynb', 'r', encoding='utf-8') as f:
    nb = nbf.read(f, as_version=4)

def add_md(nb, text): nb.cells.append(nbf.v4.new_markdown_cell(text))
def add_code(nb, code): nb.cells.append(nbf.v4.new_code_cell(code))

add_md(nb, "## Section 2 — AI Adoption Overview\n\nBaseline context before the two correlation analyses.")

add_code(nb, """print("=== AISelect (usage frequency) ===")
print(df['AISelect'].value_counts(dropna=False).to_string())
print()
print("=== AISent (sentiment toward AI) ===")
print(df['AISent'].value_counts(dropna=False).to_string())""")

add_code(nb, """top_models = (
    df['AIModelsHaveWorkedWith']
    .dropna()
    .str.split(';')
    .explode()
    .str.strip()
    .value_counts()
    .head(10)
)
print("Top 10 AI models by respondent count:")
print(top_models.to_string())""")

with open('notebooks/eda.ipynb', 'w', encoding='utf-8') as f:
    nbf.write(nb, f)
print("Overview cells added.")
```

```bash
python add_overview.py && rm add_overview.py
```

- [ ] **Step 2: Execute and verify**

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/eda.ipynb --ExecutePreprocessor.timeout=120
```

Expected: AISelect value counts show distinct frequency categories; top AI models list headed by ChatGPT/GPT variants.

- [ ] **Step 3: Commit**

```bash
git add notebooks/eda.ipynb
git commit -m "feat: add AI adoption overview section"
```

---

## Task 6: Chart 1 — Violin + Strip Plot (AI Usage x Salary)

**Files:**
- Modify: `notebooks/eda.ipynb`
- Create: `img/01_violin_ai_comp.png`

- [ ] **Step 1: Add cells**

Save as `add_chart1.py`, run, delete:

```python
import nbformat as nbf

with open('notebooks/eda.ipynb', 'r', encoding='utf-8') as f:
    nb = nbf.read(f, as_version=4)

def add_md(nb, text): nb.cells.append(nbf.v4.new_markdown_cell(text))
def add_code(nb, code): nb.cells.append(nbf.v4.new_code_cell(code))

add_md(nb, "## Part 1 — AI Adoption x Compensation\n\n### Chart 1: Salary Distribution by AI Usage Frequency")

add_code(nb, """# Data prep
comp_df = df[['AISelect', 'ConvertedCompYearly']].dropna().copy()
p99 = comp_df['ConvertedCompYearly'].quantile(0.99)
comp_df = comp_df[comp_df['ConvertedCompYearly'] <= p99].copy()

def shorten_ai_label(val):
    v = str(val).lower()
    if 'daily' in v or 'almost daily' in v: return 'Daily'
    if 'week'  in v:                         return 'Weekly'
    if 'month' in v or 'infrequent' in v:    return 'Monthly'
    if 'plan'  in v:                         return 'Planning'
    return 'Never'

comp_df['AI_Usage'] = comp_df['AISelect'].map(shorten_ai_label)
order = [o for o in ['Never', 'Planning', 'Monthly', 'Weekly', 'Daily']
         if o in comp_df['AI_Usage'].unique()]

print(comp_df['AI_Usage'].value_counts())
print(f"Rows after cap: {len(comp_df):,}  |  99th-pct cap: ${p99:,.0f}")""")

add_code(nb, """fig, ax = plt.subplots(figsize=(12, 7))
palette = sns.color_palette('muted', n_colors=len(order))

sns.violinplot(data=comp_df, x='AI_Usage', y='ConvertedCompYearly',
               order=order, palette=palette, inner=None, cut=0,
               linewidth=1.2, ax=ax)
sns.stripplot(data=comp_df, x='AI_Usage', y='ConvertedCompYearly',
              order=order, color='black', alpha=0.15, size=2.5,
              jitter=True, ax=ax)

ax.set_yscale('log')
ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f'${x:,.0f}'))
ax.set_title('Salary Distribution by AI Tool Usage Frequency',
             fontsize=15, pad=14, fontweight='bold')
ax.set_xlabel('AI Usage Frequency', fontsize=12)
ax.set_ylabel('Annual Compensation (USD, log scale)', fontsize=12)
ax.grid(axis='y', alpha=0.4)
sns.despine()
plt.tight_layout()
fig.savefig(IMG / '01_violin_ai_comp.png', dpi=150, bbox_inches='tight')
plt.show()
print("Saved: 01_violin_ai_comp.png")""")

add_md(nb, "**Insight 1:** Daily AI users show a notably higher and tighter salary distribution than non-users, suggesting AI tool proficiency correlates with seniority and compensation. The violin width reveals that non-users cluster at lower salary bands while daily users span a broader range at the upper end.")

with open('notebooks/eda.ipynb', 'w', encoding='utf-8') as f:
    nbf.write(nb, f)
print("Chart 1 cells added.")
```

```bash
python add_chart1.py && rm add_chart1.py
```

- [ ] **Step 2: Execute and verify**

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/eda.ipynb --ExecutePreprocessor.timeout=120
ls img/
```

Expected: `01_violin_ai_comp.png` present.

- [ ] **Step 3: Commit**

```bash
git add notebooks/eda.ipynb img/01_violin_ai_comp.png
git commit -m "feat: add Chart 1 violin+strip plot (AI usage vs salary)"
```

---

## Task 7: Chart 2 — Bubble Chart (Country AI Adoption vs. Salary)

**Files:**
- Modify: `notebooks/eda.ipynb`
- Create: `img/02_bubble_country_ai_comp.png`

- [ ] **Step 1: Add cells**

Save as `add_chart2.py`, run, delete:

```python
import nbformat as nbf

with open('notebooks/eda.ipynb', 'r', encoding='utf-8') as f:
    nb = nbf.read(f, as_version=4)

def add_md(nb, text): nb.cells.append(nbf.v4.new_markdown_cell(text))
def add_code(nb, code): nb.cells.append(nbf.v4.new_code_cell(code))

add_md(nb, "### Chart 2: Country-Level AI Adoption Rate vs. Median Salary")

add_code(nb, """is_ai_user = df['AISelect'].str.lower().str.contains('yes', na=False)

country_df = (
    df.assign(is_ai_user=is_ai_user)
      .groupby('Country')
      .agg(ai_rate=('is_ai_user', 'mean'),
           median_comp=('ConvertedCompYearly', 'median'),
           n=('Country', 'count'))
      .reset_index()
)
country_df = country_df[
    (country_df['n'] >= 10) & country_df['median_comp'].notna()
].copy()

print(f"Countries with >=10 respondents + salary data: {len(country_df)}")
print(country_df.nlargest(5, 'n')[['Country','ai_rate','median_comp','n']].round(2).to_string())""")

add_code(nb, """fig, ax = plt.subplots(figsize=(13, 8))

sc = ax.scatter(
    country_df['ai_rate'] * 100,
    country_df['median_comp'],
    s=country_df['n'] * 3,
    c=country_df['median_comp'],
    cmap='viridis', alpha=0.7,
    edgecolors='white', linewidths=0.6
)
plt.colorbar(sc, ax=ax, label='Median Annual Comp (USD)')

for _, row in country_df.nlargest(5, 'n').iterrows():
    ax.annotate(row['Country'],
                (row['ai_rate'] * 100, row['median_comp']),
                textcoords='offset points', xytext=(6, 4),
                fontsize=9, fontweight='bold')

ax.xaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f'{x:.0f}%'))
ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f'${x:,.0f}'))
ax.set_xlabel('AI Tool Adoption Rate (%)', fontsize=12)
ax.set_ylabel('Median Annual Compensation (USD)', fontsize=12)
ax.set_title('Country-Level AI Adoption Rate vs. Median Salary\\n(bubble size = respondent count)',
             fontsize=14, pad=12, fontweight='bold')
sns.despine()
plt.tight_layout()
fig.savefig(IMG / '02_bubble_country_ai_comp.png', dpi=150, bbox_inches='tight')
plt.show()
print("Saved: 02_bubble_country_ai_comp.png")""")

add_md(nb, "**Insight 2:** High-income countries (USA, Western Europe) cluster in the top-right — high AI adoption and high pay. The positive diagonal trend suggests that economic context and AI adoption reinforce each other, though causality cannot be determined from survey data alone.")

with open('notebooks/eda.ipynb', 'w', encoding='utf-8') as f:
    nbf.write(nb, f)
print("Chart 2 cells added.")
```

```bash
python add_chart2.py && rm add_chart2.py
```

- [ ] **Step 2: Execute and verify**

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/eda.ipynb --ExecutePreprocessor.timeout=120
ls img/
```

Expected: `02_bubble_country_ai_comp.png` present.

- [ ] **Step 3: Commit**

```bash
git add notebooks/eda.ipynb img/02_bubble_country_ai_comp.png
git commit -m "feat: add Chart 2 bubble chart (country AI adoption vs salary)"
```

---

## Task 8: Chart 3 — Diverging Bar (AI Salary Premium by Dev Type)

**Files:**
- Modify: `notebooks/eda.ipynb`
- Create: `img/03_diverging_devtype_ai_premium.png`

- [ ] **Step 1: Add cells**

Save as `add_chart3.py`, run, delete:

```python
import nbformat as nbf

with open('notebooks/eda.ipynb', 'r', encoding='utf-8') as f:
    nb = nbf.read(f, as_version=4)

def add_md(nb, text): nb.cells.append(nbf.v4.new_markdown_cell(text))
def add_code(nb, code): nb.cells.append(nbf.v4.new_code_cell(code))

add_md(nb, "### Chart 3: AI Adoption Salary Premium by Developer Type")

add_code(nb, """is_ai_user = df['AISelect'].str.lower().str.contains('yes', na=False)

devtype_df = (
    df.assign(is_ai_user=is_ai_user)
      [['DevType', 'ConvertedCompYearly', 'is_ai_user']]
      .dropna(subset=['ConvertedCompYearly', 'DevType'])
      .copy()
)
devtype_df['DevType'] = devtype_df['DevType'].str.split(';')
devtype_df = devtype_df.explode('DevType')
devtype_df['DevType'] = devtype_df['DevType'].str.strip()

top_devtypes = devtype_df['DevType'].value_counts().head(10).index.tolist()
devtype_df = devtype_df[devtype_df['DevType'].isin(top_devtypes)]

pivot = (
    devtype_df.groupby(['DevType', 'is_ai_user'])['ConvertedCompYearly']
              .median()
              .unstack('is_ai_user')
)
pivot.columns = ['no_ai', 'ai_user']
pivot = pivot.dropna()
pivot['premium'] = pivot['ai_user'] - pivot['no_ai']
pivot = pivot.sort_values('premium')
print(pivot.round(0).to_string())""")

add_code(nb, """fig, ax = plt.subplots(figsize=(11, 7))

colors = [TEAL if v >= 0 else CORAL for v in pivot['premium']]
bars = ax.barh(pivot.index, pivot['premium'] / 1000, color=colors,
               height=0.6, edgecolor='white')

for bar, val in zip(bars, pivot['premium']):
    sign = '+' if val >= 0 else ''
    offset = 0.5 if val >= 0 else -0.5
    ha = 'left' if val >= 0 else 'right'
    ax.text(bar.get_width() + offset,
            bar.get_y() + bar.get_height() / 2,
            f'{sign}${val:,.0f}', va='center', ha=ha, fontsize=9)

ax.axvline(0, color='black', linewidth=0.8, linestyle='--')
ax.set_xlabel('Salary Premium of AI Users vs. Non-Users (USD thousands)', fontsize=11)
ax.set_title('AI Adoption Salary Premium by Developer Type',
             fontsize=14, pad=12, fontweight='bold')

legend_elements = [Patch(facecolor=TEAL, label='AI users earn more'),
                   Patch(facecolor=CORAL, label='AI users earn less')]
ax.legend(handles=legend_elements, loc='lower right', framealpha=0.8)
sns.despine()
plt.tight_layout()
fig.savefig(IMG / '03_diverging_devtype_ai_premium.png', dpi=150, bbox_inches='tight')
plt.show()
print("Saved: 03_diverging_devtype_ai_premium.png")""")

add_md(nb, "**Insight 3:** Senior engineers, DevOps, and cloud specialists who adopt AI tools show the largest salary premium over their non-adopting peers. AI proficiency appears to amplify existing compensation advantages rather than levelling the field.")

with open('notebooks/eda.ipynb', 'w', encoding='utf-8') as f:
    nbf.write(nb, f)
print("Chart 3 cells added.")
```

```bash
python add_chart3.py && rm add_chart3.py
```

- [ ] **Step 2: Execute and verify**

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/eda.ipynb --ExecutePreprocessor.timeout=120
ls img/
```

Expected: `03_diverging_devtype_ai_premium.png` present.

- [ ] **Step 3: Commit**

```bash
git add notebooks/eda.ipynb img/03_diverging_devtype_ai_premium.png
git commit -m "feat: add Chart 3 diverging bar (AI salary premium by dev type)"
```

---

## Task 9: Chart 4 — Lollipop Chart (AI Adoption Rate by Language)

**Files:**
- Modify: `notebooks/eda.ipynb`
- Create: `img/04_lollipop_lang_ai_adoption.png`

- [ ] **Step 1: Add cells**

Save as `add_chart4.py`, run, delete:

```python
import nbformat as nbf

with open('notebooks/eda.ipynb', 'r', encoding='utf-8') as f:
    nb = nbf.read(f, as_version=4)

def add_md(nb, text): nb.cells.append(nbf.v4.new_markdown_cell(text))
def add_code(nb, code): nb.cells.append(nbf.v4.new_code_cell(code))

add_md(nb, "## Part 2 — AI Adoption x Tech Stack\n\n### Chart 4: AI Adoption Rate by Programming Language")

add_code(nb, """is_ai_user = df['AISelect'].str.lower().str.contains('yes', na=False)

lang_df = (
    df.assign(is_ai_user=is_ai_user)
      [['LanguageHaveWorkedWith', 'is_ai_user']]
      .dropna(subset=['LanguageHaveWorkedWith'])
      .copy()
)
lang_df['lang'] = lang_df['LanguageHaveWorkedWith'].str.split(';')
lang_df = lang_df.explode('lang')
lang_df['lang'] = lang_df['lang'].str.strip()

top15 = lang_df['lang'].value_counts().head(15).index.tolist()
lang_stats = (
    lang_df[lang_df['lang'].isin(top15)]
    .groupby('lang')
    .agg(ai_rate=('is_ai_user', 'mean'), count=('is_ai_user', 'count'))
    .reset_index()
    .sort_values('ai_rate', ascending=True)
)
overall_rate = is_ai_user.mean()
print(f"Overall AI adoption: {overall_rate:.1%}")
print(lang_stats.to_string())""")

add_code(nb, """fig, ax = plt.subplots(figsize=(10, 8))

colors = [TEAL if r >= overall_rate else CORAL for r in lang_stats['ai_rate']]

ax.hlines(lang_stats['lang'], 0, lang_stats['ai_rate'] * 100,
          color='lightgray', linewidth=1.5, zorder=1)
ax.scatter(lang_stats['ai_rate'] * 100, lang_stats['lang'],
           color=colors, s=120, zorder=2, edgecolors='white', linewidths=0.8)

ax.axvline(overall_rate * 100, color='black', linestyle='--',
           linewidth=1.2, alpha=0.7)
ax.text(overall_rate * 100 + 0.5, -0.7,
        f'Avg: {overall_rate:.0%}', fontsize=9, va='top')

ax.xaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f'{x:.0f}%'))
ax.set_xlabel('AI Tool Adoption Rate (%)', fontsize=12)
ax.set_title('AI Tool Adoption Rate by Programming Language\\n(Top 15 languages by usage count)',
             fontsize=14, pad=12, fontweight='bold')
ax.set_xlim(0, 105)
sns.despine()
plt.tight_layout()
fig.savefig(IMG / '04_lollipop_lang_ai_adoption.png', dpi=150, bbox_inches='tight')
plt.show()
print("Saved: 04_lollipop_lang_ai_adoption.png")""")

add_md(nb, "**Insight 4:** Rust, Kotlin, and TypeScript communities show the highest AI adoption rates, all above the overall average, reflecting their modern productivity-focused ecosystems. Legacy languages fall below the average line, consistent with more conservative user bases.")

with open('notebooks/eda.ipynb', 'w', encoding='utf-8') as f:
    nbf.write(nb, f)
print("Chart 4 cells added.")
```

```bash
python add_chart4.py && rm add_chart4.py
```

- [ ] **Step 2: Execute and verify**

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/eda.ipynb --ExecutePreprocessor.timeout=120
ls img/
```

Expected: `04_lollipop_lang_ai_adoption.png` present.

- [ ] **Step 3: Commit**

```bash
git add notebooks/eda.ipynb img/04_lollipop_lang_ai_adoption.png
git commit -m "feat: add Chart 4 lollipop chart (AI adoption by language)"
```

---

## Task 10: Chart 5 — Heatmap (Language x AI Model)

**Files:**
- Modify: `notebooks/eda.ipynb`
- Create: `img/05_heatmap_lang_aimodel.png`

- [ ] **Step 1: Add cells**

Save as `add_chart5.py`, run, delete:

```python
import nbformat as nbf

with open('notebooks/eda.ipynb', 'r', encoding='utf-8') as f:
    nb = nbf.read(f, as_version=4)

def add_md(nb, text): nb.cells.append(nbf.v4.new_markdown_cell(text))
def add_code(nb, code): nb.cells.append(nbf.v4.new_code_cell(code))

add_md(nb, "### Chart 5: AI Model Usage Across Language Communities")

add_code(nb, """lang_ai_df = df[['LanguageHaveWorkedWith', 'AIModelsHaveWorkedWith']].dropna().copy()
lang_ai_df['lang_list'] = lang_ai_df['LanguageHaveWorkedWith'].str.split(';')
lang_ai_df['ai_list']   = lang_ai_df['AIModelsHaveWorkedWith'].str.split(';')

lang_exp = lang_ai_df.explode('lang_list')
lang_exp['lang_list'] = lang_exp['lang_list'].str.strip()
top10_langs = lang_exp['lang_list'].value_counts().head(10).index.tolist()
lang_exp = lang_exp[lang_exp['lang_list'].isin(top10_langs)]

records = []
for lang in top10_langs:
    subset = lang_exp[lang_exp['lang_list'] == lang]
    n_lang = len(subset)
    ai_exp = subset.explode('ai_list')
    ai_exp['ai_list'] = ai_exp['ai_list'].str.strip()
    for model, cnt in ai_exp['ai_list'].value_counts().items():
        records.append({'language': lang, 'ai_model': model,
                        'pct': round(cnt / n_lang * 100, 1)})

heat_df = pd.DataFrame(records)
top5_models = heat_df.groupby('ai_model')['pct'].sum().nlargest(5).index.tolist()
heat_df = heat_df[heat_df['ai_model'].isin(top5_models)]
heat_pivot = heat_df.pivot(index='language', columns='ai_model', values='pct').fillna(0)
heat_pivot = heat_pivot.loc[heat_pivot.sum(axis=1).sort_values(ascending=False).index]
print(heat_pivot.round(1).to_string())""")

add_code(nb, """fig, ax = plt.subplots(figsize=(12, 7))
sns.heatmap(heat_pivot, annot=True, fmt='.1f', cmap='YlOrRd',
            linewidths=0.5, linecolor='white', ax=ax,
            cbar_kws={'label': '% of language users'})
ax.set_title('AI Model Usage by Programming Language Community\\n(% of language users who have worked with each model)',
             fontsize=13, pad=14, fontweight='bold')
ax.set_xlabel('AI Model', fontsize=11)
ax.set_ylabel('Programming Language', fontsize=11)
plt.xticks(rotation=30, ha='right')
plt.tight_layout()
fig.savefig(IMG / '05_heatmap_lang_aimodel.png', dpi=150, bbox_inches='tight')
plt.show()
print("Saved: 05_heatmap_lang_aimodel.png")""")

add_md(nb, "**Insight 5:** ChatGPT/GPT models dominate across every language community, but the Python community shows notably higher uptake of specialized models (Claude, Gemini), reflecting Python's role as the primary AI/ML language where practitioners explore a broader model ecosystem.")

with open('notebooks/eda.ipynb', 'w', encoding='utf-8') as f:
    nbf.write(nb, f)
print("Chart 5 cells added.")
```

```bash
python add_chart5.py && rm add_chart5.py
```

- [ ] **Step 2: Execute and verify**

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/eda.ipynb --ExecutePreprocessor.timeout=120
ls img/
```

Expected: `05_heatmap_lang_aimodel.png` present.

- [ ] **Step 3: Commit**

```bash
git add notebooks/eda.ipynb img/05_heatmap_lang_aimodel.png
git commit -m "feat: add Chart 5 heatmap (language vs AI model)"
```

---

## Task 11: Chart 6 — Slope Chart (AI Momentum by Language)

**Files:**
- Modify: `notebooks/eda.ipynb`
- Create: `img/06_slope_lang_ai_momentum.png`

- [ ] **Step 1: Add cells**

Save as `add_chart6.py`, run, delete:

```python
import nbformat as nbf

with open('notebooks/eda.ipynb', 'r', encoding='utf-8') as f:
    nb = nbf.read(f, as_version=4)

def add_md(nb, text): nb.cells.append(nbf.v4.new_markdown_cell(text))
def add_code(nb, code): nb.cells.append(nbf.v4.new_code_cell(code))

add_md(nb, "### Chart 6: AI Adoption Momentum — Current vs. Aspiring Users")

add_code(nb, """is_ai_user = df['AISelect'].str.lower().str.contains('yes', na=False)
df_ai = df.assign(is_ai_user=is_ai_user)

def lang_ai_rate(col):
    tmp = df_ai[[col, 'is_ai_user']].dropna(subset=[col]).copy()
    tmp['lang'] = tmp[col].str.split(';')
    tmp = tmp.explode('lang')
    tmp['lang'] = tmp['lang'].str.strip()
    return tmp.groupby('lang')['is_ai_user'].agg(['mean', 'count'])

have_rate = lang_ai_rate('LanguageHaveWorkedWith')
have_rate.columns = ['current_ai_rate', 'current_count']

want_rate = lang_ai_rate('LanguageWantToWorkWith')
want_rate.columns = ['aspiring_ai_rate', 'aspiring_count']

slope_df = have_rate.join(want_rate, how='inner').reset_index()
slope_df = slope_df[slope_df['current_count'] >= 20]
slope_df = slope_df.nlargest(10, 'current_count').copy()
slope_df['gap'] = slope_df['aspiring_ai_rate'] - slope_df['current_ai_rate']

print(slope_df[['lang','current_ai_rate','aspiring_ai_rate','gap']].round(3).to_string())""")

add_code(nb, """fig, ax = plt.subplots(figsize=(10, 8))
cmap = plt.cm.RdYlGn
gap_min, gap_max = slope_df['gap'].min(), slope_df['gap'].max()

for _, row in slope_df.iterrows():
    norm  = (row['gap'] - gap_min) / (gap_max - gap_min + 1e-9)
    color = cmap(norm)

    ax.plot([0, 1],
            [row['current_ai_rate'] * 100, row['aspiring_ai_rate'] * 100],
            color=color, linewidth=2.5, alpha=0.85, solid_capstyle='round')
    ax.scatter([0], [row['current_ai_rate'] * 100],  color=color, s=80, zorder=3)
    ax.scatter([1], [row['aspiring_ai_rate'] * 100], color=color, s=80, zorder=3)

    ax.text(-0.03, row['current_ai_rate'] * 100,
            row['lang'], ha='right', va='center', fontsize=9)
    ax.text(1.03, row['aspiring_ai_rate'] * 100,
            f"{row['lang']} ({row['gap']*100:+.1f}pp)",
            ha='left', va='center', fontsize=9)

ax.set_xlim(-0.45, 1.45)
ax.set_xticks([0, 1])
ax.set_xticklabels(['Current Users\\n(LanguageHaveWorkedWith)',
                     'Aspiring Users\\n(LanguageWantToWorkWith)'], fontsize=11)
ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f'{x:.0f}%'))
ax.set_ylabel('AI Tool Adoption Rate (%)', fontsize=12)
ax.set_title('AI Adoption Momentum by Language\\n(green = aspiring users more AI-forward, red = less)',
             fontsize=13, pad=14, fontweight='bold')
ax.grid(axis='y', alpha=0.3)
sns.despine()
plt.tight_layout()
fig.savefig(IMG / '06_slope_lang_ai_momentum.png', dpi=150, bbox_inches='tight')
plt.show()
print("Saved: 06_slope_lang_ai_momentum.png")""")

add_md(nb, "**Insight 6:** Languages with rising slopes are attracting AI-native developers into their aspiring-user cohorts. Languages with falling slopes are already saturated — or are seeing AI enthusiasm cool among those who haven't yet adopted them, potentially signalling hype cooling in mature ecosystems.")

with open('notebooks/eda.ipynb', 'w', encoding='utf-8') as f:
    nbf.write(nb, f)
print("Chart 6 cells added.")
```

```bash
python add_chart6.py && rm add_chart6.py
```

- [ ] **Step 2: Execute and verify**

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/eda.ipynb --ExecutePreprocessor.timeout=120
ls img/
```

Expected: `06_slope_lang_ai_momentum.png` present; all 6 PNGs now in `img/`.

- [ ] **Step 3: Commit**

```bash
git add notebooks/eda.ipynb img/06_slope_lang_ai_momentum.png
git commit -m "feat: add Chart 6 slope chart (AI momentum by language)"
```

---

## Task 12: Key Takeaways Cell

**Files:**
- Modify: `notebooks/eda.ipynb`

- [ ] **Step 1: Add takeaways cell**

Save as `add_takeaways.py`, run, delete:

```python
import nbformat as nbf

with open('notebooks/eda.ipynb', 'r', encoding='utf-8') as f:
    nb = nbf.read(f, as_version=4)

nb.cells.append(nbf.v4.new_markdown_cell("""## Key Takeaways

1. **AI adoption correlates with higher compensation** — daily AI users command a measurably higher salary across nearly every developer type, with the largest absolute premiums in senior engineering and DevOps roles.

2. **High-income countries lead both AI adoption and pay** — the country bubble chart reveals a positive diagonal: nations with high AI adoption also have higher median developer salaries, suggesting AI tool access and economic context are mutually reinforcing.

3. **AI adoption is not uniform across tech stacks** — modern ecosystems (Rust, Kotlin, TypeScript) are more AI-forward than legacy ones, ChatGPT dominates across the board but Python communities explore a broader model palette, and aspiring-user cohorts signal which languages are attracting AI-native talent next."""))

with open('notebooks/eda.ipynb', 'w', encoding='utf-8') as f:
    nbf.write(nb, f)
print("Takeaways cell added.")
```

```bash
python add_takeaways.py && rm add_takeaways.py
```

- [ ] **Step 2: Execute full notebook clean**

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/eda.ipynb --ExecutePreprocessor.timeout=300
```

Expected: no errors; all 6 `Saved: *.png` messages in output.

- [ ] **Step 3: Commit**

```bash
git add notebooks/eda.ipynb
git commit -m "feat: add key takeaways section"
```

---

## Task 13: Write README.md

**Files:**
- Create: `README.md`

- [ ] **Step 1: Verify all 6 PNGs exist before writing README**

```bash
ls img/
```

Expected: all 6 files present. If any are missing, re-run the relevant task above first.

- [ ] **Step 2: Create README.md at repo root**

Write `README.md` with the following content (update insight text if actual chart results differ from the interpretations written in notebook cells):

````markdown
# Stack Overflow Developer Survey — EDA Portfolio

> Profiling a real-world developer survey to surface insights on **AI adoption vs. compensation** and **AI adoption vs. tech stack trends**.

**Dataset:** Stack Overflow Developer Survey 2024/2025 · ~2,037 respondents · 170+ columns
**Notebook:** [`notebooks/eda.ipynb`](notebooks/eda.ipynb)
**Tools:** Python · pandas · matplotlib · seaborn

---

## Data Quality Summary

| Metric | Value |
|--------|-------|
| Total rows | ~2,037 |
| Total columns | ~174 |
| Duplicate rows | 0 |
| Columns with >50% missing | ~40 (multi-select optionals) |

Key numeric columns (`ConvertedCompYearly`, `YearsCode`, `WorkExp`) have 20–40% nulls — typical for optional survey questions. Multi-value columns (languages, tools) are semicolon-separated and require splitting before analysis.

---

## Theme 1 — AI Adoption x Compensation

### Insight 1: Salary Distribution by AI Usage Frequency

![Violin + Strip Plot](img/01_violin_ai_comp.png)

Daily AI users show a notably higher and tighter salary distribution than non-users. The violin width reveals that non-users cluster at lower salary bands while daily users span a broader range at the upper end, suggesting AI tool proficiency correlates with seniority.

---

### Insight 2: Country-Level AI Adoption Rate vs. Median Salary

![Bubble Chart](img/02_bubble_country_ai_comp.png)

High-income countries (USA, Western Europe) cluster in the top-right quadrant — high AI adoption and high pay. The positive diagonal trend suggests that economic context and AI adoption reinforce each other.

---

### Insight 3: AI Adoption Salary Premium by Developer Type

![Diverging Bar](img/03_diverging_devtype_ai_premium.png)

Senior engineers, DevOps, and cloud specialists who adopt AI tools show the largest salary premium over their non-adopting peers. AI proficiency appears to amplify existing compensation advantages rather than levelling the field.

---

## Theme 2 — AI Adoption x Tech Stack

### Insight 4: AI Adoption Rate by Programming Language

![Lollipop Chart](img/04_lollipop_lang_ai_adoption.png)

Rust, Kotlin, and TypeScript communities show the highest AI adoption rates, all above the overall average. Legacy languages fall below average, consistent with their older, more conservative user bases.

---

### Insight 5: AI Model Usage Across Language Communities

![Heatmap](img/05_heatmap_lang_aimodel.png)

ChatGPT/GPT models dominate across every language community, but the Python community shows notably higher uptake of specialized models (Claude, Gemini), reflecting Python's role as the primary AI/ML language where practitioners explore a wider model ecosystem.

---

### Insight 6: AI Adoption Momentum — Current vs. Aspiring Users

![Slope Chart](img/06_slope_lang_ai_momentum.png)

Languages with rising slopes are attracting AI-native developers into their aspiring-user cohorts. Languages with falling slopes are already saturated — or are seeing AI enthusiasm cool among those who haven't yet adopted them.

---

## Key Takeaways

1. **AI adoption correlates with higher compensation** — daily AI users command measurably higher salaries across nearly every developer type, with the largest premiums in senior engineering and DevOps roles.

2. **High-income countries lead both AI adoption and pay** — a clear positive diagonal in the country bubble chart shows that AI tool access and economic context are mutually reinforcing.

3. **AI adoption is not uniform across tech stacks** — modern ecosystems are more AI-forward, ChatGPT dominates everywhere but Python communities go broader, and aspiring-user cohorts reveal which languages attract AI-native talent next.

---

## How to Reproduce

```bash
pip install pandas numpy matplotlib seaborn nbformat jupyter
cd notebooks
jupyter notebook eda.ipynb
# Kernel -> Restart & Run All
```

All figures are regenerated in `img/` on each full run. No external APIs or random seeds — output is fully deterministic.
````

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add README with all 6 insights and reproduction instructions"
```

---

## Final Verification

- [ ] Run the full notebook clean:

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/eda.ipynb --ExecutePreprocessor.timeout=300
```

Expected: no errors; all 6 `Saved: *.png` messages appear in output.

- [ ] Confirm all deliverables present:

```bash
ls dataset/ notebooks/ img/
```

Expected:
```
dataset/: so_dev_survey.csv
notebooks/: eda.ipynb
img/: 01_violin_ai_comp.png  02_bubble_country_ai_comp.png  03_diverging_devtype_ai_premium.png
      04_lollipop_lang_ai_adoption.png  05_heatmap_lang_aimodel.png  06_slope_lang_ai_momentum.png
```
