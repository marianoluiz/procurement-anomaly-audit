# AN ENHANCEMENT OF ISOLATION FOREST ALGORITHM USING FEATURE-WEIGHTED SPLIT SELECTION FOR AUDIT PRIORITIZATION IN PUBLIC PROCUREMENT

## 🎯 Project Overview

This thesis project aims to enhance the traditional Isolation Forest anomaly detection algorithm by incorporating **feature-weighted split selection** to improve audit prioritization in Philippine public procurement data.

### Research Objectives
1. Develop a feature-weighted variant of the Isolation Forest algorithm
2. Apply it to public procurement data for identifying high-risk transactions
3. Improve audit prioritization accuracy compared to standard Isolation Forest
4. Provide interpretable results for procurement auditors

---

## 📊 Current Data Assessment

### Dataset: `data/2025_(JAN-MAR).csv`

**File Statistics:**
- **Total Records:** 724 rows (723 procurement records + header)
- **Time Period:** January - March 2025
- **Data Source:** Philippine Government Electronic Procurement System (PhilGEPS)

**Key Features Available (46 columns):**

#### 1. **Entity Information**
- `Procuring Entity` - Government agency/department
- `Region`, `Province`, `City/Municipality` - Geographic location
- `Government Branch` - Executive, Legislative, Judicial
- `PE Organization Type` - Type and grouping of procuring entity

#### 2. **Procurement Details**
- `Bid Reference No.` - Unique bid identifier
- `Notice Title` - Description of procurement
- `Classification` - Goods/Services/Consulting Services/Infrastructure
- `Procurement Mode` - Public Bidding, Small Value Procurement, etc.
- `Business Category` - Type of service/product
- `Source of Funds` - Funding source
- `Approved Budget of the Contract (ABC)` - Budget allocated

#### 3. **Timeline Features**
- `Published Date` - When bid was posted
- `Closing Date` - Deadline for submission
- `Contract Duration` - Length of contract
- `Calendar Type` - Days/months/years

#### 4. **Award Information**
- `Award Reference No.` - Award identifier
- `Award Date` - When award was given
- `Contract Amount` - Actual contract value
- `Award Notice Status` - Posted/Published status
- `Contract Effectivity Date` - When contract starts
- `Contract End Date` - When contract ends

#### 5. **Awardee Information**
- `Awardee Organization Name` - Winning bidder
- `Country/Region/Province/City of Awardee` - Awardee location
- `Awardee Size` - Micro/Small/Medium/Large enterprise
- `Awardee Joint Venture` - JV participation

#### 6. **Item/Lot Details**
- `UNSPSC Code` & `UNSPSC Description` - Standard product codes
- `Item/Lot Name` & `Description`
- `Quantity` & `Unit of Measure`
- `Item Budget`

### 🔍 Data Quality Observations

**Strengths:**
- ✅ Rich feature set with 46 variables covering multiple dimensions
- ✅ Mix of categorical and numerical features (good for feature weighting)
- ✅ Contains critical audit-relevant fields (ABC, Contract Amount, dates, entity info)
- ✅ Geographic and organizational hierarchy available
- ✅ Temporal features for timeline analysis

**Potential Issues:**
- ⚠️ **Many NULL values** in contract timeline fields (Notice to Proceed Date, Effectivity Date, End Date)
- ⚠️ **Duplicate/repeated entries** - Same contract with multiple UNSPSC codes (e.g., DPWH bridge project appears ~40+ times)
- ⚠️ **Item Budget often 0.00** - May need to use overall Contract Amount instead
- ⚠️ **Limited temporal scope** - Only 3 months of data (may need more historical data)

**Anomaly Detection Candidates:**
- Price discrepancies: ABC vs Contract Amount variance
- Timeline irregularities: Duration inconsistencies, award date patterns
- Entity concentration: Same awardee winning multiple contracts
- Geographic patterns: Regional procurement concentration
- Procurement mode appropriateness: High-value contracts using simplified modes

---

## � Feature Selection Strategy

### ✅ Columns You Should DEFINITELY Use

These columns contain numerical and categorical data that directly relate to fraud/corruption risk:

