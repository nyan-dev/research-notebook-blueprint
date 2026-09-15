# Research Notebook Pipeline Standard & Architecture Guide

This document defines the universal architectural standard, cell conventions, environment management, folder bootstrapping, and inter-notebook communication protocols for research, econometrics, and machine learning notebook pipelines.

AI assistants and human developers **MUST** read and adhere to these guidelines whenever creating a new notebook or modifying an existing one within any project adhering to this standard.

---

## 1. Core Philosophy

1. **Deterministic & Sequential**: Notebooks run in a numbered chain (`NB1` $\rightarrow$ `NB2` $\rightarrow$ `NB3` $\dots$). Outputs of `NB{N-1}` serve as deterministic inputs to `NB{N}`.
2. **Dual-Environment Parity**: Code must run identically on **Google Colab** (cloud GPU/Drive) and **Local Workstations** (Windows/Linux/macOS) without manual editing.
3. **Every Notebook Connects to Drive**: Because Google Colab assigns a fresh, ephemeral virtual machine to each notebook session, **every notebook must mount Google Drive and resolve paths in Cell 01**.
4. **Automated Folder Bootstrapping**: The folder structure is built automatically in the first notebook (`NB1`) if it does not already exist, and verified by all subsequent notebooks.
5. **Human & Agent Auditable**: Every code cell is indexed with a standardized `# Cell XX` header for instant referencing in discussion, debugging, and terminal grepping.
6. **Verified Handshakes**: Notebooks do not guess the state of upstream data; they formally validate upstream metadata using machine-readable JSON handshakes (`summary_NB{N-1}.json`).
7. **Separation of Concerns**: Single responsibility per cell. Never mix Drive mounting, imports, plotting configuration, or computation into a single monolithic block.

---

## 2. Standard Directory Architecture

Every project following this standard maintains a clean separation between raw data, interim transformations, model-ready matrices, and outputs:

```text
<PROJECT_ROOT>/
├── data/
│   ├── raw/                  # Immutable original datasets (read-only)
│   ├── interim/              # Cleaned, merged, and harmonized cross-sectional/time-series data
│   └── processed/            # Final model-ready feature matrices and target arrays
├── outputs/
│   ├── figures/              # High-resolution plots (PNG for preview, PDF/SVG for publication)
│   ├── tables/               # Formatted results, econometric tests, and summary tables (CSV / LaTeX)
│   ├── models/               # Saved model checkpoints, pickles (.pkl), and best weights
│   └── notebook_exports/     # Inter-notebook JSON Handshake summaries (summary_NB*.json)
├── notebooks/                # Sequential pipeline notebooks (NB1_*.ipynb, NB2_*.ipynb, etc.)
└── README.md
```

---

## 3. Scale-Adaptive Pipeline Decomposition

Not all data science projects require 6+ notebooks. Pipelines must be sized according to the project's scope:

### Tier 1: Coursework & Rapid Experimentation (1 – 2 Notebooks)
Ideal for small coursework assignments, term projects, or quick feasibility baselines:
* **NB1 — Exploratory Analysis & Feature Engineering**: Ingests raw data, performs visual EDA, cleans missingness, engineers features, and saves `data/processed/master_data.csv`.
* **NB2 — Modeling, Evaluation & Reporting**: Trains baseline and candidate models, evaluates cross-validation metrics, plots feature importance, and outputs final summary tables.

### Tier 2: Standard Master of Science / Capstone Project (3 – 4 Notebooks)
The recommended standard for graduate-level data science and applied machine learning projects:
* **NB1 — Data Ingestion, Cleaning & Harmonization**: Ingests raw data from single or multiple sources, standardizes timestamps and identifiers, cleans anomalies, and exports `data/interim/cleaned_data.csv`.
* **NB2 — Feature Engineering & Exploratory Data Analysis**: Performs in-depth EDA, generates lag features, interaction terms, or embeddings, runs correlation/multicollinearity checks, and exports `data/processed/master_features_ready.csv`.
* **NB3 — Model Tournament, Cross-Validation & Tuning**: Implements rigorous CV splits (Stratified, Group, or Time-Series), trains candidate model families, runs hyperparameter optimization (e.g., Optuna), and saves fitted models to `outputs/models/`.
* **NB4 — Model Evaluation, Explainability & Final Insights**: Generates test-set evaluation metrics, explainability analysis (e.g., SHAP, permutation importance), residual error diagnostics, and camera-ready figures/tables.

