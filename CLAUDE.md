# The Intender Gap: Predicting AI Tool Adoption Among Developers
> Using the Stack Overflow 2025 Developer Survey to identify what
> separates developers who intend to adopt AI tools from those who
> actually do — and what product interventions close the gap.

---

## Business Context

**Problem Statement:**
84% of developers now use or plan to use AI tools — yet 5.3%
are stuck in limbo: they intend to adopt but haven't acted.
This project builds a predictive model to identify what
distinguishes these "intenders" from developers who either
actively use AI tools or have rejected them entirely.
The output is a PM-ready targeting strategy: who to reach,
what barrier to address, and where in the adoption funnel
to intervene.

**Why It Matters:**
A PM at any AI developer tools company (GitHub Copilot, Cursor,
Salesforce Einstein, Claude Code) faces a concrete growth
problem: there is a population of high-intent, low-action
developers who have declared they want to adopt AI tools but
haven't crossed over. Understanding what blocks them — and
whether blockers differ across developer profiles — directly
informs onboarding design, enterprise sales strategy, and
product messaging. Stack Overflow's own published analysis
shows this group as a single percentage and moves on. This
project builds the targeting engine a PM would actually use.

**Target Audience for This Project:**
AI PM / Product Analytics hiring managers at Big Tech,
SaaS / PLG companies, and Healthcare Tech firms.
Demonstrates: product funnel thinking, predictive modeling,
feature selection with business justification, and the ability
to translate statistical findings into PM-ready decisions.

---

## Dataset

**Source:** Stack Overflow 2025 Annual Developer Survey
https://survey.stackoverflow.co/
(Click "Download Full Data Set (CSV)" next to 2025)

**File Locations:**
- `data/raw/survey_results_public.csv` → main dataset (never modify)
- `data/raw/survey_results_schema.csv` → data dictionary
  (maps column codes to full question text — read this first)

**Working sample:**
- Raw rows: 49,196
- Valid rows after dropping AISelect NAs: 33,720
- NA rows confirmed MCAR via dropout curve analysis (see DECISIONS.md D-07)

**Target Variable:**
- Column: `AISelect`
- Question: "Do you currently use AI tools in your development process?"
- Four groups defined for this project:

| Group | AISelect value | n | Role in analysis |
|-------|---------------|---|-----------------|
| Active users | Daily + Weekly | 21,841 | Reference group in EDA |
| Casual users | Monthly + Infrequently | 4,628 | Reference group in EDA |
| Intenders | No, but I plan to soon | 1,797 | PRIMARY target class |
| Rejectors | No, and I don't plan to | 5,454 | Comparison class |

Primary model: Intenders vs. Rejectors only (binary classification).
Both groups are non-users — holding access and habit constant,
isolating psychological and attitudinal drivers of intent.

**Confirmed Feature List:**

AI attitude columns (universal questions — asked of all respondents):
- `AISent` — overall stance toward AI tools
- `AIThreat` — whether AI is perceived as a job threat
- `AIFrustration` — frustrations encountered (multi-select)
- `LearnCodeAI` — whether they actively learned AI tooling in past year

AI attitude columns (conditional — check missingness for intenders):
- `AIComplex` — perceived capability of AI at complex tasks
  (may be skipped for non-users — verify before including)

Developer profile columns:
- `Age` — ordinal ranges, encode as ordinal
- `EdLevel` — ordinal, encode as ordinal
- `DevType` — multi-select, explode to binary indicators
- `Industry` — single-select categorical
- `Employment` — collapse to 3 categories before encoding
  (Employed full-time / Independent or Freelance / Student or other)
- `YearsCode` — continuous numeric
- `WorkExp` — continuous numeric
  (compute correlation with YearsCode; if r > 0.85, drop WorkExp)
- `OrgSize` — single-select categorical
- `ICorPM` — individual contributor vs. people manager
- `RemoteWork` — borderline include, cut if no variation in EDA
- `PurchaseInfluence` — tech purchase influence at org

