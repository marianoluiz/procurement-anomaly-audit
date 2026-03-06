# Enhancement of Isolation Forest Algorithm for Public Procurement Audit

Bachelor's Thesis Project - Anomaly Detection in Philippine Government Procurement Data

## Quick Start

### 1. Set Up Virtual Environment (Recommended)

#### On Linux/Mac (bash):
```bash
# Create virtual environment
python3 -m venv .venv

# Activate it
source .venv/bin/activate
```

#### On Windows (PowerShell):
```bash
# Create virtual environment
python -m venv .venv

# Activate it
.venv\Scripts\Activate.ps1
```

### 2. Install Dependencies
```bash
# Install pip-tools
pip install pip-tools

# Install dependencies
pip-sync requirements.txt
```

### 3. Run Analysis
Start with the notebooks in order:
1. `notebooks/01_exploratory_data_analysis.ipynb`
2. `notebooks/02_data_preprocessing.ipynb`
3. `notebooks/03_baseline_isolation_forest.ipynb`
4. `notebooks/04_feature_weighting_experiments.ipynb`
5. `notebooks/05_weighted_isolation_forest.ipynb`
6. `notebooks/06_results_and_evaluation.ipynb`

## Project Structure
```
procurement/
├── data/               # Dataset files
├── notebooks/          # Jupyter notebooks for analysis
├── src/               # Python modules
├── results/           # Output files and visualizations
└── experiments/       # Experiment tracking
```

## Dataset
- **Source:** Philippine Government Electronic Procurement System (PhilGEPS)
- **Period:** Q1 2025 (Jan-Mar)
- **Records:** 724 procurement transactions
- **Features:** 46 columns

## Documentation
- [Project Plan](PROJECT_PLAN.md) - Full thesis roadmap
- [Copilot Instructions](.github/copilot-instructions.md) - AI coding context
