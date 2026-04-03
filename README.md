# The Intender Gap: What Stops Developers from Adopting AI Tools?

> A predictive analytics project using the Stack Overflow 2025 Developer Survey
> to identify what separates developers who **intend** to adopt AI tools from
> those who actually do — and what product interventions close the gap.

---

## The Problem

84% of developers now use or plan to use AI tools. But 5.3% — roughly 1,800
developers in this survey alone — sit in an unusual middle ground: they have
**declared intent to adopt but haven't acted.**

Stack Overflow's own published analysis shows this group as a single percentage
and moves on. This project asks the question they don't answer:

**Who are these developers, what is blocking them, and what should a PM at
GitHub Copilot, Cursor, or any AI developer tools company do about it?**

---

## Why This Matters

Companies building AI developer tools face a concrete growth problem. Marketing
to developers who have already rejected AI tools is expensive and unlikely to
work. Marketing to developers who are already daily users is unnecessary.

The intender — the developer who says "I plan to use AI soon" but hasn't crossed
over — is the highest-value, most tractable growth segment. Understanding what
blocks them has direct implications for onboarding design, product messaging,
and where to invest engineering resources.

This project builds the targeting engine a PM would actually use, not just
better bar charts of the same statistics Stack Overflow already publishes.

---

## The Approach

Rather than treating this as a binary adoption question (user vs. non-user),
this project isolates **intenders vs. rejectors** — two groups who are both
non-users, holding access and habit constant. The only meaningful difference
between them is intent. That clean comparison isolates the psychological and
attitudinal drivers of adoption without confounding them with usage history.

**Three-act analytical structure:**

| Act | Question | Methods |
|-----|----------|---------|
| 1 — EDA | Who are the four groups and how do they differ? | Descriptive analysis, group comparisons |
| 2 — Prediction | What most strongly predicts intender vs. rejector status? | Random Forest → Logistic Regression → CART |
| 3 — Segmentation | Are all intenders blocked for the same reason? | K-Means clustering on intender subgroup |

---

## Dataset