**Columns explicitly excluded and why:**
- `AISelect` sub-values outside intender/rejector: not in model scope
- `AIAgents`, `AIAgentChange`: target leakage (post-adoption behavior)
- `AIModelsChoice`: target leakage (requires active usage)
- `AIHuman`: future hypothetical, indirect mechanism
- `MainBranch`: sampling artifact, near-zero variance
- `Country`: fails actionability gate
- All RO columns (TechEndorse_*, TechOppose_*, JobSatPoints_*,
  SO_Actions_*): measure general preferences, not AI-specific
- All TE columns (AIOpen, AIExplain, etc.): require NLP, out of scope

**Data Notes:**
- 15,471 NA rows in AISelect dropped — confirmed MCAR via dropout
  analysis (see DECISIONS.md D-07). Do not impute.
- Multi-select columns (DevType, AIFrustration) store values as
  semicolon-separated strings: "Python;JavaScript;SQL"
  These must be exploded into binary indicator columns.
- Age and EdLevel are categorical strings, not numbers.
  Encode as ordered ordinal categories.
- After multi-select explosion, expect ~35-40 total features
  before Random Forest trimming.
- AIComplex may have high NA for intenders/rejectors if it was
  conditionally routed to current users only. Check this in EDA
  before including in the model.

---

## Tech Stack

- **Language:** Python 3.13
- **Environment:** JupyterLab
- **Core Libraries:** pandas, numpy, matplotlib, seaborn
- **Modeling Libraries:** scikit-learn
- **Specific sklearn components needed:**
  - RandomForestClassifier
  - LogisticRegression
  - DecisionTreeClassifier
  - train_test_split (stratified)
  - classification_report, confusion_matrix, roc_auc_score
  - RocCurveDisplay
- **Versioning:** Git / GitHub

---

## Folder Structure

```
so-2025-ai-adoption/
│
├── data/
│   ├── raw/
│   │   ├── survey_results_public.csv   → original, never modified
│   │   └── survey_results_schema.csv   → data dictionary
│   └── processed/
│       ├── df_valid.csv                → after NA removal
│       ├── df_intenders_rejectors.csv  → model-ready binary dataset
│       └── feature_names.txt           → list of final features used
│
├── notebooks/
│   ├── 01_eda.ipynb           → exploratory data analysis (Act 1)
│   ├── 02_preprocessing.ipynb → cleaning & feature engineering
│   ├── 03_modeling.ipynb      → Random Forest → Logistic Reg → CART
│   └── 04_clustering.ipynb    → K-Means on intender subgroup (conditional)
│
├── outputs/
│   ├── figures/               → all charts saved as PNG
│   └── results/               → model outputs, feature importance CSVs
│
├── README.md                  → project write-up for GitHub
├── CLAUDE.md                  → this file
└── DECISIONS.md               → full analytical decision log
```

---

## Analytical Approach

### Step 1 — Exploratory Data Analysis (01_eda.ipynb)

**Business question to open the notebook with:**
"Who are the developers who intend to use AI tools but haven't
yet — and how do they compare to those who use AI tools daily
and those who have rejected AI entirely?"

**What to explore:**

1. Group size and proportions
   - Count and percentage of each AISelect group after NA removal
   - Bar chart: AISelect distribution (valid responses only)

2. Four-group comparison across key dimensions
   For each of the following features, produce a grouped bar chart
   or distribution plot comparing Active users / Casual users /
   Intenders / Rejectors:
   - AISent (overall AI sentiment)
   - AIAcc (trust in accuracy) — if available for non-users
   - AIThreat (job threat perception)
   - LearnCodeAI (active AI learning behavior)
   - AIFrustration (top frustrations — for groups that answered)
   - Age distribution
   - DevType (top 5 roles)
   - OrgSize
   - YearsCode / WorkExp distributions
   - ICorPM (IC vs. manager split)
   - Industry (top 5 industries)
   - PurchaseInfluence

3. Missingness check — CRITICAL before modeling
   For AIComplex and AIFrustration specifically:
   Calculate the % of intenders and rejectors who answered
   each question (non-null). If >40% missing for intenders,
   exclude from model and document the reason.

