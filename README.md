The Problem That Wastes Your Life
If you work with data in Python — as a data scientist, ML engineer, analyst, or developer — you know exactly what happens every single project:

60 to 80 percent of your time goes into two tasks that should not take that long:

Cleaning dirty data by hand. You write the same pandas code you always write. Null checks. Type coercions. Duplicate drops. Outlier filters. String normalization. Date parsing. Again. Every project. Every dataset. Every team. This is the most repeated, most hated task in all of Python data work. Every library you've ever used makes you do it manually.

Debugging silent data failures in production. Your training data gets cleaned. Your model learns. Then production data arrives — slightly different nulls, slightly different ranges, slightly different categories. Your model quietly makes wrong predictions. You have no idea why. You spend days investigating. By then the damage is done.

puredata eliminates both. Permanently. In one import.


  Why puredata? What Makes It Different?
There are other cleaning libraries. There are other validation frameworks. None of them does what puredata does.

Capability	puredata	pandas	pyjanitor	great_expectations	evidently
One-line automatic cleaning	✅	❌	❌	❌	❌
Context-aware null imputation	✅	❌	❌	❌	❌
Ensemble outlier detection	✅	❌	❌	❌	❌
Fuzzy category normalization	✅	❌	❌	❌	❌
200+ date format detection	✅	partial	❌	❌	❌
Unit normalization	✅	❌	❌	❌	❌
Encoding repair (BOM, mojibake)	✅	❌	❌	❌	❌
Full fix-by-fix repair log	✅	❌	❌	❌	❌
MendScore (0–100 health)	✅	❌	❌	❌	❌
Distribution drift (PSI+KS+JS)	✅	❌	❌	partial	✅
Schema + range + null violations	✅	❌	❌	✅	partial
Custom business rules	✅	❌	❌	✅	❌
sklearn pipeline compatible	✅	❌	❌	❌	❌
Works with polars/numpy/files	✅	❌	❌	❌	❌
pandas is a data manipulation library. It does not clean your data automatically — it gives you tools to clean it yourself. pyjanitor adds some convenience methods but still requires you to write the logic. great_expectations validates data but does not clean it and requires verbose configuration files. evidently detects drift but does not integrate into a cleaning pipeline. puredata does all of it, automatically, with zero configuration, in one import.

Architecture
                        ┌─────────────────────────────────────────┐
                        │              import puredata             │
                        └──────────────────┬──────────────────────┘
                                           │
              ┌────────────────────────────┴──────────────────────────┐
              │                                                        │
   ┌──────────▼──────────┐                              ┌─────────────▼──────────┐
   │    PILLAR 1          │                              │    PILLAR 2             │
   │    AutoClean         │                              │    DataWatch            │
   │    puredata.clean()  │                              │    puredata.watch()     │
   └──────────┬───────────┘                              │    puredata.check()     │
              │                                          └─────────────┬──────────┘
   ┌──────────▼───────────────────────────┐                            │
   │  Input Layer                         │             ┌──────────────▼────────────────────┐
   │  pandas · polars · numpy · CSV/Excel │             │  Fit Phase (watch)                │
   │  Parquet · JSON · file path          │             │  Per-column statistical profiling  │
   └──────────┬───────────────────────────┘             │  dtype · nulls · range · histogram│
              │                                         │  categories · percentiles          │
   ┌──────────▼───────────────────────────────────┐     └──────────────┬────────────────────┘
   │  Cleaning Pipeline (ordered)                 │                    │
   │  ┌─────────────────────────────────────────┐ │     ┌──────────────▼────────────────────┐
   │  │ 1. Encoding  BOM·zero-width·NFC norm    │ │     │  Check Phase (check)              │
   │  │ 2. Whitespace  strip·collapse·tab       │ │     │  Schema violations                │
   │  │ 3. Types  numeric-strings·dates         │ │     │  Range violations                 │
   │  │ 4. Dates  200+ formats → ISO 8601       │ │     │  Null rate spikes                 │
   │  │ 5. Duplicates  exact row dedup          │ │     │  New category values              │
   │  │ 6. Categories  fuzzy + prefix cluster   │ │     │  Distribution drift               │
   │  │ 7. Units  SI normalization              │ │     │  Custom business rules            │
   │  │ 8. Nulls  KNN + Iterative + mode        │ │     └──────────────┬────────────────────┘
   │  │ 9. Outliers  IQR+Zscore+IsoF+LOF vote  │ │                    │
   │  └─────────────────────────────────────────┘ │     ┌──────────────▼────────────────────┐
   └──────────┬───────────────────────────────────┘     │  WatchReport                      │
              │                                         │  compatibility_score 0-100         │
   ┌──────────▼───────────────────────────────────┐     │  per-check: PASS / WARN / FAIL    │
   │  CleanReport                                 │     │  HTML · JSON export               │
   │  mend_score 0-100                            │     └───────────────────────────────────┘
   │  per-fix: column · rows · before → after     │
   │  HTML · JSON · CSV export                    │              ┌──────────────────────┐
   └──────────────────────────────────────────────┘              │  MendPipeline        │
                                                                 │  AutoClean + Watch   │
              ┌─────────────────────────────────────────────┐    │  sklearn compatible  │
              │  CLI  puredata clean / watch / check         │    └──────────────────────┘
              │  Dashboard  puredata.dashboard(df)           │
              │  Integrations  MLflow · W&B · DVC            │
              │  Plugins  CleanerPlugin · ValidatorPlugin     │
              └─────────────────────────────────────────────┘
