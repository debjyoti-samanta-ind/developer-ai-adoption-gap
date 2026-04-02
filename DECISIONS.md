# DECISIONS.md
## Project Decision Log — AI Tool Adoption Analysis
### Stack Overflow 2025 Developer Survey
### UW Foster MBA Portfolio Project | Debjyoti Samanta | June 2026

---

> **How to use this file:**
> Every significant analytical decision made during Phase 1 and Phase 2
> is recorded here with full reasoning. This is your interview prep
> document — every entry answers a "why did you do X?" question.
> Each entry has three parts: what we decided, why we chose it over
> alternatives, and what limitation or nuance to acknowledge.
> Never delete old entries. Add new ones as the project progresses.

---

## Quick Reference — Decision Index

| ID | Decision | Phase |
|----|----------|-------|
| D-01 | Dataset selection | 1 |
| D-02 | Business question selection | 1 |
| D-03 | Project story structure (three acts) | 1 |
| D-04 | Target variable selection | 1 |
| D-05 | Exclusion of RO columns | 1 |
| D-06 | Exclusion of TE columns | 1 |
| D-07 | NA / missing value treatment | 1 |
| D-08 | Target variable group definitions | 2 |
| D-09 | Feature selection — AI attitude columns | 2 |
| D-10 | Feature selection — developer profile columns | 2 |
| D-11 | Model sequencing and evaluation metric | 2 |
| D-12 | Output design — what a PM walks away with | 2 |
| D-13 | Final feature confirmations from EDA | 2 |
| D-14 | Modeling results and key findings | 2 |
| D-15 | Clustering results — three intender personas | 3 |

---

## Phase 1 — Problem Definition

---

### D-01 | Dataset Selection

**Decision:** Use the Stack Overflow 2025 Annual Developer Survey.

**Why this dataset:**
- 100% real survey data — 49,000+ actual developer responses from
  177 countries. Not synthetic, not scraped, not fabricated.
  Fully defensible in any interview.
- Published July 2025 by Stack Overflow directly.
  Free download at survey.stackoverflow.co
- Rich AI-specific section: 30+ columns covering AI usage, sentiment,
  trust, frustrations, agent adoption, and workflow integration —
  exactly what our business question needs.
- Timely: the 2025 data captures the trust collapse narrative
  (adoption up to 84%, trust down to 29%) which is more
  interview-relevant than 2024 data.

**Why not 2024:**
- 2025 is newer and more relevant for interviews in 2026.
- 2025 has an expanded AI section (15 new questions vs. 2024).
- The adoption-trust gap story is sharper in 2025.
- 2024 has a slightly larger sample (65K vs. 49K) but this
  difference is immaterial at these sample sizes.

**Why not other datasets considered:**
- Telco Churn (IBM): synthetic data — cannot defend "real user
  behavior" claim in an interview.
- Spotify behavior (Kaggle): most versions are synthetic fan projects.
- Wikipedia Clickstream: most original but hardest to scope and
  execute in 2-3 weeks.
- Yelp Open Dataset: real data but AI angle is forced and unnatural.

---

### D-02 | Business Question Selection

**Decision:** Framing B — "What separates developers who intend to
use AI tools from those who actually do — and what is blocking the
intender group from crossing over?"

**Five options considered and ruled out:**

*Option 1 — "What predicts AI tool adoption?" (binary):*
Collapses intenders and rejectors into one group. A developer who
says "I plan to use AI soon" is fundamentally different from one
who says "I have no plans." Lumping them loses the most actionable
insight. Also: this exact analysis exists all over Kaggle.
Not differentiated.

*Option 2 — "What predicts trust in AI tools?" (AIAcc as target):*
Predicts an attitude, not a behavior. Weaker business hook.
AIAcc is also one of our strongest potential features — using it
as the target removes it from the feature set entirely.

*Option 3 — "What makes an AI tool sticky?" (retention):*
Interesting but the 2025 schema is less clean for this question.
Applies only to existing users — filtered population reduces
sample size and generalizability.

