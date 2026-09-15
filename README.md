<div align="center">

# 📓 Research Notebook Blueprint 1.0
### An AI-Agent-Ready Framework for Deterministic, Reproducible Multi-Notebook Data Science Pipelines

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Google Colab](https://img.shields.io/badge/Google%20Colab-Ready-orange.svg)](https://colab.research.google.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![AI Agent Compatible](https://img.shields.io/badge/AI--Agent-Antigravity%20%7C%20Cursor%20%7C%20Claude-purple.svg)](#install-as-an-ai-agent-skill)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](#)

*Eliminate notebook spaghetti, path breaks between Google Colab and local machines, and broken inter-notebook data flows.*

</div>

---

## 🌪️ The Problem: "Jupyter Notebook Hell"

Jupyter notebooks are great for exploratory analysis, but when scaling research or Master's-level data science pipelines across multiple notebooks, friction quickly accumulates:

* 💥 **Environment Locking**: Hardcoding `/content/drive/MyDrive/...` immediately crashes code when pulled locally or run in VS Code.
* 🧩 **Silent Upstream Breakages**: `NB2` silently fails or produces corrupted calculations because a column or feature was renamed in `NB1`.
* 🤖 **AI Assistant Chaos**: LLM coding assistants (ChatGPT, Claude, Cursor) generate unnumbered cells, mix data cleaning with plotting, and disrupt execution order.
* 🍝 **Monolithic Spaghetti Cells**: A single 120-line cell mounts Drive, imports 25 packages, cleans data, runs regressions, and plots 3 figures.

---

## ⚡ The Solution: The 4 Core Pillars

This framework introduces a structured, production-grade standard that keeps multi-notebook pipelines deterministic, human-auditable, and AI-agent ready:

```text
┌────────────────────────────────────────────────────────┐
│ ZONE 1: PREAMBLE & CONTRACT (Cells 00 to 03)           │
│  - Cell 00 [MD]   : Stage Title, Abstract, I/O Contract│
│  - Cell 01 [Code] : Mount Storage & Dual-Path Resolver │
│  - Cell 02 [Code] : Imports, Global Seeds & Style      │
│  - Cell 03 [Code] : Upstream Handshake Validation      │
├────────────────────────────────────────────────────────┤
│ ZONE 2: MODULAR EXECUTION UNITS (Cells 04 to N-2)      │
│  - Markdown Section Dividers (--- ## SECTION X)        │
│  - Single-Responsibility Code Cells                    │
│  - Standardized `# Cell XX — Category: Action` headers │
│  - Deterministic status checkmark printing             │
├────────────────────────────────────────────────────────┤
│ ZONE 3: PERSISTENCE & HANDOVER (Last 2 Cells)          │
│  - Cell N-1 [Code]: Artifact Export (Tables / Figures) │
│  - Cell N   [Code]: Downstream JSON Handshake Export   │
│  - Markdown Summary: Synthesis & Handover Notes        │
└────────────────────────────────────────────────────────┘
```

### 1. The Universal "3-Zone" Notebook Template
Every notebook is divided into three functional zones:
- **Zone 1 (Preamble & Contract)**: Connects storage, loads packages, sets reproducibility seeds (`np.random.seed(42)`), and asserts upstream data integrity.
- **Zone 2 (Modular Execution Units)**: Granular, single-responsibility cells prefixed with `# Cell XX — Category: Action`.
- **Zone 3 (Persistence & Handover)**: Saves publication figures (PNG + PDF), results tables, and exports downstream metadata.

### 2. Dual-Environment Path Parity (Colab + Local)
No more commenting out `drive.mount('/content/drive')`! A single 15-line block detects whether it is running on Google Colab or your local machine, mounts Drive only when needed, and standardizes `pathlib.Path` objects.

### 3. Automated Google Drive Folder Bootstrapping
In Google Colab, virtual machines are ephemeral and reset every session.
* **`NB1`** automatically bootstraps your entire directory tree on Google Drive (`data/raw/`, `outputs/figures/`, etc.) if it doesn't exist yet.
* **Subsequent Notebooks (`NB2+`)** automatically reconnect to the exact same persistent storage with zero crashes.

### 4. The Two-Way JSON Handshake Protocol
Notebooks do not guess whether upstream data is correct. Every notebook exports a machine-readable summary (`summary_NB{X}.json`) in its final cell:
* The next notebook **validates this JSON in Cell 03** before executing a single line of code.
* If upstream data is missing, corrupted, or stale, execution halts immediately with an informative error message.

---

## 📐 Standard Directory Architecture

```text
my-research-project/
├── data/
│   ├── raw/                  # Immutable original data feeds (read-only)
│   ├── interim/              # Cleaned, merged, and harmonized tables
│   └── processed/            # Final model-ready feature matrices
├── outputs/
│   ├── figures/              # Publication plots (PNG 300dpi + Vector PDF)
│   ├── tables/               # Formatted results & diagnostic tables (CSV / LaTeX)
│   ├── models/               # Saved model checkpoints & pickles (.pkl)
│   └── notebook_exports/     # Inter-notebook JSON Handshake summaries
├── notebooks/                # Sequential pipeline notebooks (NB1_*.ipynb, NB2_*.ipynb)
└── README.md
```

---

## 🎯 Scale-Adaptive Pipeline Decomposition

Not every project requires 6+ notebooks. Choose the tier that matches your scope:

| Tier | Best For | Notebook Count | Pipeline Flow |
| :--- | :--- | :--- | :--- |
| **Tier 1: Coursework & Rapid Sprints** | Homework, term projects, quick baselines | **1 – 2** | `NB1` (EDA & Features) → `NB2` (Models & Reporting) |
| **Tier 2: Master of Science / Capstones** | Graduate projects, thesis coursework | **3 – 4** | `NB1` → `NB2` → `NB3` → `NB4` (Evaluation & SHAP) |
| **Tier 3: Empirical Journal Publications** | Peer-reviewed manuscripts, doctorates | **5 – 7** | Ingestion → Econometrics → Tournament → SHAP → Ablation → Publication |

---

## 🚀 Quick Start (1 Minute)

### Option 1: Drop-in Storage & Path Setup (Cell 01)
Copy this block into line 1 of your Colab or Jupyter notebook:

```python
# Cell 01 — Mount Storage & Define Paths

import os
import sys
from pathlib import Path

IN_COLAB = "google.colab" in sys.modules
PROJECT_FOLDER_NAME = (
    "MyResearchProject"  # <--- Your Google Drive subfolder name
)

if IN_COLAB:
  from google.colab import drive

  drive.mount("/content/drive")
  BASE = Path(f"/content/drive/MyDrive/{PROJECT_FOLDER_NAME}")
else:
  BASE = (
      Path(__file__).resolve().parents[1]
      if "__file__" in locals()
      else Path.cwd().resolve()
  )

# Canonical project subdirectories
RAW = BASE / "data" / "raw"
INTERIM = BASE / "data" / "interim"
PROCESSED = BASE / "data" / "processed"
FIGURES = BASE / "outputs" / "figures"
TABLES = BASE / "outputs" / "tables"
MODELS = BASE / "outputs" / "models"
EXPORTS = BASE / "outputs" / "notebook_exports"

ALL_DIRS = [RAW, INTERIM, PROCESSED, FIGURES, TABLES, MODELS, EXPORTS]

# Bootstrap directories automatically
for folder in ALL_DIRS:
  folder.mkdir(parents=True, exist_ok=True)

print(
    f"✅ Environment: {'Google Colab' if IN_COLAB else 'Local Workstation'} |"
    f" Base: {BASE}"
)
```

### Option 2: Use Pre-Built Starter Templates
Ready-to-use template notebooks are available in the [`templates/`](templates/) folder:
1. [`01_ingestion_template.ipynb`](templates/01_ingestion_template.ipynb): Bootstraps Drive directories, ingests raw data, and exports `summary_NB1.json`.
2. [`02_preprocessing_template.ipynb`](templates/02_preprocessing_template.ipynb): Validates NB1 handshake, engineers features, and outputs model-ready datasets.
3. [`03_modeling_template.ipynb`](templates/03_modeling_template.ipynb): Validates NB2 handshake, trains candidate models, and persists checkpoints.

---

## 🤖 Install as an AI Agent Skill

This framework includes an official agent skill specification in [`skills/notebook-pipeline/SKILL.md`](skills/notebook-pipeline/SKILL.md).

When paired with AI pair-programming assistants like **Google Antigravity**, **Cursor**, or **Claude**, you can instruct your agent to strictly follow these standards.

### For Antigravity / Agentic IDEs:
Copy the skill folder into your workspace:
```bash
mkdir -p .agents/skills/
cp -r skills/notebook-pipeline .agents/skills/
```

### For Claude / ChatGPT / Cursor System Prompts:
Simply attach [`notebook-standards.md`](notebook-standards.md) to your prompt:
> *"Read `notebook-standards.md` and follow the 3-Zone architecture, dual Colab/Local path resolver, and `# Cell XX` numbering convention when generating this notebook."*

---

## 📖 Complete Documentation

For the comprehensive, full-length technical specification (including handshake JSON schemas, publication plotting typography, and error-handling protocols), read **[`notebook-standards.md`](notebook-standards.md)**.

---

## 📄 License

Distributed under the **MIT License**. See [`LICENSE`](LICENSE) for details. Feel free to use, modify, and integrate this template into your own academic or commercial projects.