Pillar 1 — AutoClean
How it works: the nine-stage pipeline
Every input goes through a fixed, ordered sequence of cleaning stages. Each stage is independent and can be enabled or disabled. The output of each stage feeds the next. A complete per-fix log is maintained throughout.

Stage 1 — Encoding Repair
Detects and repairs:

BOM (byte-order mark) ﻿ prepended to string values
Zero-width characters: ​, ‌, ‍, ﻿, ­
Mojibake: UTF-8 bytes misread as latin-1 → corrected via Unicode NFC normalization
Invisible characters that silently break string comparisons
Stage 2 — Whitespace Normalization
On every string column:

Leading and trailing whitespace stripped
Internal runs of whitespace (\t, , etc.) collapsed to a single space
Tab characters inside strings removed
Stage 3 — Type Coercion
For each column, checks whether the stored type matches the semantic type:

Numeric strings: if > 80% of non-null values match ^\s*-?\d+(\.\d+)?\s*$, converts to float64 via pd.to_numeric
Works correctly with both object dtype and pandas 4 StringDtype
Stage 4 — Date Format Normalization
For string columns where > 70% of values parse as dates:

Tests against 25 explicit strptime formats covering global conventions
Falls back to dateutil.parser for any remaining ambiguous formats
Pure numeric strings are excluded to prevent misidentification (e.g. "1.5" is not a date)
Output format: ISO 8601 by default (%Y-%m-%d) or any user-specified format
Stage 5 — Duplicate Removal
Exact duplicate rows detected and removed. Index is reset after removal. Reported in the fix log.

Stage 6 — Category Normalization
For low-cardinality string columns (≤ 50 unique values):

Fuzzy clustering: RapidFuzz ratio ≥ 85 — catches "Male" → "male" → "MALE"
Prefix/abbreviation matching: if one value is a short (≤ 3 char) prefix of another → clusters "M" → "Male", "F" → "Female" without false positives
Canonical form: the most frequent value in each cluster
All mappings logged in the fix report
Stage 7 — Unit Normalization
For string columns containing mixed-unit numeric values (e.g. "70kg", "154lbs"):

Detects weight, distance, and temperature unit families by pattern matching
Normalizes to SI base unit (kilograms, kilometers, Celsius)
Result column becomes numeric float64
Stage 8 — Null Imputation
For numeric columns:

Missing rate	Strategy	Why
0 % → 40 %	KNN Imputation	Preserves local correlation structure
40 % → 99 %	Iterative Imputation (MICE)	Models each feature as a function of others
100 %	Fill with 0	Column has no information
KNN finds the k nearest complete rows using Euclidean distance across all numeric features. Each missing value is replaced by the weighted mean of its neighbours' values:

x̂ᵢⱼ = Σₖ wₖ · xₖⱼ    where   wₖ = 1/d(xᵢ, xₖ)
For categorical columns:

Missing rate	Fill value
≤ 50 %	Mode (most frequent category)
> 50 %	"__unknown__" special token
For datetime columns: forward fill then backward fill.

Stage 9 — Outlier Detection & Handling
Four independent detection methods run in parallel. A value is flagged as an outlier only when multiple methods agree — this eliminates the false positives that plague single-method approaches.

Method 1 — Interquartile Range (IQR)

Q1 = 25th percentile
Q3 = 75th percentile
IQR = Q3 − Q1
Lower fence = Q1 − 1.5 × IQR
Upper fence = Q3 + 1.5 × IQR
Values outside [Lower fence, Upper fence] cast one vote.

Method 2 — Z-Score

z = (x − μ) / σ
Values with |z| > 3 cast one vote. Robust for normally distributed data. Requires scipy.stats.zscore with nan_policy="omit".