*Option 4 — "What frustrations are most associated with
non-adoption?" (pure EDA):*
Good descriptive question but produces only better bar charts,
not a predictive model. Incorporated as Act 1 (EDA) setup
rather than the core question.

*Option 5 — "Can we cluster developers into AI personas?"
(clustering):*
Exploratory and original but harder to frame as a product
decision. Retained as Act 3 (conditional) of the three-act
story structure.

**Why Framing B won:**
1. Behaviorally precise — "No, but I plan to soon" is a clean,
   observable signal of intent without action.
2. Novel — Stack Overflow's own analysis shows the intender
   percentage as a single number and moves on. No published
   analysis has modeled what distinguishes them.
3. Natural three-way comparison: active users / intenders /
   rejectors.
4. Business recommendation writes itself — identify the strongest
   barrier, tell the PM what intervention to prioritize.
5. Personal narrative connection: at Merck, the GenAI platform
   evaluation was essentially the same decision at org level —
   to cross from "intending to adopt" to "actively using."
   This project does it at population scale across 49K developers.

**Limitation to acknowledge:**
The intender group is small (~1,797 respondents after removing NAs).
Statistical power is sufficient for modeling but limits fine-grained
subgroup analysis. Mitigation: use stratified train/test split.
"What I'd do differently": collect a targeted follow-up survey
focused specifically on this group.

---

### D-03 | Project Story Structure (Three Acts)

**Decision:** Build the project as a three-act analytical narrative,
not a single model exercise.

**Act 1 — Who are the three groups? (EDA)**
Characterize active users, intenders, and rejectors across key
dimensions. Compare AIFrustration distributions, sentiment, trust
levels, role, experience, and company size. Surface the 3 most
striking differences between groups. Sets up the "why" before the
model runs.
Deliverable: 01_eda.ipynb

**Act 2 — What predicts which group you're in? (Predictive Modeling)**
Logistic Regression + CART + Random Forest. Feature importance
ranking. Translate top findings into PM-ready language.
Deliverable: 02_preprocessing.ipynb + 03_modeling.ipynb

**Act 3 — Are all intenders the same? (Clustering — CONDITIONAL)**
K-Means + Hierarchical Clustering on the intender subgroup only.
Condition for inclusion: only proceed if Act 2 shows intenders
are NOT explained by a single dominant factor. If one variable
explains everything, clustering adds nothing and is cut.
If it works: produces distinct intender personas blocked for
different reasons, each requiring different PM interventions.
Deliverable: 04_clustering.ipynb (if included)

**Course concepts covered:**
- Logistic Regression (Act 2 — primary model)
- CART / Decision Trees (Act 2 — human-readable rules)
- Random Forest (Act 2 — feature importance)
- K-Means Clustering (Act 3 — intender sub-segmentation)
- Hierarchical Clustering (Act 3 — validating k selection)

Note: Linear Regression deliberately excluded — forcing it in
would be "technique for the sake of it," which undermines the
project's analytical integrity.

---

### D-04 | Target Variable Selection

**Decision:** AISelect is the target variable (the thing we predict).

**What AISelect measures:**
"Do you currently use AI tools in your development process?"
Five responses: daily / weekly / monthly or infrequently /
plan to soon / don't plan to.

**Why AISelect and not other columns:**
- Only column capturing current behavioral status toward AI adoption
  (as opposed to attitudes or perceptions).
- Enables the three-group segmentation that Framing B requires.
- Clean, observable outcome — not an opinion or a prediction.
- All other AI columns (AISent, AIAcc, AIThreat, etc.) measure
  attitudes and perceptions. These are the FEATURES — the
  potential explanations. AISelect is the BEHAVIOR they explain.
  That asymmetry is fundamental.

**What the model is and is not:**
This is an association model, not a causal model. We identify which
characteristics are most strongly associated with being an intender
vs. active user vs. rejector. We are NOT claiming that changing one
variable will cause someone to adopt AI tools. Causation requires
experimental design (e.g., A/B test). Our findings generate
hypotheses for PMs to test, not proven interventions.