4. YearsCode vs. WorkExp correlation
   Compute Pearson r. If r > 0.85, drop WorkExp.
   Document the result in a markdown cell.

5. Key questions the EDA should answer:
   - Do intenders look more like active users or rejectors
     on sentiment and trust dimensions?
   - What frustrations do intenders report vs. rejectors?
   - Is there a role or experience level over-represented
     among intenders?
   - Does org size differ between intenders and rejectors?
   - Do intenders show higher LearnCodeAI rates than rejectors?

**EDA narrative goal:**
By the end of notebook 01, the reader should be able to say:
"Intenders share [X] with active users but differ on [Y]
— that tension is what the model will explain."
Surface at least 3 striking observations to set up modeling.

---

### Step 2 — Data Preprocessing (02_preprocessing.ipynb)

**Business question to open the notebook with:**
"How do we prepare the raw survey data so the model can
learn from it — and what choices do we make along the way?"

**Cleaning steps in order:**

1. Filter to valid AISelect responses only
   Drop all rows where AISelect is null (15,471 rows).
   Save as df_valid.csv.

2. Create group labels
   Map AISelect responses to four group labels:
   - "active" = "Yes, I use AI tools daily" OR "Yes, I use AI tools weekly"
   - "casual" = "Yes, I use AI tools monthly or infrequently"
   - "intender" = "No, but I plan to soon"
   - "rejector" = "No, and I don't plan to"

3. Filter to modeling subset
   Keep only intenders and rejectors for the primary model.
   Save as df_intenders_rejectors.csv.
   Preserve the full four-group df_valid for EDA reference.

4. Handle multi-select columns
   For DevType and AIFrustration (and any other
   semicolon-separated columns in the feature list):
   - Split on ";" to get individual values
   - Create one binary indicator column per unique value
   - Name format: DevType_BackendDev, DevType_FullStackDev, etc.
   - Drop the original semicolon column after explosion
   - Only keep indicator columns where at least 5% of rows
     have a value (drop very rare categories to avoid noise)

5. Encode ordinal columns
   Age: encode as ordered integers
   (18-24=1, 25-34=2, 35-44=3, 45-54=4, 55-64=5, 65+=6)
   EdLevel: encode as ordered integers from no formal education
   through graduate/professional degree
   Preserve the mapping in a markdown cell for reference.

6. Encode Employment
   Collapse to three categories first, then encode:
   - "employed_full_time" → 0
   - "independent_freelance" → 1
   - "student_other" → 2

7. Encode remaining single-select categoricals
   Use pd.get_dummies() with drop_first=True for nominal columns:
   Industry, OrgSize, ICorPM, RemoteWork, PurchaseInfluence
   For AISent, AIThreat, AIComplex (if included): encode as
   ordinal if the response options have a natural order,
   otherwise use get_dummies.

8. Handle remaining missing values
   After the above steps, check remaining null counts per column.
   For numeric columns (YearsCode, WorkExp): impute with median.
   For categorical columns: impute with mode OR add a
   "missing" indicator column — document which approach and why.
   Do not drop rows that have NAs in features (only in target).

9. Create binary target column
   intender_flag: 1 = intender, 0 = rejector

10. Train/test split
    80% train, 20% test, stratified on intender_flag.
    random_state=42 for reproducibility.
    Print class distribution in both splits to confirm
    stratification worked.

11. Save final feature list
    Write the column names of X_train to feature_names.txt
    in data/processed/. This is the authoritative feature list
    that the modeling notebook reads from.

---

### Step 3 — Modeling (03_modeling.ipynb)

**Business question to open the notebook with:**
"Among developers who are not currently using AI tools, what
most strongly predicts whether someone has intent to adopt —
and what does that tell a PM about where to intervene?"

**Model sequence: Random Forest → Logistic Regression → CART**
(See DECISIONS.md D-11 for full rationale)

#### 3a — Random Forest (empirical feature selection)

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import roc_auc_score