### Tier 3: Deep Empirical Research / Journal Publication (5 – 7 Notebooks)
Reserved for full-scale academic manuscripts, peer-reviewed journal papers, or master's/doctoral theses with extensive methodological subsections:
* **NB1 — Ingestion & Multi-Source Harmonization**: Merges disparate macro, micro, and environmental feeds; handles frequency aggregation.
* **NB2 — Econometric Diagnostics & Preprocessing**: Unit root tests (ADF/KPSS), variance inflation factor (VIF) tiers, regime dummies, and policy wedges.
* **NB3 — Model Tournament & Time-Series Validation**: Expanding-window cross-validation comparing classical econometric baselines (SARIMAX, ARDL, VAR) against ML models.
* **NB4 — Deep Explainability (SHAP)**: TreeExplainer / KernelExplainer global summaries, beeswarms, dependence plots, and temporal lag distributions.
* **NB5 — Causal Ablation & Policy Break / Event Studies**: Grouped feature removal experiments, permutation hypothesis tests, and event-study dynamics.
* **NB6 / NB7 — Publication Synthesis**: Multi-panel composite figures, LaTeX tables, and out-of-sample prediction stitching.

---

## 4. Google Drive & Folder Bootstrapping Protocol

### 4.1 Why Every Notebook Must Mount Google Drive
In Google Colab, **each notebook runs in an independent, ephemeral virtual machine**. When you switch from `NB1` to `NB2`, or reopen a notebook tomorrow:
- The virtual machine memory and local `/content/` disk are reset.
- Google Drive is **not** automatically mounted unless explicitly instructed.
- Therefore, **every single notebook must begin in Cell 01 with mounting Google Drive and setting the identical base path**.

### 4.2 How the Folder Structure is Created
1. **First Notebook (`NB1`) — Initializer / Bootstrapper**:
   - When starting a brand-new project, the directories on Google Drive do not exist yet.
   - In `NB1`, Cell 01 defines the folder tree and executes `folder.mkdir(parents=True, exist_ok=True)`.
   - Google Drive automatically creates the entire directory tree (`data/raw/`, `outputs/figures/`, etc.) in your Drive folder.
2. **Subsequent Notebooks (`NB2`, `NB3`, ... `NB{N}`) — Reconnect & Verify**:
   - Mounts Google Drive to access the persistent storage.
   - Connects to the same `BASE` directory.
   - Runs `folder.mkdir(parents=True, exist_ok=True)` as an idempotent safeguard (which takes 0 ms if the folders already exist, but guarantees no notebook crashes if an output folder was accidentally omitted).
   - Validates that the base project folder exists before executing further code.

---

## 5. The Universal "3-Zone" Notebook Template

Every notebook generated or edited **MUST** follow this 3-zone pattern:

```text
┌────────────────────────────────────────────────────────┐
│ ZONE 1: PREAMBLE & CONTRACT (Cells 00 to 03)           │
│  - Cell 00 [MD]   : Title, Abstract, I/O Contract      │
│  - Cell 01 [Code] : Mount Storage, Bootstrap Paths     │
│  - Cell 02 [Code] : Imports, Global Seeds & Style      │
│  - Cell 03 [Code] : Upstream Handshake Validation      │
├────────────────────────────────────────────────────────┤
│ ZONE 2: MODULAR EXECUTION UNITS (Cells 04 to N-2)      │
│  - Markdown Section Dividers (--- ## SECTION X)        │
│  - Single-Responsibility Code Cells                    │
│  - Standardized `# Cell XX — Category: Action` headers │
│  - Status checkmark printing at end of each cell       │
├────────────────────────────────────────────────────────┤
│ ZONE 3: PERSISTENCE & HANDOVER (Last 2 Cells)          │
│  - Cell N-1 [Code]: Artifact Export (Tables / Figures) │
│  - Cell N   [Code]: Downstream JSON Handshake Export   │
│  - Markdown Summary: Synthesis & Next Steps            │
└────────────────────────────────────────────────────────┘
```

---

## 6. Drop-in Zone 1 Template Code

### Cell 00 [Markdown] — Title & Contract
```markdown
# NB{X}: {Title of Stage}
## {Project or Paper Title}

**Stage Overview**: Brief 2-3 sentence description of what this notebook computes.
- **Inputs Consumed**: `path/to/input.csv` (from NB{X-1})
- **Outputs Produced**: `data/processed/...`, `outputs/figures/...`, `outputs/tables/...`
- **Handshake Artifact**: `outputs/notebook_exports/summary_NB{X}.json`
```

### Cell 01 [Code] — Mount Storage & Bootstrap Paths (Used in EVERY Notebook)
*Place this exact code at the top of every notebook. In NB1, it automatically creates any missing folders. In subsequent notebooks, it safely reconnects to them.*

```python
# Cell 01 — Mount Storage & Define Paths

import os
import sys
from pathlib import Path

# 1. Detect runtime environment
IN_COLAB = "google.colab" in sys.modules