---

### D-05 | Exclusion of RO (Rank Order) Columns

**Decision:** Exclude all RO-type columns from the model.
(TechEndorse_*, TechOppose_*, JobSatPoints_*, SO_Actions_*)

**Primary reason — low relevance:**
RO columns measure general work preferences and technology
endorsement philosophy. They are not specific to AI adoption
behavior. A feature that doesn't vary meaningfully across the
three AISelect groups won't help the model discriminate between
them. Including them adds noise without signal.

**Secondary reason — wrong question:**
These columns answer "what do you value in technology generally?"
not "what is blocking you from adopting AI tools specifically?"
Features must be theoretically motivated — each one should have
a plausible reason to affect AI adoption status.

**Validation plan:**
During EDA, confirm that RO distributions do not differ
meaningfully across the three AISelect groups.
If they do — revisit this decision.

---

### D-06 | Exclusion of TE (Text Entry) Columns

**Decision:** Exclude all TE-type free-text columns from the model.
(AIOpen, AIExplain, AIAgentKnowWrite, etc.)

**Primary reason — technical feasibility:**
Free-text columns require NLP preprocessing (tokenization, TF-IDF
or embeddings) to convert to numeric features. This is a separate
technical discipline, adds significant complexity, and is out of
scope for a 2-3 week beginner Python project.

**Secondary reason — weaker signal:**
Open-ended responses are noisier than structured responses for
predicting a categorical outcome. The structured MC columns
(AISent, AIAcc, AIComplex, AIFrustration) capture the underlying
sentiments more cleanly.