rf_model = RandomForestClassifier(
    n_estimators=200,
    max_depth=None,
    min_samples_leaf=10,
    class_weight='balanced',  # handles 25/75 class imbalance
    random_state=42
)
rf_model.fit(X_train, y_train)
```

Outputs to generate:
- Feature importance bar chart (top 20 features, horizontal bars)
  Save to outputs/figures/rf_feature_importance.png
  Use business-friendly feature names on axis — not raw column names
- AUC-ROC score on test set
- Print the top 10 most important features with their scores
- Identify features with importance < 0.01 as candidates for removal
  before Logistic Regression

#### 3b — Logistic Regression (direction and magnitude)

Trim the feature list: remove features with Random Forest
importance < 0.01. Document how many were removed in a markdown cell.

Run VIF check before fitting:
```python
from statsmodels.stats.outliers_influence import variance_inflation_factor
# Compute VIF for each feature in trimmed X_train
# Flag any feature with VIF > 5
# Drop the flagged feature with lower RF importance
# Document which features were dropped and why
```

```python
from sklearn.linear_model import LogisticRegression

lr_model = LogisticRegression(
    max_iter=1000,
    class_weight='balanced',
    random_state=42,
    solver='lbfgs'
)
lr_model.fit(X_train_trimmed, y_train)
```

Outputs to generate:
- Coefficient plot: horizontal bar chart of top 15 coefficients
  Positive = associated with intender, negative = associated with rejector
  Color positive bars one color, negative bars another
  Save to outputs/figures/lr_coefficients.png
  Use business-friendly labels — not raw column names
- AUC-ROC score on test set
- Confusion matrix with labeled axes
  Save to outputs/figures/lr_confusion_matrix.png
- Classification report (printed to cell output)
- Odds ratio table: exp(coefficients) for top 10 features
  Print as a clean DataFrame: Feature | Coefficient | Odds Ratio | Interpretation

#### 3c — CART Decision Tree (human-readable rules)

Use the trimmed feature list from Logistic Regression step.

```python
from sklearn.tree import DecisionTreeClassifier, plot_tree
import matplotlib.pyplot as plt

cart_model = DecisionTreeClassifier(
    max_depth=4,         # keep readable — no deeper than 4 levels
    min_samples_leaf=50, # avoid tiny leaves
    class_weight='balanced',
    random_state=42
)
cart_model.fit(X_train_trimmed, y_train)
```

Outputs to generate:
- Decision tree visualization (max_depth=4)
  Save to outputs/figures/cart_decision_tree.png
  figsize=(20, 10) minimum for readability
  Use feature_names and class_names=['Rejector', 'Intender']
- AUC-ROC score on test set
- Print the top 3 decision paths in plain English:
  "Path 1: If [feature A] is [value] AND [feature B] is [value]
  → [X]% probability of being an intender ([n] developers)"

#### 3d — Model comparison summary

Create a summary table comparing all three models:

| Model | AUC-ROC (test) | Key output |
|-------|---------------|------------|
| Random Forest | X.XX | Feature importance ranking |
| Logistic Regression | X.XX | Coefficients + odds ratios |
| CART | X.XX | Decision rules |

Save to outputs/results/model_comparison.csv

---

### Step 4 — Clustering (04_clustering.ipynb) — CONDITIONAL

**Only proceed with this notebook if:** Random Forest feature
importance shows NO single dominant feature (no feature with
importance > 0.25 that alone explains intender status).
If one feature dominates, clustering will just reproduce that
split. Document this decision at the top of the notebook.

**Business question to open the notebook with:**
"Among the 1,797 developers who intend to adopt AI tools,
are there meaningfully different sub-groups — and do they
suggest different product interventions?"

**Scope:** Cluster the intender group only using AI attitude
features (AISent, AIThreat, LearnCodeAI, and any others
confirmed in modeling).

```python
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import silhouette_score
from scipy.cluster.hierarchy import dendrogram, linkage