**Source:** [Stack Overflow 2025 Annual Developer Survey](https://survey.stackoverflow.co/)
— 49,196 respondents, 177 countries. Free public download.

**Working sample:** 33,720 valid responses after removing 15,471 rows where
the target variable (`AISelect`) was blank. Missingness confirmed as random
via dropout-curve analysis — see `DECISIONS.md` D-07 for full evidence.

**Four groups defined from `AISelect`:**

| Group | n | % | Strategic role |
|-------|---|---|----------------|
| Active users (daily + weekly) | 21,841 | 64.8% | Reference — what success looks like |
| Casual users (monthly + infrequently) | 4,628 | 13.7% | Habit formation problem |
| **Intenders** (plan to soon) | **1,797** | **5.3%** | **Primary target — this project** |
| Rejectors (don't plan to) | 5,454 | 16.2% | Comparison group |

**Primary model:** Binary classification — intenders (1) vs. rejectors (0).
7,251 rows, approximately 25/75 class split.

---

## Key Findings

### Finding 1 — The barrier is attitudinal, not demographic

43 of 60 features had near-zero predictive importance. Organization size,
industry, developer role, years of experience, and company type do not
meaningfully separate intenders from rejectors.

What does separate them: **how they feel about AI and how they behave toward it.**

This rules out expensive demographic-targeting strategies. A PM cannot build
their way to solving the intender gap by adding enterprise features or
healthcare-specific compliance — the gap exists uniformly across all developer
profiles.

### Finding 2 — Intenders are prepared, not unaware

Intenders actively learn AI tooling at **3x the rate of rejectors** (40.3% vs
13.3%). They are not passive or uninformed. The adoption gap is not an
awareness problem — it is an **activation problem.** Something happens between
"I've been learning about AI tools" and "I use them every day" that stalls the
conversion.

**PM implication:** Don't spend budget on top-of-funnel awareness campaigns
for intenders. They already know. Invest in reducing friction at the moment of
first committed use.

### Finding 3 — Sentiment is the dominant predictor

AI Sentiment: Very Unfavorable is the single most important feature at 0.32
Random Forest importance — accounting for nearly a third of total predictive
power. A developer with very unfavorable sentiment toward AI is **24x more
likely to be a rejector than an intender** (odds ratio: 0.041).

Critically, intenders are not characterized by very *favorable* sentiment —
they cluster in "Favorable" (30%) and "Indifferent" (34%). Their problem is
not that something is actively repelling them from AI. **Nothing is strongly
enough pulling them forward.**

The paradox: sentiment shapes adoption *before* direct experience. Intenders
form cautious optimism without committing, while rejectors form strong negative
views and close the door.

---

## Model Performance

| Model | AUC-ROC | Key output |
|-------|---------|------------|
| Random Forest | 0.8543 | Feature importance ranking — identified 17 meaningful features from 60 |
| Logistic Regression | 0.8537 | Coefficients and odds ratios — direction and magnitude of each feature |
| CART Decision Tree | 0.8332 | Human-readable decision rules — three developer personas |

All three models exceed the 0.8 "good" threshold. Logistic Regression AUC
essentially matches Random Forest after trimming 43 near-zero features —
confirming the trimmed 17-feature set preserved all meaningful signal.

---

## Three Developer Personas (CART Decision Tree)

**Path 1 — The Hesitant Intender** (n=948, 61.5% intender probability)
Not hostile toward AI. Not actively learning it. No strong opinion on AI
complexity. Blocked by inertia, not hostility. PM intervention: create the
pull — a specific, low-friction first use trigger that removes the
"I'll get to it eventually" excuse.

**Path 2 — The Firm Young Rejector** (n=763, 97.3% rejector probability)
Very unfavorable sentiment, young (18-34), not learning AI, no job threat
concern. Ideological rejection with no external pressure to change. Do not
spend acquisition resources here — 97.3% won't cross over.

**Path 3 — The Experienced Rejector** (n=698, 92.6% rejector probability)
Very unfavorable sentiment, 35+, 19+ years coding experience. Deep-rooted
skepticism backed by long experience with technology hype cycles. Hard to move.

---

## Three Intender Sub-Personas (K-Means Clustering, k=3)

The predictive model correctly identified intenders as a group but masked
internal differences. Clustering on attitude variables within intenders only
revealed three distinct sub-segments requiring different interventions:

**Cluster 0 — Never-Started Intenders** (n=635, 44.2% of intenders)
93% have never used AI tools and have no formed opinion on capability. Blocked
by pure inertia — no trial, no friction, no direct experience. PM intervention:
zero-config frictionless trial. One-click install, instant output, no setup.
If activation costs anything above "open it and type," they stay in intent mode.

**Cluster 1 — Engaged Fence-Sitters** (n=649, 45.2% of intenders)
Highest active-learning rate (54%), most positive sentiment. Have tried AI,
see value, but haven't had a breakthrough moment. Closest to crossing over.
PM intervention: workflow-specific onboarding with a forced first win. AI for
code review, documentation, test scaffolding — tailored to how they actually
work, not a generic demo.

**Cluster 2 — Quality-Disappointed Intenders** (n=153, 10.6% of intenders)
100% rated AI complexity as "Very Poor." Frustrated by "almost right but not
quite" outputs (52%) and debugging AI mistakes (50%). Still actively learning
despite disappointment — genuine intent to find a version of AI that works.
PM intervention: accuracy transparency, confidence scores on outputs, use-case
steering toward where AI is most reliable. Converted members of this cluster
become credible product champions — their earned skepticism makes their
endorsement more persuasive than a first-time adopter's.

**Key insight:** Clusters 0 and 1 require opposite interventions. Cluster 0
needs simplicity — remove every barrier to a first interaction. Cluster 1
needs depth — provide a specific, meaningful use case. A single generic
onboarding flow underserves both.

---

## PM Recommendations

Based on the full three-act analysis:

**1. Stop targeting rejectors for conversion.**
Path 2 and Path 3 show 92-97% rejector probability. Their barriers are
attitudinal and deeply held. Redirect that acquisition budget to intenders —
specifically Cluster 1 Engaged Fence-Sitters, who have the highest ROI for
a targeted activation nudge.

**2. Build two distinct onboarding journeys, not one.**
Never-Started Intenders (Cluster 0) need a frictionless zero-config entry
point. Engaged Fence-Sitters (Cluster 1) need a personalized first-win
experience. Detect which segment a new user belongs to in the first session
and route them accordingly.

**3. Lead with accuracy, not productivity.**
The most common intender frustration is "almost right but not quite" — not
"it's too slow" or "it costs too much." Onboarding messaging that leads with
productivity claims misses the actual barrier. Show confidence scores. Be
transparent about where the tool is reliable and where it isn't. That honesty
builds the trust that converts intenders.

**4. Invest in developer advocacy targeting Cluster 2.**
A Quality-Disappointed developer who converts becomes your most credible
voice. Their testimonial — "I tried it, it failed me, I gave it another
chance, and here's specifically what changed" — is more persuasive than any
first-time adopter's endorsement. Identify and invest in this segment
disproportionately.

---

## Personal Context

I am a second-year MBA student at the University of Washington Foster School
of Business with five years of pharma commercial analytics experience, currently
looking to pivot into AI Product Management roles. This project is part of that
transition — built deliberately to demonstrate product thinking, not just
technical execution.

The project came from a question I kept encountering while exploring AI PM
opportunities: who is the right user to target, and what actually moves them?
Generic adoption statistics — "84% of developers use or plan to use AI" — don't
answer that. They describe the population. They don't tell a PM where to invest.

I wanted to build something that answered the product version of the question:
among developers who aren't yet using AI tools, which ones are worth pursuing,
what is specifically blocking them, and what intervention has the highest
return? The Stack Overflow 2025 survey gave me 49,000 real developer responses
to work with. The result is an analysis I can walk into an AI PM interview and
defend — not as a classroom exercise but as a product decision framework built
on real data.

The finding that resonated most: the intender gap is not a demographics
problem. It is an attitude and trust problem. That reframes how a PM should
think about growing AI tool adoption — less segmentation by industry or role,
more investment in onboarding experiences that build trust at the moment of
first use. That is the kind of insight I want to bring to an AI PM role.

---

## Repo Structure

```
developer-ai-adoption-gap/
│
├── data/
│   ├── raw/                          ← survey CSVs (not in repo — see below)
│   └── processed/                    ← train/test splits, feature list
│
├── notebooks/
│   ├── 01_eda.ipynb                  ← Act 1: who are the four groups?
│   ├── 02_preprocessing.ipynb        ← cleaning, encoding, train/test split
│   ├── 03_modeling.ipynb             ← Act 2: RF → Logistic Reg → CART
│   └── 04_clustering.ipynb           ← Act 3: intender sub-personas
│
├── outputs/
│   ├── figures/                      ← all charts referenced above
│   └── results/                      ← model outputs, cluster profiles
│
├── CLAUDE.md                         ← analytical build brief
├── DECISIONS.md                      ← full decision log with interview Q&A
└── README.md                         ← this file
```

**To reproduce this analysis:**
1. Download the 2025 Stack Overflow Developer Survey from
   [survey.stackoverflow.co](https://survey.stackoverflow.co/)
2. Place `survey_results_public.csv` and `survey_results_schema.csv`
   in `data/raw/`
3. Run notebooks in order: 01 → 02 → 03 → 04

---

## What I'd Do Differently

Two things with more time:

**Look more closely at casual users.** This project focused on intenders vs.
rejectors. The casual user — someone who uses AI monthly but hasn't made it a
daily habit — faces a different problem: habit formation rather than initial
activation. Understanding what separates a casual user from a daily user would
complete the full adoption lifecycle picture and is a natural extension of this
analysis.

**Collect a targeted follow-up survey.** The 1,797 intenders in this dataset
answered general survey questions — not questions designed specifically to
probe activation barriers. A short follow-up survey targeted at developers who
identify as intenders ("what has stopped you from using AI tools in the past
30 days specifically?") would produce far richer signal than what's available
in the general survey data.

---

## Tools and Methods

- **Python 3.13** — pandas, numpy, matplotlib, seaborn, scikit-learn
- **Models:** RandomForestClassifier, LogisticRegression,
  DecisionTreeClassifier, KMeans
- **Evaluation:** AUC-ROC (primary), confusion matrix (secondary)
- **Feature selection:** Random Forest importance → VIF validation
- **Cluster selection:** Silhouette score with minimum cluster size constraint

---

*Built as part of a data science portfolio for AI PM and Product Analytics
roles. UW Foster School of Business MBA, graduating June 2026.*