# 2. Configure project base path
# Set this to your Google Drive folder name
PROJECT_FOLDER_NAME = "MyResearchProject"

if IN_COLAB:
  from google.colab import drive

  drive.mount("/content/drive")
  BASE = Path(f"/content/drive/MyDrive/{PROJECT_FOLDER_NAME}")
else:
  # Local workstation fallback (resolves repository root relative to notebook directory)
  BASE = (
      Path(__file__).resolve().parents[1]
      if "__file__" in locals()
      else Path.cwd().resolve()
  )
  if (BASE / "data").exists() is False and (Path.cwd() / "data").exists():
    BASE = Path.cwd().resolve()

# 3. Define canonical project subdirectories
RAW = BASE / "data" / "raw"
INTERIM = BASE / "data" / "interim"
PROCESSED = BASE / "data" / "processed"
FIGURES = BASE / "outputs" / "figures"
TABLES = BASE / "outputs" / "tables"
MODELS = BASE / "outputs" / "models"
EXPORTS = BASE / "outputs" / "notebook_exports"

ALL_DIRS = [RAW, INTERIM, PROCESSED, FIGURES, TABLES, MODELS, EXPORTS]

# 4. Bootstrap folder structure: creates all directories if they do not exist yet
created_count = 0
for folder in ALL_DIRS:
  if not folder.exists():
    folder.mkdir(parents=True, exist_ok=True)
    created_count += 1

print(
    f"✅ Environment: {'Google Colab' if IN_COLAB else 'Local Workstation'} |"
    f" Base: {BASE}"
)
if created_count > 0:
  print(f"📁 Bootstrapped {created_count} new directories in project tree.")
else:
  print("📁 All project directories verified and ready.")
```

### Cell 02 [Code] — Imports, Global Seeds & Publication Styling
```python
# Cell 02 — Install & Import Libraries, Seeds & Style Settings

import json
import os
import sys
import warnings

import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import seaborn as sns

warnings.filterwarnings('ignore')

# Reproducibility seed
SEED = 42
np.random.seed(SEED)

# Standard plotting typography & resolution
plt.rcParams.update({
    'font.family': 'serif',
    'font.size': 10,
    'axes.titlesize': 11,
    'axes.labelsize': 10,
    'xtick.labelsize': 9,
    'ytick.labelsize': 9,
    'legend.fontsize': 9,
    'figure.titlesize': 12,
    'figure.dpi': 150,
    'savefig.dpi': 300,
    'savefig.bbox': 'tight',
})

print('✅ Libraries loaded, seed fixed to 42, and visualization styles set.')
```

### Cell 03 [Code] — Upstream Handshake Validation (Skip for NB1)
*Ensures downstream notebooks fail immediately if upstream stages haven't produced required data.*

```python
# Cell 03 — Validate Upstream Handshake (from NB{X-1})

UPSTREAM_NOTEBOOK = "NB1"  # <--- Update to relevant upstream stage
upstream_handshake = EXPORTS / f"summary_{UPSTREAM_NOTEBOOK}.json"

if not upstream_handshake.exists():
  raise FileNotFoundError(
      f"❌ Missing upstream handshake: {upstream_handshake}\n"
      f"Please execute {UPSTREAM_NOTEBOOK} before running this notebook."
  )

with open(upstream_handshake, "r", encoding="utf-8") as f:
  upstream_meta = json.load(f)

# Assert essential upstream contract
assert upstream_meta.get("status") == "Success", (
    f"❌ Upstream stage {UPSTREAM_NOTEBOOK} did not finish successfully!"
)

print(f"✅ Upstream handshake verified from {UPSTREAM_NOTEBOOK}.")
print(f"   Shape: {upstream_meta.get('output_shape', 'N/A')}")
print(f"   Date Range: {upstream_meta.get('date_range', 'N/A')}")
```

---

## 7. Zone 2 Rules: Modular Execution Units

### 7.1 Cell Header Syntax
Every single code cell in Zone 2 must start with a standardized comment header on line 1:
```python
# Cell XX — <Category>: <Action Description>
```
- **`XX`**: Two-digit zero-padded integer (`04`, `05`, ..., `12`, `25`).
- **`<Category>`**: Standardized task type:
  - `Load`: Ingesting data from disk.
  - `Transform`: Cleaning, reshaping, calculating lags, merging.
  - `Feature`: Engineering specific features or transformations.
  - `Test`: Statistical or econometric tests (ADF, VIF, t-test, normality).
  - `Model`: Training, hyperparameter optimization, fitting.
  - `Plot`: Generating diagnostic or publication visual.
  - `Export`: Saving tables or models to disk.
- **`<Action Description>`**: Concise, specific summary of what the cell accomplishes.

*Examples*:
```python
# Cell 04 — Load: Read Cleaned Interim Dataset
# Cell 05 — Feature: Compute Rolling Volatility & Lags
# Cell 06 — Test: Augmented Dickey-Fuller Stationarity Suite
# Cell 07 — Model: XGBoost Hyperparameter Search with Optuna
```

### 7.2 Code Cell Rules
1. **Single Responsibility**: One cell should do one logical operation. Never lump data cleaning, model training, and plotting into a single monolithic cell.
2. **Deterministic Logging**: Every cell must conclude with a clear print statement indicating success and summary statistics:
   ```python
   print(
       f"✅ Created feature: rolling_std. Matrix shape: {df.shape}, Missing:"
       f" {df['rolling_std'].isna().sum()}"
   )