| Column | Why It's Useful | What Anomaly It Might Reveal |
|--------|----------------|------------------------------|
| **Approved Budget of the Contract (ABC)** | The official budget. This is your baseline. | A contract awarded far below ABC (maybe poor quality) or suspiciously close to ABC (maybe rigged). |
| **Contract Amount** | What they actually paid. Compare with ABC. | The difference between ABC and Contract Amount is a key feature. Too small a difference? Too large? |
| **Published Date** | When bidding started. | If the window between publish and closing is too short, it might be designed to exclude other bidders. |
| **Closing Date** | When bidding ended. | Same as above. Calculate the duration (Closing - Published) as a feature. |
| **Award Date** | When winner was chosen. | If award happens suspiciously fast after closing, maybe the fix was in. |
| **Contract Duration** | How long the project should take. | A 44-month bridge project (normal) vs. a 44-month catering job (anomaly). |
| **Item Budget** | Budget for specific items (in Multi Lot). | Useful for detecting overpricing on specific items within a larger contract. |
| **Quantity** | How many units they bought. | Unusually high or low quantities for a given item type. |
| **UNSPSC Code** | The category of what's being bought. | A company that usually sells computers suddenly winning a catering contract. |

### 🔧 Columns You Should TRANSFORM (Create New Features)

Sometimes raw data isn't useful by itself, but you can combine columns to create powerful features:

| New Feature | How to Create It | Why It's Powerful |
|-------------|------------------|-------------------|
| **Bid Spread** | `ABC - Contract Amount` | A tiny spread might indicate collusion (bidder knew exactly the budget). A huge spread might indicate a lowball bid to win and then ask for change orders later. |
| **Bidding Duration** | `Closing Date - Published Date` | Too short = red flag. Too long = maybe unusual, but less likely to be fraud. |
| **Award Speed** | `Award Date - Closing Date` | If the government awards the contract the next day, did they really evaluate all bids properly? |
| **Price Per Unit** | `Item Budget / Quantity` | This lets you compare prices across different bids for the same item. If one agency pays ₱5,000 for a laptop and another pays ₱50,000 for the same laptop, that's an anomaly. |
| **Winner Frequency** | Count how many times a company wins in the dataset | A company winning 50 contracts while everyone else wins 1-2 is worth a look. |
| **Price Variance %** | `(Contract Amount - ABC) / ABC * 100` | Percentage deviation from budget (normalized measure). |
| **Procurement Mode Risk Score** | Encode based on risk level | Small Value Procurement for high amounts = higher risk. |
| **Awardee-Entity Pair Frequency** | Count unique awardee-procuring entity combinations | Same company repeatedly winning from same agency. |

### ❌ Columns to EXCLUDE from Model Training

These columns are either too random, too text-heavy, or too repetitive to be useful for isolation forest:

| Column | Why Skip from Model | **BUT Keep for...** |
|--------|---------------------|---------------------|
| **Bid Reference No.** | It's a random ID. The algorithm might flag bids with "weird" numbers, which means nothing. | ✅ **Identification** - Essential for tracking which bid was flagged |
| **Award Reference No.** | Another random ID with no semantic meaning. | ✅ **Identification** - Links bid to award record |
| **Notice Title** | Too much text. It would need heavy NLP processing, which complicates your model. | ✅ **Reporting** - Helps auditors understand what was procured |
| **Item/Lot Description** | Same as above. Too much unstructured text. | ✅ **Reporting** - Details for case study analysis |
| **Awardee Organization Name** | Don't feed the name into the model directly. Instead, create a feature like "Winner Frequency" based on the name. | ✅ **Reporting & Feature Engineering** - Use for creating Winner Frequency, but keep for identifying who won |
| **Procuring Entity** | Use for feature engineering (entity-awardee pairs) but don't encode the name itself. | ✅ **Reporting** - Which agency made the suspicious procurement |
| **Classification** | Categories like Goods/Services may have value, but test first. | ✅ **Reporting & Potential Feature** - Context for interpreting anomalies |
| **Country of Awardee** | Almost all are "Philippines." No variation = useless for anomaly detection. | ✅ **Reporting** - Flag if foreign awardee appears |
| **Region/Province/City** | Low priority for initial model. Add later if needed for geographic analysis. | ✅ **Reporting & Future Feature** - Geographic pattern analysis |
| **PE Organization Type** | Most are "NGA" or "GOCC." Not much variation, so not a strong signal. | ✅ **Reporting** - Context on entity type |
| **Trade Agreement** | Likely constant or very low variance. | ✅ **Reporting** - Regulatory context |