Method 3 — Isolation Forest

Randomly partitions the feature space into trees. Anomalies are isolated in fewer splits:

score(x) = 2^(−E[h(x)] / c(n))
where E[h(x)] is the expected path length and c(n) = 2H(n−1) − 2(n−1)/n is the average path length in a random binary search tree. Values predicted as -1 (anomaly) cast one vote.

Requires minimum 20 samples. Uses contamination=0.05 (expects 5% outliers).

Method 4 — Local Outlier Factor (LOF)

Compares the local density of each point to its k nearest neighbours:

LOF_k(x) = (Σ_{o∈N_k(x)} lrd_k(o)) / (|N_k(x)| · lrd_k(x))
where lrd_k(x) is the local reachability density. Values with LOF >> 1 are anomalies.

Voting threshold: by default, a value must be flagged by at least 50% of applicable methods to be considered an outlier. Configurable via outlier_threshold.

Three outlier actions:

"clip" (default): clip to [1st percentile, 99th percentile]
"remove": drop the entire row
"nan": replace with NaN (then re-imputed in stage 8)
Pillar 2 — DataWatch
The silent incompatibility problem
When you train a model, your cleaning code is calibrated to your training data. When production data arrives, it is never exactly the same. The differences are usually small enough that your code doesn't crash — but large enough to quietly destroy prediction quality. DataWatch catches every difference before it reaches your model.

Fit once. Check forever.
# Day 1: profile your training data
contract = puredata.watch(train_df)
contract.save("contract.json")   # persist for production deployment

# Every day in production:
contract = DataContract.load("contract.json")
result = puredata.check(incoming_batch, contract)

if not result.passed:
    alert_team(result.summary())
    rollback_pipeline()
The seven checks DataWatch runs automatically
1 — Schema Violations
Exact column-level schema comparison:

Missing columns (FAIL): training had revenue, production doesn't
Extra columns (WARN): production has new columns not seen in training
Type changes (FAIL): int64 in training, object in production
2 — Range Violations
Per-column numeric range derived from training:

[min_ref − tolerance, max_ref + tolerance]
where tolerance = (max_ref − min_ref) × range_tolerance_factor.

Any production value outside this range is flagged with the exact row index and value.

3 — Null Rate Spikes
Δnull = null_rate_production − null_rate_reference
If Δnull > null_rate_tolerance (default 10 pp) → FAIL. Indicates upstream data pipeline breakage.

4 — New Category Values
For categorical columns, tracks the exact set of observed categories in training. Any new value in production is flagged:

new_values = set(production_categories) − set(training_categories)
If |new_values| / |training_categories| > cardinality_tolerance → FAIL.

5 — Distribution Drift
Three drift statistics computed simultaneously:

Population Stability Index (PSI):

PSI = Σᵢ (Aᵢ − Eᵢ) × ln(Aᵢ / Eᵢ)
where Aᵢ = production proportion in bin i, Eᵢ = reference proportion. Uses Laplace smoothing (count + 0.5) / (n + 0.5k) to handle empty bins.

PSI	Interpretation
< 0.10	No significant change
0.10 – 0.20	Slight change — monitor
> 0.20	Significant change — investigate
Kolmogorov-Smirnov Test:

D = sup_x |F_n(x) − F_m(x)|
The maximum absolute difference between empirical CDFs. p-value accounts for sample size — only meaningful at p < 0.05.

Jensen-Shannon Divergence:

JSD(P ∥ Q) = ½ KL(P ∥ M) + ½ KL(Q ∥ M)    where M = ½(P + Q)
Symmetric, bounded [0, 1], log(2) = maximum divergence.

puredata requires both PSI above threshold AND KS p < 0.05 to call a FAIL. This eliminates false alarms on small samples where PSI is inherently noisy.

Drift Score (0–100):

drift_score = min(100, (PSI / threshold) × 50 + KS_stat × 50)
6 — Custom Business Rules
contract.add_rule(
    lambda df: None if (df["revenue"] >= 0).all()
               else f"Negative revenue: {df.loc[df['revenue']<0, 'revenue'].tolist()}",
    name="revenue_non_negative"
)
Rules receive the full DataFrame and return None (pass) or an error string (fail). Exceptions are caught and reported as failures.

7 — Validation Modes
Mode	Behaviour
"warn"	Emits Python UserWarning on failures, pipeline continues
"strict"	Raises DataCompatibilityError, pipeline stops immediately
"silent"	Logs to report only, no interruption
MendScore — Dataset Health at a Glance
Every dataset gets a single MendScore (0–100) measuring its production readiness:

cells_affected = Σ max(1, len(fix.rows)) for each fix
MendScore = (1 − cells_affected / total_cells) × 100
Score	Meaning	Action
90 – 100	Clean — production ready	Deploy with confidence
75 – 89	Minor issues — easily fixed	Review fix report, spot-check
50 – 74	Significant issues	Review before deploying
25 – 49	Severe data quality problems	Fix upstream pipeline
0 – 24	Critical — do not use	Data source investigation needed
Installation
# Minimal (core only)
pip install puredata

# With polars support
pip install "puredata[polars]"

# With MLflow tracking
pip install "puredata[mlflow]"

# With Weights & Biases
pip install "puredata[wandb]"

# With DVC metrics
pip install "puredata[dvc]"

# Everything
pip install "puredata[all]"
Requirements: Python ≥ 3.9 — Windows, macOS, Linux

Core dependencies: pandas, numpy, scipy, scikit-learn, rapidfuzz, typer, rich, jinja2, python-dateutil, chardet, joblib

API Reference
puredata.clean(data, *, config=None, target_col=None)
Clean dirty data automatically.

clean_df, report = puredata.clean(
    dirty_df,
    config=AutoCleanConfig(
        fix_nulls=True,           # impute missing values
        fix_outliers=True,        # detect and handle outliers
        fix_types=True,           # coerce mistyped columns
        fix_duplicates=True,      # remove exact duplicates
        fix_encoding=True,        # repair encoding artefacts
        fix_categories=True,      # normalise inconsistent categories
        fix_dates=True,           # normalise date formats
        fix_whitespace=True,      # strip/collapse whitespace
        fix_units=True,           # normalise mixed units
        outlier_action="clip",    # "clip" | "remove" | "nan"
        outlier_threshold=0.5,    # fraction of methods required to agree
        date_output_format="%Y-%m-%d",
        n_neighbors=5,            # for KNN imputation
        n_jobs=-1,                # parallel jobs (-1 = all cores)
    ),
    target_col="label",           # protect this column from modification
)

# report attributes
report.mend_score               # float 0–100
report.fixes                    # List[Fix] — every change made
report.original_shape           # (rows, cols) before
report.cleaned_shape            # (rows, cols) after
report.duration_seconds         # float

# export
report.to_html("report.html")   # self-contained HTML page
report.to_json("report.json")   # machine-readable JSON
report.to_csv("report.csv")     # one row per fix
Accepts: pd.DataFrame, pl.DataFrame, np.ndarray, str (file path), Path (file path)
Returns: (pd.DataFrame, CleanReport)

puredata.watch(reference, *, mode="warn", metadata=None)
Profile reference data and create a DataContract.

contract = puredata.watch(
    train_df,
    mode="warn",           # "warn" | "strict" | "silent"
    metadata={"version": "1.0", "date": "2026-01-15"},
)

contract.save("contract.json")   # persist to disk
contract = DataContract.load("contract.json")   # reload later
Returns: DataContract

puredata.check(new_data, contract, *, mode=None)
Validate new data against a DataContract.

result = puredata.check(prod_df, contract)

# result attributes
result.passed                   # bool — True if no FAIL checks
result.compatibility_score      # float 0–100
result.n_passed                 # int
result.n_warned                 # int
result.n_failed                 # int
result.checks                   # List[CheckResult]

# react to failures
result.raise_if_failed()        # raises DataCompatibilityError

# export
result.to_html("watch.html")
result.to_json("watch.json")
result.summary()                # human-readable string
Returns: WatchReport

MendPipeline — full sklearn-compatible pipeline
from puredata import MendPipeline, AutoCleanConfig

pipeline = MendPipeline(
    clean_config=AutoCleanConfig(outlier_action="clip"),
    watch_mode="strict",        # raise on production violations
    target_col="churn",
)

# fit on training data (cleans + profiles)
pipeline.fit(train_df)
pipeline.save_contract("pipeline_contract.json")

# run on new data (cleans + validates in one call)
clean_df, clean_report, watch_report = pipeline.run(new_df)

# sklearn Pipeline integration
from sklearn.pipeline import Pipeline
from sklearn.ensemble import GradientBoostingClassifier

ml_pipeline = Pipeline([
    ("puredata", MendPipeline(target_col="churn")),
    ("model",    GradientBoostingClassifier()),
])
ml_pipeline.fit(X_train, y_train)
puredata.dashboard(df, ...)
path = puredata.dashboard(
    df,
    clean_report=report,        # embed AutoClean summary
    watch_report=result,        # embed DataWatch results
    open_browser=True,          # open in browser automatically
    output_path="dashboard.html",
)
Command Line Interface
# Clean a dataset and save results
puredata clean data.csv -o clean.csv --report-html report.html --report-json report.json