# 1. Standardize features before clustering
# 2. Run K-Means for k=2 through k=6
# 3. Plot silhouette scores to select optimal k
# 4. Validate with hierarchical clustering dendrogram
# 5. Profile each cluster: mean feature values per cluster
# 6. Name each cluster with a 2-3 word business label
#    e.g. "Trust-blocked intenders", "Org-blocked intenders"
```

Outputs to generate:
- Silhouette score plot (k=2 to 6)
  Save to outputs/figures/clustering_silhouette.png
- Cluster profile heatmap (features × clusters, mean values)
  Save to outputs/figures/cluster_profiles.png
- Hierarchical dendrogram for validation
  Save to outputs/figures/hierarchical_dendrogram.png
- Cluster size and business label summary table
  Save to outputs/results/intender_clusters.csv

**If clustering produces unclear or trivial segments:**
Write a markdown cell explaining why clustering did not add
value here, and what that implies (intenders are a relatively
homogeneous group blocked by a common mechanism — which
actually strengthens the main model's finding).

---

## Coding Standards

- Use **clear, descriptive variable names** — no single letters
  except in loops (i, j are fine in enumerate)
- Add a **markdown cell before every code cell** explaining
  what it does and why in plain English — not what the code does
  mechanically, but why this step matters for the analysis
- Add **inline comments** for any non-obvious logic
- Every notebook must start with a **business question**
  stated in plain English as the very first markdown cell
- Keep functions short and single-purpose with docstrings
- Avoid overly complex or clever code — prioritize readability
  and interpretability above all else
- **Do not use black-box approaches** without explanation —
  the author must be able to defend every decision in interviews
- All charts must have:
  - Clear, plain-English title (not variable names)
  - Labeled axes with units where relevant
  - Business-friendly tick labels (not raw encoded integers)
  - A one-sentence annotation or caption explaining the
    key takeaway from the chart
  - Consistent color scheme across all notebooks
    (suggest: intenders = #534AB7 purple, rejectors = #993C1D coral,
    active users = #1D9E75 teal, casual users = #BA7517 amber)
- Save every figure to outputs/figures/ before the notebook ends
- Save every result table to outputs/results/ as CSV

---

## Critical Preprocessing Reminders

These are the most likely places where errors or incorrect
assumptions will cause problems downstream. Read before coding.

1. **Never modify raw data files.** All cleaning happens in code.
   The raw CSVs in data/raw/ must remain untouched.

2. **Multi-select explosion creates many columns.**
   DevType alone may produce 15-20 indicator columns. After
   explosion, print the shape of the dataframe and a list of
   new column names before proceeding.

3. **AIComplex missingness check is mandatory.**
   Before including AIComplex in any model, print:
   - % of intenders who answered (non-null)
   - % of rejectors who answered (non-null)
   If either is below 60%, exclude from the model and
   add a markdown note explaining why.

4. **Stratified split is non-negotiable.**
   Always pass stratify=y to train_test_split.
   Print class counts in train and test after splitting
   to confirm the 25/75 ratio is preserved.

5. **class_weight='balanced' on all models.**
   The 25/75 intender/rejector split requires this in every
   sklearn classifier. Without it, the model will bias toward
   predicting "rejector" for everything.

6. **Business-friendly labels on all charts.**
   Never display raw column names (e.g., "DevType_BackendDev")
   on a chart axis. Create a label mapping dictionary at the
   top of each modeling notebook and apply it consistently.

7. **Save the final feature list.**
   After all preprocessing decisions are made, write the
   column names of X_train to data/processed/feature_names.txt.
   The modeling notebook should read from this file, not
   hard-code feature names.

---

## Success Criteria

Before moving to Phase 4 (Interpretation), every item below
must be checked off:

**EDA (01_eda.ipynb)**
- [ ] AISelect group sizes confirmed and match expected counts
- [ ] Missingness check completed for AIComplex and AIFrustration
- [ ] YearsCode / WorkExp correlation computed, decision documented
- [ ] At least 3 striking observations surfaced and written up
      in plain English in the notebook
- [ ] All charts saved to outputs/figures/

**Preprocessing (02_preprocessing.ipynb)**
- [ ] Raw data never modified
- [ ] Multi-select columns exploded correctly
- [ ] Ordinal encoding applied to Age and EdLevel
- [ ] Employment collapsed to 3 categories
- [ ] Stratified 80/20 split confirmed with class count print
- [ ] feature_names.txt written to data/processed/
- [ ] df_valid.csv and df_intenders_rejectors.csv saved

**Modeling (03_modeling.ipynb)**
- [ ] Random Forest run with class_weight='balanced'
- [ ] Feature importance chart saved and readable
- [ ] VIF check completed before Logistic Regression
- [ ] Features with RF importance < 0.01 removed and documented
- [ ] Logistic Regression coefficient chart saved
- [ ] Odds ratio table printed in notebook
- [ ] CART tree visualization saved (max_depth=4)
- [ ] Top 3 decision paths written in plain English
- [ ] All three AUC-ROC scores reported and compared
- [ ] Model comparison summary table saved to results/

**Overall**
- [ ] Each notebook reads top-to-bottom like a story —
      business question → analysis → insight
- [ ] No raw variable names visible on any chart
- [ ] Every model decision has a markdown explanation
- [ ] outputs/figures/ contains all charts needed for the deck
- [ ] outputs/results/ contains all tables needed for the deck

---

## Author Context

- MBA student (UW Foster, graduating June 2026), beginner in Python
- Background: 5 years pharma commercial analytics at ZS Associates
  (HCP segmentation, ML-based recommendation engines, sales incentive
  analytics, IQVIA data) + Merck Global Pipeline Analytics internship
  (GenAI platform evaluation for IBD patient journey modeling,
  build-vs-buy recommendation, HCP segmentation for Immunology launch)
- Code will be read line-by-line and defended in interviews —
  every decision must be explainable in plain English
- Business framing and interpretation matter more than technical
  sophistication — always
- When in doubt, choose the more interpretable approach over
  the more complex one
- Do not introduce techniques or libraries not listed in this
  file without flagging them with a markdown explanation of
  why they were needed

---

## Notes from Claude.ai Scoping Session

**Key decisions made:**
- Framing B selected: intender gap question (not binary adoption)
  Rationale: novel vs. published SO analysis, three-way comparison,
  PM-actionable output, personal narrative connection to Merck work
- Primary model: intenders vs. rejectors only (binary)
  Rationale: both are non-users, isolates psychological drivers
- Model sequence: RF → Logistic Regression → CART
  Rationale: RF handles messy features for empirical trimming,
  LR gives interpretable coefficients, CART gives PM-readable rules
- Evaluation metric: AUC-ROC (not accuracy) due to 25/75 class split

**Approaches considered but not chosen:**
- Binary adoption prediction (user vs. non-user): collapses
  intenders and rejectors, loses the most actionable insight
- Logistic Regression first with VIF trimming: valid but cumbersome
  with 35-40 features after multi-select explosion; VIF retained
  as secondary validation within the LR step instead
- Combining casual users with intenders: rejected because they
  represent different funnel stages (post- vs. pre-activation)

**Interview questions to prepare for:**
- "Why not just use Stack Overflow's own published analysis?"
- "The intender group is only 1,797 people — isn't that too small?"
- "Why did you run Random Forest before Logistic Regression?"
- "Why AUC-ROC and not accuracy?"
- "What would you do differently with more time or data?"
- "Isn't this just describing the data, not causing anything?"
See DECISIONS.md for complete prepared answers to all of these.

**Course concepts applied (Prof. Wagner, UW Foster):**
- Logistic Regression → 03_modeling.ipynb, Logistic Regression step
- CART (Classification Trees) → 03_modeling.ipynb, CART step
- Random Forests → 03_modeling.ipynb, Random Forest step
- K-Means Clustering → 04_clustering.ipynb (conditional)
- Hierarchical Clustering → 04_clustering.ipynb, validation step
- Train/test split and model evaluation → covered in Wagner's
  Linear and Logistic Regression slides

**Full decision log:** See DECISIONS.md in repo root.
Every analytical decision from dataset selection through
output design is documented with reasoning and interview answers.