**Key Principle:** 🔑 **Keep ALL columns in your dataset** - just separate them into:
- **Model features** (numerical/engineered features for Isolation Forest)
- **Identification columns** (IDs, names, titles for post-detection reporting)
- **Context columns** (descriptive fields for interpretation and case studies)

### 🔍 Feature Engineering Priority List

**Phase 1 (Essential Features):**
1. Price variance: `(Contract Amount - ABC) / ABC`
2. Bidding duration: Days between published and closing
3. Award speed: Days between closing and award
4. Contract amount (absolute value)
5. ABC (absolute value)

**Phase 2 (Risk Indicators):**
6. Winner frequency (per awardee)
7. Price per unit (where applicable)
8. Contract duration appropriateness (by classification)
9. Procurement mode encoding
10. Business category encoding

**Phase 3 (Advanced Features):**
11. Awardee-entity pair frequency
12. Geographic concentration scores
13. Temporal patterns (month, quarter, year)
14. UNSPSC category diversity per awardee
15. Bid spread absolute vs percentage

---

## �🏗️ Project Structure

```
procurement/
├── data/
│   ├── raw/                    # Raw data files
│   │   └── 2025_(JAN-MAR).csv
│   ├── processed/              # Cleaned and preprocessed data
│   └── features/               # Engineered features
├── src/
│   ├── data_processing/
│   │   ├── cleaning.py         # Data cleaning and validation
│   │   ├── feature_engineering.py  # Feature creation
│   │   └── preprocessing.py    # Encoding, scaling
│   ├── models/
│   │   ├── isolation_forest.py # Standard Isolation Forest
│   │   ├── weighted_isolation_forest.py  # Enhanced algorithm
│   │   └── feature_weighting.py  # Weight calculation methods
│   ├── evaluation/
│   │   ├── metrics.py          # Performance metrics
│   │   ├── visualization.py    # Plots and charts
│   │   └── interpretation.py   # Feature importance analysis
│   └── utils/
│       ├── config.py           # Configuration settings
│       └── helpers.py          # Utility functions
├── notebooks/
│   ├── 01_exploratory_data_analysis.ipynb
│   ├── 02_data_preprocessing.ipynb
│   ├── 03_baseline_isolation_forest.ipynb
│   ├── 04_feature_weighting_experiments.ipynb
│   ├── 05_weighted_isolation_forest.ipynb
│   └── 06_results_and_evaluation.ipynb
├── experiments/                # Experiment tracking
├── results/                    # Output files, metrics, plots
├── docs/                       # Documentation
├── tests/                      # Unit tests
├── requirements.txt            # Python dependencies
├── README.md                   # Project README
└── PROJECT_PLAN.md            # This file
```

---

## 📋 Implementation Roadmap

### Phase 1: Data Preparation & EDA (Week 1-2)
- [ ] Load and explore the 2025 Q1 dataset
- [ ] Handle missing values and NULL fields
- [ ] Deduplicate entries (handle multi-UNSPSC records)
- [ ] Create derived features:
  - Price variance: `(Contract Amount - ABC) / ABC`
  - Timeline anomalies: Award date vs closing date gaps
  - Entity frequency: Count contracts per awardee
  - Regional concentration metrics
- [ ] Statistical analysis and visualization
- [ ] Identify ground truth anomalies (if labels available)

### Phase 2: Baseline Implementation (Week 3)
- [ ] Implement standard Isolation Forest
- [ ] Encode categorical variables
- [ ] Scale numerical features
- [ ] Run baseline anomaly detection
- [ ] Establish performance benchmarks
- [ ] Analyze detected anomalies

### Phase 3: Feature Weighting Methodology (Week 4-5)
- [ ] Research feature weighting approaches:
  - Information gain
  - Mutual information
  - Domain expert weights (audit risk factors)
  - Statistical measures (variance, correlation with anomaly scores)
- [ ] Implement weight calculation functions
- [ ] Validate weights against domain knowledge
- [ ] Document weighting rationale

### Phase 4: Enhanced Algorithm Development (Week 6-7)
- [ ] Modify Isolation Forest split selection:
  - Incorporate feature weights into split probability
  - Implement weighted random feature selection
  - Adjust isolation tree construction
- [ ] Test different weight integration strategies
- [ ] Validate algorithmic correctness
- [ ] Performance optimization