**"What I'd do with more time":**
Sentiment analysis on AIOpen ("what skills will remain valuable
in 3-5 years?") could add a qualitative dimension to the feature
set — surfacing whether intenders express different future-skill
anxieties than active users or rejectors.

---

### D-07 | NA / Missing Value Treatment for AISelect

**Decision:** Drop all 15,471 rows where AISelect is NA.
Treat as Missing Completely at Random (MCAR).
Work with 33,720 valid responses.

**Evidence supporting MCAR — this was verified, not assumed:**

*Finding 1 — Dropout curve confirms survey fatigue, not avoidance:*
Completion rate through the NA group drops sharply mid-survey:
- Early demographics (MainBranch, Age): 100% completion
- Mid-survey (OrgSize, RemoteWork): ~45-47% completion
- Industry / JobSat: drops to ~11-26% — clear dropout cliff
- AISelect and all subsequent AI columns: ~0% completion
93.6% of the NA group dropped out before even reaching the
Technology section, let alone the AI section.
They were never asked AISelect.

*Finding 2 — Demographic profile of NA group is normal:*
Age distribution: 25-34 (36%), 35-44 (24%), 18-24 (24%) —
typical developer population.
MainBranch: 74% professional developers — also typical.
No evidence of systematic bias toward AI-avoiders.

*Finding 3 — AI columns after AISelect are also blank:*
AISent, AIAcc, AIComplex completion rates: 0.2-0.3% in NA group.
Confirms these respondents never reached the AI section.

**One nuance to acknowledge in writeup:**
2,952 respondents (19% of NA group) answered AIThreat but dropped
before AISelect — a small subset partially completed the AI section.
Still dropout behavior, not avoidance.
Limitations statement: "A small subset of NA respondents partially
completed the AI section. Given the overall dropout pattern and
normal demographic profile, we treat missing values as MCAR
and exclude them."

**Why this matters for credibility:**
Most analysts drop NAs without investigation. We investigated and
confirmed the assumption with data. That is a meaningful
differentiator in an interview.

---

## Phase 2 — Approach & Planning

---

### D-08 | Target Variable Group Definitions

**Decision:** Four behaviorally distinct groups derived from AISelect.

| Group | AISelect responses | n | Strategic problem |
|-------|-------------------|---|-------------------|
| Active users | Daily + Weekly | 21,841 | Retained — what does success look like? |
| Casual users | Monthly + Infrequently | 4,628 | Habit formation gap |
| Intenders | No, but I plan to soon | 1,797 | Activation gap — intent without action |
| Rejectors | No, and I don't plan to | 5,454 | Deliberate non-adoption |

**Primary model (Act 2):** Intenders vs. Rejectors only.
The sharpest, cleanest version of Framing B. Among developers
who are not currently using AI tools, what separates those with
intent from those without? Both groups are non-users, so tool
access and company context are held roughly constant. The only
meaningful difference is intent — isolating the psychological
and attitudinal drivers cleanly.

**Secondary use of four groups:**
All four groups appear in EDA (Act 1) for full characterization.
Casual users serve as an interesting reference point — if intenders
and casual users share similar frustration profiles, it suggests
the activation gap and habit formation gap have common root causes,
which has implications for onboarding design.

**Why not combine monthly/infrequently with "plan to soon":**
"Monthly or infrequently" users have already crossed the activation
threshold — they have experience with AI tools.
"Plan to soon" have not — they have never committed to first use.
These represent different stages of the product funnel:
- Casual users: habit formation problem (post-activation)
- Intenders: activation problem (pre-activation)
A PM would design completely different interventions for each.
Combining them for statistical convenience would obscure the
core insight.

**The population size trap — explicitly avoided:**
The intender group (n=1,797) is small relative to other groups.
This is not a reason to expand its definition. 1,797 observations
is statistically sufficient for logistic regression and CART.
The group is kept narrow to preserve conceptual clarity.
Stratified train/test split handles class imbalance.

**Interview answer if challenged on small intender count:**
"I deliberately kept the intender group separate despite its
smaller size because they represent a fundamentally different
stage in the adoption funnel — pre-activation versus post-
activation. Combining them for statistical convenience would
have obscured the very insight I was trying to find."

---

### D-09 | Feature Selection — AI Attitude Columns

**Confirmed includes:**

`AISent` — Overall stance toward AI tools. Asked of ALL respondents
regardless of AISelect answer. Coverage: 99.1% of valid sample
(~1,780 of 1,797 intenders expected to have answered). Not
circular — sentiment predicts behavior, that's the point. The
intender who says "Very favorable" but hasn't adopted is the
core paradox of Framing B. Missingness was verified before
confirming inclusion — concern was methodologically valid,
data resolved it.

`AIThreat` — Threat perception. Universal question. Mechanism:
threat as urgency driver. Test both directions in EDA — high
threat may increase OR decrease intent depending on whether it
motivates or paralyzes.

`AIFrustration` — Perceived barriers. Universal question. Covers
secondhand frustrations reported by non-users. Conditional
include: check missingness for intenders in EDA.
If >40% missing, exclude.

`LearnCodeAI` — Active learning investment in AI programming.
Universal question. High learning investment + no adoption =
interesting intender profile (prepared but not yet activated).
Strong Gate 1 story.

**Conditional includes — verify missingness for intenders in EDA:**

`AIComplex` — "How well do AI tools handle complex tasks?"
Likely a conditional question routed only to current users.
If intenders were skipped, near-100% NA expected for intender
group — must be excluded from model.
Check missingness by AISelect group before confirming.

**Excluded:**

`AIHuman` — Future-oriented hypothetical. Indirect mechanism.
Multi-select complexity not justified by expected predictive
value. Keep in EDA as descriptive context only.

`AIAgents`, `AIAgentChange` — Target leakage. Measure depth of
AI engagement downstream of adoption. Consequences of adoption,
not predictors of it.

`AIModelsChoice` — Target leakage. Requires active usage to
answer meaningfully.

**Key methodological insight:**
SO 2025 questions are not all universal — some are conditional
(routed only to subgroups). Features conditionally asked of
current AI users will have high NA for intenders and rejectors.
Must check missingness by AISelect group during EDA before
finalizing feature list.

**Interview answer on AISent inclusion:**
"I verified that 99.1% of valid respondents answered AISent
regardless of their AISelect response — it's a universal question.
The concern about circularity is real but misplaced: sentiment
predicting behavior is not circular, it's the hypothesis. The
interesting finding would be intenders with positive sentiment
who still haven't adopted — that tension is exactly what we're
trying to explain."

---

### D-10 | Feature Selection — Developer Profile Columns

**Included:**

`Age` — Ordinal ranges. Adoption curve dynamics — younger
developers have lower switching costs. Encode as ordinal.

`EdLevel` — Ordinal. CS education may correlate with AI skepticism
or AI literacy — either direction is interesting. Encode as ordinal.

`DevType` — Multi-select. Role shapes opportunity and threat
perception. Requires binary indicator explosion in preprocessing.

`Industry` — Single-select. Regulated industries face compliance
constraints creating organizational barriers independent of
individual intent.

`Employment` — Single-select. Collapse to three categories:
Employed full-time / Independent or Freelance / Student or other.
Incentive structures and organizational access differ meaningfully.

`YearsCode` and `WorkExp` — Both retained into EDA. Compute
Pearson correlation on valid dataset. If r > 0.85, drop WorkExp
and keep YearsCode. Decision deferred to EDA — do not prejudge.

`OrgSize` — Single-select. Large organizations have security
policies and IT approval processes that block adoption even when
individual intent exists.

`ICorPM` — Single-select. Individual contributor vs. people
manager. Managers have different incentive structures. Actionable:
PM can tailor enterprise messaging by persona type.
Note: initially considered for exclusion on the grounds that
"a PM can't change whether someone is a manager" — corrected.
Actionability means "can a PM act on this finding," not "can
a PM change this attribute about the user."

`RemoteWork` — Borderline include. Mechanism: remote developers
have weaker peer influence channels. Cut if no variation in EDA.

`PurchaseInfluence` — Developers with no purchase influence may
have intent but face organizational barriers they cannot
personally overcome. Informs top-down vs. bottom-up PM strategy.

**Excluded:**

`MainBranch` — Sampling artifact. Near-zero variance in valid
sample. Using it would be like using survey participation
as a predictor.

`Country` — Fails actionability gate. Proxies for OrgSize,
Industry, and regulatory environment — all measured more
directly by other features.

**Three-gate framework applied to all features:**
1. Theoretical motivation — is there a plausible causal story?
2. Variation across groups — does this differ meaningfully
   between intenders, rejectors, active users, casual users?
3. Actionability — can a PM do something different Monday
   morning because of this finding?
A feature must pass all three gates to be included.

**Preprocessing flags for Claude Code (Phase 3):**
- `DevType` and `AIFrustration`: multi-select → binary indicator
  explosion required
- `Age` and `EdLevel`: encode as ordinal, not nominal
- `Employment`: collapse small categories before encoding
- `YearsCode` / `WorkExp`: compute correlation, drop one if r > 0.85
- `RemoteWork` and `AIComplex`: check variation across AISelect
  groups in EDA before confirming inclusion

---

### D-11 | Model Sequencing and Evaluation Metric

**Model sequence: Random Forest → Logistic Regression → CART**

*Step 1 — Random Forest (all confirmed features)*
Run first. Handles correlated features and multi-select binary
indicator explosion (35-40 columns) naturally. Produces empirical
feature importance rankings. Use to trim feature list before
Logistic Regression. Also captures non-linear relationships —
importance scores reflect true predictive value more fairly than
raw Logistic Regression coefficients on an untrimmed set.

*Step 2 — Logistic Regression (trimmed feature list)*
Run second on features confirmed important by Random Forest.
Produces direction, magnitude, and statistical significance for
each feature — stable because noisy/correlated features removed.
Key output: odds ratios in plain English for PM audience.
VIF check (threshold: VIF > 5) run as secondary validation for
residual multicollinearity. P-values used to confirm statistical
significance of retained features.

*Step 3 — CART (communication layer)*
Run last. Translates findings into human-readable decision rules
a PM can act on without statistical training. Does not add new
statistical insight — bridges analysis to business recommendation.
Key output: decision tree with 3-4 most important splits and
predicted group membership at each leaf.

**Alternative approach considered and partially incorporated:**
Classical: run Logistic Regression directly, use VIF to detect
multicollinearity, trim by p-values, iterate. Valid approach.
Not used as primary method because: (1) multi-select explosion
makes iterative VIF trimming cumbersome; (2) Random Forest
handles non-linear relationships more fairly; (3) two-step
sequence produces a cleaner portfolio narrative.
VIF retained as secondary validation within Step 2.

**Evaluation metric: AUC-ROC (primary) + Confusion Matrix
(secondary)**

*Why AUC-ROC and not Accuracy:*
Class split is approximately 25/75 (1,797 intenders vs. 5,454
rejectors). A model predicting "rejector" for everyone achieves
75% accuracy while learning nothing. Accuracy is misleading
for imbalanced classes. AUC-ROC measures discrimination ability
across all thresholds — immune to class imbalance.
Target: 0.5 = random, 0.7+ = useful, 0.8+ = good.

*Why Confusion Matrix as secondary:*
Gives practical precision for PM use: "of the developers the
model flags as intenders, what percentage actually are?"

**Train/test split: 80/20, stratified**
Preserves 25/75 class ratio in both train and test sets.
Without stratification, test set may have too few intenders
to evaluate model performance properly.

---

### D-12 | Output Design — What a PM Walks Away With

**Three PM-ready outputs. Structure for each:
Finding → So what → Product recommendation**

*Output 1 — Segment definition (Who to target)*
Profile of the highest-conversion intender — the developer most
likely to convert given the right nudge. Translates to: which
acquisition channels to prioritize, which message to lead with,
which product tier or trial to offer.
Example: "Target mid-career developers at large organizations
with favorable AI sentiment who haven't yet adopted — they are
your highest-conversion intender profile."

*Output 2 — Barrier identification (What to fix)*
The single strongest predictor of being a rejector rather than
an intender. Translates directly into a product or messaging
intervention.
Examples by barrier type:
- Trust in accuracy → lead onboarding with accuracy
  demonstrations before asking users to commit
- Organizational size → build enterprise compliance features,
  SSO, admin controls, procurement-friendly pricing
- Threat perception → reframe from "AI replaces you" to
  "AI amplifies you"

*Output 3 — Funnel stage diagnosis (Where to intervene)*
Where in the adoption journey the conversion gap opens —
awareness, trial, activation, or organizational approval.
Tells PM which funnel stage to invest in. High marketing
spend is wasteful if the gap is at activation, not awareness.
Key behavioral features (LearnCodeAI, PurchaseInfluence)
distinguish between funnel stages.

**Format rules for PM audience:**
- One headline finding per output (one sentence, no jargon)
- One business implication (what this means for the product)
- One recommended action (what the PM does next)
- No model metrics on summary slides — AUC and confusion
  matrix go in the methodology appendix only

**The test for every output:**
"Can a PM act on this Monday morning without understanding
what a coefficient or importance score is?"
If yes — ready. If no — rephrase until all statistical
language is replaced with business language.

---

### D-13 | Final Feature Confirmations from EDA

**AIComplex — confirmed include:**
Missingness check in 01_eda.ipynb showed AIComplex was answered
by both intenders and rejectors above the 60% non-null threshold.
The question is NOT conditionally routed to current users only.
Include in model with mode imputation for remaining NAs.

**AIFrustration — confirmed include:**
Both intenders and rejectors answered at sufficient rates (above
60% non-null threshold). High response rate from non-users confirms
they have formed opinions about AI tool quality through secondhand
experience or trial use. Include in model.

**WorkExp — dropped despite r = 0.844 (below 0.85 threshold):**
Rule said keep both at r < 0.85, but 0.844 is close enough to
the threshold that multicollinearity risk in Logistic Regression
remains meaningful. YearsCode retained as it more directly captures
coding-context experience relevant to AI tool adoption. WorkExp
dropped. Decision documented for model transparency.

**ICorPM — flagged as likely low-importance predictor:**
EDA showed near-identical IC/PM split across intenders and
rejectors (~85% IC in both groups). Kept in for now — Random
Forest importance score will confirm. Expect near-zero importance,
likely trimmed before Logistic Regression.

**PurchaseInfluence — flagged as likely low-importance predictor:**
EDA showed similar influence profiles between intenders and
rejectors. Active users show higher influence but this appears
to be a consequence of adoption rather than a predictor of intent.
Kept in for now — expect low importance in model.

---

### D-14 | Modeling Results and Key Findings

**Model performance:**
Random Forest AUC-ROC 0.8543, Logistic Regression AUC-ROC 0.8537,
CART AUC-ROC 0.8332. All three above the 0.7 useful threshold.
RF and LR essentially identical — confirming the 17-feature trimmed
set preserved all meaningful signal.

**Key finding 1 — Sentiment dominates:**
AI Sentiment Very Unfavorable is the single most important feature
at 0.32 RF importance. Odds ratio 0.041 — developers with very
unfavorable sentiment are 24x more likely to be rejectors than
intenders. Consistent across all three models as the first CART split.

**Key finding 2 — Active learning is the strongest intender signal:**
Actively Learned AI Tooling ranks 2nd in RF importance at 0.11,
odds ratio 2.602. Developers who actively learned AI tooling are
2.6x more likely to be intenders. The gap is behavioral not
attitudinal — intenders have invested in preparation but not
crossed over to committed use.

**Key finding 3 — Perceived quality uncertainty keeps intenders stuck:**
AI Complexity Good Not Great odds ratio 2.037, AI Almost Right Not
Quite odds ratio 1.221. Intenders have tried AI, found it partially
useful but not reliably enough. Output quality perception is the
specific friction point.

**Key finding 4 — Demographics are weak predictors:**
43 of 60 features trimmed with importance below 0.01. OrgSize,
Industry, DevType, ICorPM, PurchaseInfluence did not appear in
top 10. The intender gap is attitudinal and behavioral, not
demographic.

**Three CART personas:**
- Path 1 — The Hesitant Intender: n=948, 61.5% intender probability.
  Not hostile toward AI, not actively learning, no strong complexity
  opinion. Blocked by inertia not hostility. PM intervention: low
  friction first-use trigger.
- Path 2 — The Firm Young Rejector: n=763, 97.3% rejector probability.
  Very unfavorable sentiment, young (18-34), no job threat concern.
  Ideological rejection. Do not spend acquisition resources here.
- Path 3 — The Experienced Rejector: n=698, 92.6% rejector probability.
  Very unfavorable sentiment, age 35+, 19.5+ years coding experience.
  Deep-rooted skepticism. Hard to move.

**VIF check:** All 17 features passed. Highest VIF 2.96, no trimming
required at the VIF > 5 threshold.

**Clustering decision:**
Proceeding with Act 3 clustering on intenders only. Although Very
Unfavorable sentiment exceeded the 0.25 dominance threshold in the
full model, this feature has near-zero variance within the
intender-only group (intenders by definition do not hold very
unfavorable sentiment at high rates). Clustering on attitude variables
within intenders may reveal meaningful sub-personas not visible in
the full intenders-vs-rejectors model.

---

### D-15 | Clustering Results — Three Intender Personas

Optimal k=3 selected. k=4, 5, 6 disqualified — produced clusters of
only 31 developers, below the 50-developer minimum threshold for
reliable profiling. Silhouette score 0.1187 — weak but expected for
binary survey data.

**Cluster 0 — Never-Started Intenders (n=635, 44.2%):**
93.2% have never used AI for complex tasks and have no formed opinion.
Blocked by pure inertia — no trial, no friction, no engagement.
PM intervention: zero-config frictionless trial, one-click install,
instant output. Remove every barrier between intent and first use.

**Cluster 1 — Engaged Fence-Sitters (n=649, 45.2%):**
Highest active-learning rate (54.1%), most positive sentiment, blocked
by ambivalence — tool works but no breakthrough moment yet.
Closest to crossing over.
PM intervention: workflow-specific onboarding, use-case targeted first
win, developer peer case studies.

**Cluster 2 — Quality-Disappointed Intenders (n=153, 10.6%):**
100% rated AI complexity as Very Poor. Almost Right frustration (52.3%)
and Debugging Slower (50.3%). Still actively learning (43.1%) despite
disappointment.
PM intervention: accuracy transparency, confidence scores, steer toward
reliable use cases — boilerplate generation, documentation drafting,
test scaffolding.

**Key insight from clustering:**
The predictive model correctly identified intenders as a group but
masked internal heterogeneity. Clusters 0 and 1 require opposite
interventions — Cluster 0 needs a frictionless entry point
(simplicity), Cluster 1 needs a breakthrough moment (depth). A single
onboarding strategy underserves both. Cluster 2 converted developers
become credible product champions — their earned skepticism makes their
endorsement more persuasive than a first-time adopter's.

**Why train data only:**
Clustering is unsupervised and technically has no leakage risk.
Training data used for consistency with Act 2 modeling conventions.
Full intender population (n=1,797) would produce identical personas.

---

## Interview Q&A — Master List

**Q: Why the Stack Overflow survey instead of behavioral data?**
The survey is real data from 49,000 actual developers. Alternatives
require scraping (ToS issues) or proprietary access I don't have.
Key limitation: self-reported behavior, acknowledged in writeup.

**Q: Isn't Stack Overflow's own analysis already doing this?**
No. Stack Overflow's published analysis is descriptive — one
variable at a time, pre-defined segments, percentage bar charts.
My project is predictive. I built a model identifying which
combination of characteristics predicts whether a developer with
stated intent to adopt AI tools actually does. Stack Overflow
shows the percentages. I built the targeting engine a PM would
use to act on them.

**Q: You dropped 31% of your data. Doesn't that bias your results?**
This was tested, not assumed. Analyzed all 15,471 NA rows: dropout
happened before respondents reached AISelect, demographic profile
matches overall sample. Both findings support MCAR. Acknowledged
in limitations section.

**Q: The intender group is only 1,797 people. Isn't that too small?**
1,797 is well above the threshold for logistic regression and CART.
Kept narrow deliberately — casual users represent a different funnel
stage (post-activation vs. pre-activation) and combining them would
obscure the core insight. Stratified split handles class imbalance.

**Q: Why Random Forest before Logistic Regression?**
After multi-select column explosion the feature set grows to 35-40
columns with correlated binary indicators. Running Logistic
Regression on that untrimmed set risks distorted coefficients.
Random Forest handles correlated features naturally and importance
scores provide an empirical trim before Logistic Regression runs.
VIF was also run as a secondary validation check within
Logistic Regression as a safety net.

**Q: Why AUC-ROC and not accuracy?**
Class split is approximately 25/75. A model predicting "rejector"
for everyone achieves 75% accuracy while learning nothing.
Accuracy is misleading for imbalanced classes. AUC-ROC measures
how well the model distinguishes intenders from rejectors
regardless of class distribution.

**Q: What would you do differently with more time or data?**
Three things: (1) Test whether dropouts differ from completers on
observable characteristics to more rigorously confirm MCAR;
(2) Collect a targeted follow-up survey specifically aimed at the
intender group to get richer activation barrier signal;
(3) Add NLP analysis of the AIOpen free-text column to capture
qualitative future-skill anxieties the structured columns miss.

---

*Document last updated: End of Phase 2*
*Next update: After Phase 3 — add modeling decisions and surprises*
*Final update: After Phase 4 — add key findings and interpretations*
