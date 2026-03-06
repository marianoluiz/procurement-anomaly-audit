# GitHub Copilot Instructions

## Project Context

This is a **thesis project** focused on enhancing the Isolation Forest algorithm for audit prioritization in Philippine public procurement data.

**Title:** AN ENHANCEMENT OF ISOLATION FOREST ALGORITHM USING FEATURE-WEIGHTED SPLIT SELECTION FOR AUDIT PRIORITIZATION IN PUBLIC PROCUREMENT

## Dataset

- **Source:** Philippine Government Electronic Procurement System (PhilGEPS)
- **Current Data:** `data/2025_(JAN-MAR).csv` (724 records, 46 features)
- **Domain:** Public procurement transactions including bids, awards, awardees, and contract details

### Key Features
- Financial: `Approved Budget of the Contract (ABC)`, `Contract Amount`
- Temporal: `Published Date`, `Closing Date`, `Award Date`, contract dates
- Categorical: Procuring entities, regions, procurement modes, business categories
- Awardee info: Organization names, locations, sizes

## Code Style Preferences

### Python
- Use **type hints** for all functions
- Follow **PEP 8** style guide
- Prefer **pandas** for data manipulation
- Use **scikit-learn** conventions for ML models
- Write **docstrings** (Google style) for all classes and functions
- Add **inline comments** for complex logic

### Data Processing
- Handle missing values explicitly (document strategy)
- Always validate data types after loading
- Use meaningful variable names (e.g., `contract_amount` not `ca`)
- Log data transformations for reproducibility

### Machine Learning
- Set `random_state=42` for reproducibility
- Use pipelines for preprocessing + modeling
- Separate train/test/validation clearly
- Document hyperparameters and their rationale

### File Naming
- Snake case for Python files: `feature_engineering.py`
- Descriptive notebook names: `01_exploratory_data_analysis.ipynb`
- Version outputs: `results_v1.csv`, `model_v2.pkl`

## Domain-Specific Guidelines

### Audit Risk Factors (High Priority Features for Weighting)
When implementing feature weighting, prioritize these audit-relevant features:
1. **Price variance:** `(Contract Amount - ABC) / ABC` - Large deviations indicate potential overpricing
2. **Timeline anomalies:** Unusually short/long durations, awards before closing dates
3. **Entity concentration:** Same awardee winning multiple high-value contracts
4. **Procurement mode appropriateness:** High-value contracts using simplified procurement
5. **Geographic patterns:** Unusual regional concentration

### Data Quality Considerations
- **Duplicates:** Same contract may appear multiple times with different UNSPSC codes - handle aggregation carefully
- **NULL values:** Many contract timeline fields are NULL - implement robust imputation or exclusion strategy
- **Zero values:** `Item Budget` often 0.00 - use `Contract Amount` for financial analysis
- **Date parsing:** Ensure proper datetime conversion for timeline features

### Anomaly Detection Best Practices
- Standard Isolation Forest is the **baseline** - always compare against it
- Feature weights should be **interpretable** and **auditor-friendly**
- Focus on **precision over recall** - auditors have limited resources, false positives are costly
- Generate **explanations** for flagged anomalies (which features contributed most)

## Project Structure Awareness

```
src/
├── data_processing/     # Data cleaning, feature engineering, preprocessing
├── models/             # Isolation Forest implementations
├── evaluation/         # Metrics, visualization, interpretation
└── utils/              # Configuration, helpers

notebooks/              # Jupyter notebooks for analysis
data/                  # Raw, processed, features
experiments/           # Experiment tracking
results/              # Outputs, metrics, plots
```

## Common Tasks

### Loading Data
```python
import pandas as pd

df = pd.read_csv('data/2025_(JAN-MAR).csv', parse_dates=['Published Date', 'Closing Date', 'Award Date'])
```

### Feature Engineering Pattern
```python
def create_price_variance_feature(df: pd.DataFrame) -> pd.DataFrame:
    """
    Calculate price variance as (Contract Amount - ABC) / ABC.
    
    Args:
        df: DataFrame with 'Contract Amount' and 'Approved Budget of the Contract (ABC)' columns
        
    Returns:
        DataFrame with added 'price_variance' column
    """
    df = df.copy()
    df['price_variance'] = (df['Contract Amount'] - df['Approved Budget of the Contract (ABC)']) / df['Approved Budget of the Contract (ABC)']
    return df
```

### Model Implementation Pattern
```python
from sklearn.ensemble import IsolationForest
from typing import Dict, Any

class WeightedIsolationForest:
    """Enhanced Isolation Forest with feature-weighted split selection."""
    
    def __init__(self, feature_weights: Dict[str, float], **kwargs):
        """
        Initialize weighted isolation forest.
        
        Args:
            feature_weights: Dictionary mapping feature names to weights
            **kwargs: Additional parameters for IsolationForest
        """
        self.feature_weights = feature_weights
        self.kwargs = kwargs
```

## Testing Requirements

- Write unit tests for all feature engineering functions
- Validate data transformations with assertions
- Test edge cases (all NULL, zero variance, single value)
- Verify model outputs are reproducible with same random_state

## Documentation Standards

- Every function needs a docstring explaining purpose, args, returns
- Complex algorithms should have step-by-step comments
- Document assumptions (e.g., "Assumes Contract Amount > 0")
- Add TODO comments for future improvements

## Experimentation

- Track experiments systematically (consider MLflow or simple CSV log)
- Save configurations with results
- Version models and datasets
- Document negative results (what didn't work and why)

## Performance Considerations

- This is research code, prioritize **clarity over speed**
- Use vectorized pandas operations when possible
- Profile only if performance becomes an issue
- Document computational complexity of custom algorithms

## Thesis-Specific Reminders

- This is **academic work** - cite sources, document methodology
- Focus on **novelty** in feature weighting approach
- Maintain **reproducibility** - all results must be recreatable
- Generate publication-quality visualizations (clear labels, legends, captions)
- Keep code clean for thesis appendix/GitHub repository submission