```
3. **No Unnumbered Cells**: Code cells must never omit the `# Cell XX` header. Markdown headers do not consume code cell numbers.

---

## 8. Zone 3 Rules: Handover & The JSON Handshake Protocol

The final code cell in every notebook exports `summary_NB{X}.json`. This provides an auditable, machine-readable contract for downstream notebooks, automation scripts, and LLMs.

### Standard Handshake Schema
```python
# Cell {N} — JSON Handshake Summary (NB{X} → NB{X+1})

import json

summary = {
    "notebook_name": "NB2_Feature_Engineering",
    "status": "Success",
    "execution_timestamp": pd.Timestamp.now().isoformat(),
    "inputs_consumed": ["data/interim/cleaned_data.csv"],
    "primary_output_file": "data/processed/master_features_ready.csv",
    "output_shape": list(df.shape),
    "date_range": f"{df.index.min().date()} to {df.index.max().date()}",
    "total_observations": len(df),
    "target_variables": ["target_y"],
    "feature_groups": {
        "macro": ["interest_rate", "cpi"],
        "technical": ["rolling_std_3m", "lag_1"],
    },
    "diagnostics": {
        "stationary_features": ["rolling_std_3m"],
        "high_vif_features": ["cpi_lag1"],
    },
    "key_output_artifacts": [
        "data/processed/master_features_ready.csv",
        "outputs/tables/nb2_feature_summary.csv",
        "outputs/figures/nb2_correlation_heatmap.png",
    ],
    "downstream_instructions": {
        "intended_recipient": "NB3_Model_Tournament",
        "recommended_cv_splits": 5,
    },
}

summary_path = EXPORTS / "summary_NB2.json"
with open(summary_path, "w", encoding="utf-8") as f:
  json.dump(summary, f, indent=4)

print(f"✅ Handshake summary exported successfully to {summary_path}")
```

---

## 9. Visual & Table Artifact Standards

1. **Deterministic Naming**: Prefix all saved files with the notebook number:
   - Figures: `outputs/figures/nb{X}_{descriptive_snake_case}.png`
   - Tables: `outputs/tables/nb{X}_{descriptive_snake_case}.csv`
   - Models: `outputs/models/nb{X}_{model_name}_{target}.pkl`
2. **Publication Quality Plots**:
   - Always save figures with `dpi=300` and `bbox_inches='tight'`.
   - For academic submissions, save both `.png` (for instant preview / Colab display) and vector `.pdf` or `.svg` (for camera-ready manuscripts):
     ```python
     fig.savefig(FIGURES / 'nb2_feature_heatmap.png', dpi=300)
     fig.savefig(FIGURES / 'nb2_feature_heatmap.pdf')
```
3. **Table Formatting**:
   - Always export tables with readable column headers and rounded floating point decimals:
     ```python
     results_df.round(4).to_csv(
         TABLES / 'nb3_cv_performance.csv', index=False
     )
```

---

## 10. Directives for AI Assistants

When an AI assistant is asked to write, refactor, or fix notebooks in projects governed by this guide:

1. **Before Generating or Editing**:
   - Inspect the existing notebooks to determine the project's scale tier (Tier 1, Tier 2, or Tier 3).
   - Check the highest notebook index (`NB{N}`) and current cell numbers.
   - Always include the canonical Cell 01 Google Drive mounting and folder bootstrapping block. Never use raw hardcoded string paths.
2. **When Inserting a Cell**:
   - If inserting a cell between existing cells (e.g., between `# Cell 05` and `# Cell 06`), re-sequence all downstream cell numbers in that notebook to maintain clean, sequential ordering.
3. **When Modifying Features or Output Formats**:
   - Check if downstream notebooks depend on the columns or shapes being modified.
   - Update the final JSON Handshake cell to reflect any schema modifications.
4. **Never Hardcode Colab-Only Paths**:
   - Avoid direct strings like `pd.read_csv('/content/drive/MyDrive/...')`. Always use the standardized `Path` variables (`PROCESSED / 'file.csv'`).