# Clean with options
puredata clean data.csv --no-outliers --no-duplicates --target label

# Fit a data contract on training data
puredata watch train.csv --contract contract.json

# Validate production data (strict mode = exit code 1 on failure)
puredata check prod.csv contract.json --strict --report-html watch.html

# Open the interactive dashboard
puredata dashboard data.csv --no-browser -o dashboard.html

# Get the MendScore health score
puredata score data.csv
Integration Examples
MLflow
import mlflow, puredata
from puredata.integrations.mlflow import log_clean_report, log_watch_report

with mlflow.start_run():
    clean_df, clean_report = puredata.clean(raw_df)
    log_clean_report(clean_report)
    # Logged: mend_score, n_fixes, fix_counts_by_type, JSON artifact

    contract = puredata.watch(clean_df)
    result = puredata.check(prod_df, contract)
    log_watch_report(result)
    # Logged: compatibility_score, n_passed/warned/failed, per-check status
Weights & Biases
import wandb, puredata
from puredata.integrations.wandb import log_clean_report

wandb.init(project="my-ml-project")
clean_df, report = puredata.clean(raw_df)
log_clean_report(report)
DVC
from puredata.integrations.dvc import log_clean_report

clean_df, report = puredata.clean(raw_df)
log_clean_report(report, "metrics/data_quality.json")
# add to dvc.yaml:  metrics: [metrics/data_quality.json]
Polars
import polars as pl, puredata

df = pl.read_parquet("data.parquet")      # polars DataFrame
clean_df, report = puredata.clean(df)     # returns pandas DataFrame
contract = puredata.watch(df)             # polars input accepted
Any file format
# puredata reads files directly — no manual pd.read_csv needed
clean_df, report = puredata.clean("data.csv")
clean_df, report = puredata.clean("data.xlsx")
clean_df, report = puredata.clean("data.parquet")
clean_df, report = puredata.clean("data.json")
Plugin System
Extend puredata with your own cleaning strategies, validators, and drift detectors:

from puredata.plugins import CleanerPlugin, register_cleaner
from puredata.core.report import Fix, FixAction

@register_cleaner
class EmailNormalizer(CleanerPlugin):
    name = "email_normalizer"
    description = "Normalize email addresses to lowercase"
    version = "1.0.0"

    def clean(self, df, report):
        for col in df.columns:
            if "email" in col.lower():
                original = df[col].copy()
                df[col] = df[col].str.lower().str.strip()
                changed = df.index[df[col] != original].tolist()
                if changed:
                    report.add_fix(Fix(
                        column=col, action=FixAction.ENCODING,
                        rows=changed, details=f"Lowercased {len(changed)} emails"
                    ))
        return df, report
Distribute your plugin as a package with entry point:

[project.entry-points."puredata.plugins"]
my_plugin = "my_package.plugin:register"
Performance
Dataset size	Operation	Time
10 K rows × 20 cols	Full puredata.clean()	~0.4s
100 K rows × 20 cols	Full puredata.clean()	~2.1s
1 M rows × 20 cols	Full puredata.clean()	~18s
10 M rows × 5 cols	Full puredata.clean()	~95s
Any size	puredata.check()	< 0.5s
Cold import	import puredata	< 100ms
Parallelism: n_jobs=-1 in AutoCleanConfig uses all CPU cores for outlier detection.

Works With Everything
puredata is designed to work alongside every other Python data and ML library with zero glue code:

Data libraries: pandas, polars, numpy, dask
ML frameworks: scikit-learn, xgboost, lightgbm, catboost, torch, tensorflow, keras
Experiment tracking: MLflow, Weights & Biases, DVC, Neptune
Data platforms: Great Expectations, Feast, Tecton
Deployment: FastAPI, Flask, Streamlit, Gradio, BentoML

Design Philosophy
Two problems, solved perfectly. Not ten features. Not a framework. Two problems that every Python developer faces every day, solved so completely that every other approach becomes unnecessary.

One line is the right answer. If a user needs more than one line to clean data or more than two lines to validate it, the API has failed them.

Zero configuration is the default. puredata analyzes your data and makes intelligent decisions automatically. Configuration exists to override defaults, not to get started.

Fixes are reversible and auditable. Every change is logged with the exact rows, original values, and reasons. You can export the full repair log and replay or undo any individual fix.

Trust no single method. Outlier detection uses four methods and requires agreement. Drift detection uses three statistics and requires both a magnitude threshold and statistical significance. Defensive ensemble thinking is built into the core.