### Phase 5: Experimentation & Evaluation (Week 8-9)
- [ ] Compare weighted vs standard Isolation Forest:
  - Anomaly detection accuracy
  - Precision/Recall/F1 (if labels available)
  - Feature importance interpretability
  - Computational efficiency
- [ ] Sensitivity analysis on hyperparameters
- [ ] Cross-validation and robustness testing
- [ ] Statistical significance testing

### Phase 6: Results & Documentation (Week 10-12)
- [ ] Generate visualizations (ROC curves, anomaly distributions)
- [ ] Case study analysis: Real anomaly examples
- [ ] Write thesis chapters
- [ ] Prepare presentation materials
- [ ] Code documentation and README

---

## 🔬 Research Questions

1. **RQ1:** How can feature importance be effectively incorporated into Isolation Forest's split selection mechanism?
2. **RQ2:** Does feature weighting improve anomaly detection accuracy in public procurement data?
3. **RQ3:** Which weighting method (information-based vs domain-expert vs hybrid) performs best?
4. **RQ4:** How does the enhanced algorithm compare to baseline in terms of interpretability and computational cost?

---

## 🛠️ Technical Stack

### Core Libraries
```python
# Data Processing
pandas>=2.0.0
numpy>=1.24.0

# Machine Learning
scikit-learn>=1.3.0
scipy>=1.11.0

# Visualization
matplotlib>=3.7.0
seaborn>=0.12.0
plotly>=5.14.0

# Notebook
jupyter>=1.0.0
ipykernel>=6.25.0

# Utilities
python-dateutil>=2.8.0
openpyxl>=3.1.0  # For Excel exports
```

### Development Tools
- Version Control: Git
- Environment: Python 3.10+
- IDE: VS Code / Jupyter

---

## 📊 Expected Deliverables

1. **Code Repository**
   - Clean, documented codebase
   - Reproducible experiments
   - Unit tests for core functions

2. **Thesis Document**
   - Literature review on Isolation Forest and procurement fraud detection
   - Methodology chapter (algorithm enhancement details)
   - Results and analysis
   - Conclusions and future work

3. **Supplementary Materials**
   - Jupyter notebooks with analysis
   - Result visualizations
   - Performance comparison tables
   - Detected anomaly reports

---

## 📈 Success Metrics

1. **Technical:**
   - Improved AUC-ROC compared to baseline (if labels exist)
   - Higher precision in top-k anomalies
   - Reduced false positive rate
   - Maintained computational efficiency

2. **Practical:**
   - Detected anomalies align with audit risk factors
   - Feature importance matches domain expertise
   - Results interpretable by non-technical auditors
   - Scalable to larger datasets

---

## 🚨 Risk Factors & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Limited labeled data** | Cannot validate accuracy | Use proxy labels (extreme price variance, expert review), unsupervised metrics |
| **Data quality issues** | Poor model performance | Robust preprocessing, outlier handling, data augmentation |
| **Small dataset size** | Overfitting, low statistical power | Acquire more historical data, cross-validation, synthetic data generation |
| **Algorithm complexity** | Implementation bugs | Unit testing, compare with reference implementations, code review |
| **Class imbalance** | Biased detection | Stratified sampling, appropriate evaluation metrics |

---

## 📚 Key References (to explore)

1. Liu, F. T., Ting, K. M., & Zhou, Z. H. (2008). Isolation forest. *ICDM 2008*
2. Feature selection in anomaly detection
3. Fraud detection in public procurement
4. Philippine procurement regulations (RA 9184)
5. Weighted ensemble methods

---

## 🔄 Next Immediate Steps

1. **Set up Python environment** and install dependencies
2. **Create initial data exploration notebook** to understand the dataset deeply
3. **Research additional data sources** - acquire more historical procurement data (2024, 2023)
4. **Consult domain experts** - understand what auditors look for in procurement fraud
5. **Literature review** - survey latest anomaly detection and procurement fraud papers

---

## 📝 Notes

- Consider requesting more historical data from PhilGEPS (2022-2024)
- Look into COA (Commission on Audit) red flags for procurement
- Investigate if there are existing audit reports to use as labels
- Think about real-time deployment considerations for future work

---

**Last Updated:** March 7, 2026  
**Author:** [Your Name]  
**Advisor:** [Advisor Name]  
**Institution:** [University Name]
