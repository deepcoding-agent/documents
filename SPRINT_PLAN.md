# SPRINT_PLAN.md — PrepPilot Development Roadmap (v1)

> Last updated: 2026-03-27

---

## Current State (Sprint 2.95 Completed)

| Sprint | Status | Deliverable |
|--------|--------|-------------|
| 1 | ✅ Done | Data preprocessing pipeline, PrepConfig, report card |
| 2 | ✅ Done | Plotly interactive charts, EDA report, fullscreen |
| 2.5 | ✅ Done | AI-first agent rewrite: planner-driven routing |
| 2.7 | ✅ Done | AI Auto-Clean, AI Auto-Prepare, folder card, landing page redesign |
| 2.9 | ✅ Done | 350 handlers (7 categories x 50), two-stage routing, NLP/Analysis, 20+ file formats |
| 2.95 | ✅ Done | 396 handlers (modular packages), /insights, /report, translate, multi-format export |

### What We Have Today

**396 handlers across 7 categories:**

| Category | Count | Highlights |
|----------|-------|------------|
| Stats | 56 | Descriptive, hypothesis tests, correlation, distribution, chi2, ANOVA |
| Clean | 54 | Null handling, dedup, normalize, anonymize, parse, trim |
| Transform | 59 | Encoding (one-hot, ordinal, label), scaling, pivot, merge, date ops |
| Viz | 57 | 57 Plotly chart types (bar, scatter, heatmap, sankey, candlestick, ...) |
| Feature | 54 | PCA, SVD, WoE, binning, interaction, polynomial, cumulative |
| NLP | 57 | Tokenize, sentiment, NER, translate, TF-IDF, n-gram, stopwords |
| Analysis | 59 | Clustering, profiling, time series, Granger, market basket, lift |

**4 slash commands:**

| Command | Purpose |
|---------|---------|
| `/cleaning` | AI Auto-Clean — analyze + plan + execute clean handlers |
| `/ml-prepare` | AI Auto-Prepare — AI picks PrepConfig, 10-step pipeline, outputs X_train/X_test/y_train/y_test |
| `/insights` | Technical insights — weaknesses, possible analyses, action plan |
| `/report` | Business EDA — use cases, market analysis, promotion strategies |

**Architecture:**
- Two-stage AI routing (category router 150 tokens -> focused planner)
- 20+ file format upload (Excel, CSV, JSON, TXT, YAML, Parquet, ...)
- Sandboxed Python execution
- Multi-provider LLM (Anthropic Claude + OpenAI GPT)
- Plotly-first interactive charts
- Client-side export (CSV, XLSX, JSON, TSV, XML, MD, HTML, PDF)

---

## Sprint 3: Complete Data Preparation

**Goal:** Close every remaining gap in data preparation so PrepPilot can handle any preprocessing task before model training.

**Status:** 🔲 Not Started

### Part 3.1 — Imbalanced Data Handling

Most real-world datasets are imbalanced. Without these handlers, ML models trained on PrepPilot data will be biased toward majority classes.

| # | Handler | Category | Description |
|---|---------|----------|-------------|
| 1 | `smote_oversample` | clean | SMOTE — generate synthetic minority samples using k-NN interpolation |
| 2 | `random_oversample` | clean | Randomly duplicate minority class rows |
| 3 | `random_undersample` | clean | Randomly remove majority class rows |
| 4 | `adasyn_oversample` | clean | ADASYN — adaptive synthetic sampling, focuses on harder-to-learn examples |
| 5 | `class_weight_calc` | stats | Calculate balanced class weights for model training |
| 6 | `imbalance_report` | analysis | Class distribution analysis with visual + resampling recommendation |

**New dependency:** `imbalanced-learn>=0.12`

**Files:**
```
api/handlers/clean/smote_oversample.py
api/handlers/clean/random_oversample.py
api/handlers/clean/random_undersample.py
api/handlers/clean/adasyn_oversample.py
api/handlers/stats/class_weight_calc.py
api/handlers/analysis/imbalance_report.py
```

### Part 3.2 — Advanced Feature Selection

Current feature handlers include PCA, SVD, and interaction terms but lack automated feature selection methods that evaluate feature relevance against a target variable.

| # | Handler | Category | Description |
|---|---------|----------|-------------|
| 7 | `rfe_select` | feature | Recursive Feature Elimination — iteratively remove weakest features |
| 8 | `mutual_info_select` | feature | Rank features by mutual information with target |
| 9 | `select_k_best` | feature | SelectKBest using chi2, f_classif, or f_regression scoring |
| 10 | `variance_threshold` | feature | Drop features with variance below threshold |
| 11 | `correlation_select` | feature | Drop one of each pair of highly correlated features |
| 12 | `boruta_select` | feature | Boruta all-relevant feature selection (shadow features + random forest) |

**New dependency:** `boruta>=0.3`

**Files:**
```
api/handlers/feature/rfe_select.py
api/handlers/feature/mutual_info_select.py
api/handlers/feature/select_k_best.py
api/handlers/feature/variance_threshold.py
api/handlers/feature/correlation_select.py
api/handlers/feature/boruta_select.py
```

### Part 3.3 — Time Series Feature Engineering

Existing analysis handlers cover stationarity test and decomposition, but we lack feature engineering specifically designed for time series ML (lag, rolling, seasonal encoding).

| # | Handler | Category | Description |
|---|---------|----------|-------------|
| 13 | `lag_features` | feature | Create lag columns (user-configurable 1..N lags) |
| 14 | `rolling_window` | feature | Rolling mean, std, min, max (configurable window size) |
| 15 | `expanding_window` | feature | Expanding/cumulative statistics |
| 16 | `diff_features` | feature | 1st and 2nd order differencing for stationarity |
| 17 | `datetime_decompose` | transform | Extract year, month, day, weekday, hour, quarter, is_weekend |
| 18 | `seasonal_features` | feature | Cyclical sin/cos encoding for periodic features |

**Files:**
```
api/handlers/feature/lag_features.py
api/handlers/feature/rolling_window.py
api/handlers/feature/expanding_window.py
api/handlers/feature/diff_features.py
api/handlers/feature/seasonal_features.py
api/handlers/transform/datetime_decompose.py
```

### Part 3.4 — Advanced Encoding & Sampling

Current encoding covers one-hot, ordinal, and label. Missing: target-aware encoding methods that improve ML performance, and sampling strategies for large datasets.

| # | Handler | Category | Description |
|---|---------|----------|-------------|
| 19 | `target_encode` | transform | Target encoding — replace category with mean of target variable |
| 20 | `frequency_encode` | transform | Replace category with its frequency count |
| 21 | `leave_one_out_encode` | transform | LOO encoding — target encoding with leave-one-out to reduce overfitting |
| 22 | `stratified_sample` | transform | Stratified random sampling preserving class ratios |
| 23 | `systematic_sample` | transform | Select every N-th row |

**Files:**
```
api/handlers/transform/target_encode.py
api/handlers/transform/frequency_encode.py
api/handlers/transform/leave_one_out_encode.py
api/handlers/transform/stratified_sample.py
api/handlers/transform/systematic_sample.py
```

### Part 3.5 — Data Validation & Quality

Missing: automated schema enforcement and data drift detection to catch data quality issues before training.

| # | Handler | Category | Description |
|---|---------|----------|-------------|
| 24 | `schema_validate` | analysis | Validate column types, ranges, nullability against expected schema |
| 25 | `data_drift_detect` | analysis | Compare two datasets for distribution drift (KS test, PSI) |
| 26 | `constraint_check` | analysis | Check uniqueness, value range, regex format constraints |
| 27 | `auto_dtype_infer` | clean | Smart data type inference + auto-conversion |

**Files:**
```
api/handlers/analysis/schema_validate.py
api/handlers/analysis/data_drift_detect.py
api/handlers/analysis/constraint_check.py
api/handlers/clean/auto_dtype_infer.py
```

### Sprint 3 Summary

| Metric | Value |
|--------|-------|
| New handlers | +27 |
| Total handlers after sprint | ~423 |
| New dependencies | `imbalanced-learn`, `boruta` |
| Modified files | `api/handlers/__init__.py`, `api/agents/planner.py` (handler descriptions) |
| New handler files | 27 |

### Sprint 3 Acceptance Criteria

- [ ] All 27 handlers registered in `HANDLER_REGISTRY`
- [ ] All 27 handlers described in `planner.py` for AI routing
- [ ] SMOTE/ADASYN work on both binary and multiclass targets
- [ ] Feature selection handlers return selected column list + scores
- [ ] Time series features handle missing datetime index gracefully
- [ ] `python -c "from api.main import app; print('OK')"` passes
- [ ] Existing 396 handlers still work (no regressions)

---

## Sprint 4: AI Auto ML — Model Training Pipeline

**Goal:** Users type `/train` or "train a model to predict price" in chat. The system automatically selects the best algorithm, tunes hyperparameters, evaluates performance with charts, and saves the model — all through the chat interface.

**Status:** 🔲 Not Started

### System Architecture

```
User: "/train" or "เทรนโมเดลทำนาย churn"
  │
  ├── Frontend detects /train command or training intent
  │   └── Sends to POST /train with dataset_id + options
  │
  ▼
┌──────────────────────────────────────────────────────┐
│  POST /train (api/routes/train.py)                    │
│  Receives: dataset rows, target_column, options       │
│  Delegates to TrainAgent                              │
└──────────────┬───────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────┐
│  TrainAgent (api/agents/train_agent.py)               │
│                                                       │
│  Step 1: ANALYZE                                      │
│    - Detect task type (classification/regression)     │
│    - Check class balance, feature types, data size    │
│    - Determine if auto-prep needed                    │
│                                                       │
│  Step 2: AUTO-PREP (if raw data)                      │
│    - Handle nulls, encode categoricals, scale         │
│    - Train/test split (stratified if classification)  │
│    - Save preprocessing pipeline for reuse            │
│                                                       │
│  Step 3: SELECT ALGORITHMS                            │
│    - Pick 5-8 candidates from ModelRegistry           │
│    - Consider data size, feature count, task type     │
│                                                       │
│  Step 4: TRAIN + CROSS-VALIDATE                       │
│    - 5-fold stratified CV for each algorithm          │
│    - Collect mean + std for all metrics               │
│                                                       │
│  Step 5: HYPERPARAMETER TUNING                        │
│    - Optuna optimization on top 3 models              │
│    - 50 trials per model (configurable)               │
│                                                       │
│  Step 6: FINAL EVALUATION                             │
│    - Retrain best model on full train set             │
│    - Evaluate on held-out test set                    │
│    - Generate all evaluation charts                   │
│                                                       │
│  Step 7: SAVE & REPORT                                │
│    - Save model + pipeline as .joblib                 │
│    - Build comparison table + charts                  │
│    - Return TrainResultCard artifact                  │
└──────────────────────────────────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────────────┐
│  TrainResultCard (frontend)                           │
│                                                       │
│  ┌────────────────────────────────────────────────┐  │
│  │  Best Model: XGBoost                           │  │
│  │  Task: Binary Classification                    │  │
│  │  Accuracy: 0.95 | F1: 0.94 | AUC: 0.98        │  │
│  └────────────────────────────────────────────────┘  │
│                                                       │
│  ┌─ Model Comparison ────────────────────────────┐   │
│  │ Algorithm        | Acc  | F1   | AUC  | Time  │   │
│  │ XGBoost          | 0.95 | 0.94 | 0.98 | 2.3s  │   │
│  │ LightGBM         | 0.94 | 0.93 | 0.97 | 1.8s  │   │
│  │ RandomForest      | 0.93 | 0.92 | 0.96 | 3.1s  │   │
│  │ GradientBoosting  | 0.92 | 0.91 | 0.95 | 4.5s  │   │
│  │ LogisticRegression| 0.88 | 0.86 | 0.91 | 0.2s  │   │
│  └───────────────────────────────────────────────┘   │
│                                                       │
│  Charts: [Confusion Matrix] [ROC Curve]               │
│          [Feature Importance] [Learning Curve]         │
│                                                       │
│  Actions: [Download Model .joblib]                    │
│           [Use Model to Predict]                      │
└──────────────────────────────────────────────────────┘
```

### Part 4.1 — Model Registry

All supported algorithms organized by task type.

**Classification (12 algorithms):**

| Algorithm | Library | Best For |
|-----------|---------|----------|
| LogisticRegression | sklearn | Baseline, interpretable, linear relationships |
| DecisionTree | sklearn | Interpretable, non-linear, small datasets |
| RandomForest | sklearn | General purpose, handles missing data well |
| ExtraTrees | sklearn | Faster than RF, good with noisy data |
| GradientBoosting | sklearn | High accuracy, sequential ensemble |
| AdaBoost | sklearn | Boost weak learners, good with balanced data |
| XGBoost | xgboost | Competition-winning, handles imbalanced data |
| LightGBM | lightgbm | Fast training, large datasets, categorical native |
| SVM (SVC) | sklearn | High-dimensional data, small-medium datasets |
| KNN | sklearn | Simple, no training phase, similarity-based |
| NaiveBayes | sklearn | Text classification, very fast, probabilistic |
| MLP | sklearn | Complex non-linear patterns, neural network |

**Regression (11 algorithms):**

| Algorithm | Library | Best For |
|-----------|---------|----------|
| LinearRegression | sklearn | Baseline, interpretable, linear relationships |
| Ridge | sklearn | Linear with L2 regularization, multicollinearity |
| Lasso | sklearn | Linear with L1, automatic feature selection |
| ElasticNet | sklearn | Combined L1+L2, many correlated features |
| DecisionTree | sklearn | Non-linear, interpretable |
| RandomForest | sklearn | General purpose, robust to outliers |
| GradientBoosting | sklearn | High accuracy, sequential ensemble |
| XGBoost | xgboost | Competition-winning, handles missing values |
| LightGBM | lightgbm | Fast, large datasets, categorical native |
| SVR | sklearn | High-dimensional, non-linear kernels |
| KNN | sklearn | Local patterns, no assumptions on distribution |

**Clustering (4 algorithms):**

| Algorithm | Library | Best For |
|-----------|---------|----------|
| KMeans | sklearn | Spherical clusters, known K |
| DBSCAN | sklearn | Arbitrary shape, noise detection |
| Hierarchical | sklearn | Dendrogram visualization, unknown K |
| GaussianMixture | sklearn | Soft clustering, elliptical clusters |

**File:** `api/agents/model_registry.py`

```python
MODEL_REGISTRY = {
    "classification": {
        "logistic_regression": {
            "class": "sklearn.linear_model.LogisticRegression",
            "default_params": {"max_iter": 1000},
            "tunable_params": {
                "C": ("float_log", 0.001, 100),
                "penalty": ("categorical", ["l1", "l2"]),
            },
        },
        "random_forest_clf": {
            "class": "sklearn.ensemble.RandomForestClassifier",
            "default_params": {"n_estimators": 100, "random_state": 42},
            "tunable_params": {
                "n_estimators": ("int", 50, 500),
                "max_depth": ("int", 3, 30),
                "min_samples_split": ("int", 2, 20),
            },
        },
        # ... 10 more classification algorithms
    },
    "regression": {
        # ... 11 regression algorithms
    },
    "clustering": {
        # ... 4 clustering algorithms
    },
}
```

### Part 4.2 — Training Agent

Core orchestrator that runs the full pipeline.

**File:** `api/agents/train_agent.py`

Key responsibilities:
1. **Task detection** — Analyze target column: binary (2 classes) → binary classification, 3+ classes → multiclass, continuous → regression, no target → clustering
2. **Smart algorithm selection** — Based on dataset size, feature count, class balance:
   - Small dataset (<1K rows): prefer simpler models (Logistic, KNN, SVM)
   - Large dataset (>100K rows): prefer LightGBM, XGBoost (fast training)
   - Many features (>50): prefer tree-based (handle high-dimensional naturally)
   - Imbalanced classes: prefer XGBoost, LightGBM (built-in class_weight)
3. **Auto-preprocessing** — If data not already prepared:
   - Fill nulls (median for numeric, mode for categorical)
   - Encode categoricals (one-hot if <10 unique, label if >=10)
   - Scale numerics (StandardScaler)
   - Save pipeline as sklearn Pipeline object
4. **Cross-validation** — StratifiedKFold(5) for classification, KFold(5) for regression
5. **Hyperparameter tuning** — Optuna with TPE sampler, 50 trials per top-3 model
6. **Final training** — Best model retrained on full train set, evaluated on test set

### Part 4.3 — Model Evaluator

Generates all metrics and Plotly charts.

**File:** `api/agents/model_evaluator.py`

**Classification metrics:**
- Accuracy, Precision, Recall, F1 Score (macro + weighted)
- AUC-ROC (binary + multiclass OVR)
- Log Loss
- Cohen's Kappa
- Matthews Correlation Coefficient

**Classification charts (Plotly):**
- Confusion Matrix heatmap
- ROC Curve (per class for multiclass)
- Precision-Recall Curve
- Feature Importance bar chart (top 20)
- Learning Curve (train vs validation over training size)
- Class Prediction Distribution

**Regression metrics:**
- RMSE, MAE, MAPE
- R-squared, Adjusted R-squared
- Explained Variance

**Regression charts (Plotly):**
- Actual vs Predicted scatter
- Residual plot
- Residual distribution histogram
- Feature Importance bar chart (top 20)
- Learning Curve

**Clustering metrics:**
- Silhouette Score
- Calinski-Harabasz Index
- Davies-Bouldin Index

**Clustering charts (Plotly):**
- Cluster scatter (PCA 2D)
- Silhouette plot
- Cluster size distribution bar

### Part 4.4 — Model Storage

Save and manage trained models linked to conversations.

**File:** `api/agents/model_storage.py`

Each saved model includes:
```python
{
    "model_id": "uuid",
    "conversation_id": "...",
    "dataset_id": "...",
    "task_type": "classification",
    "algorithm": "xgboost_clf",
    "target_column": "churn",
    "feature_columns": ["age", "balance", "tenure", ...],
    "metrics": {"accuracy": 0.95, "f1": 0.94, "auc": 0.98},
    "hyperparameters": {"n_estimators": 200, "max_depth": 6, ...},
    "preprocessing_pipeline": "<serialized sklearn Pipeline>",
    "model_binary": "<serialized model>",
    "created_at": "2026-03-27T10:00:00Z",
    "training_duration_seconds": 45.2,
    "dataset_shape": [10000, 25],
}
```

Storage: `.joblib` files in `ml-datascience/models/` directory (Sprint 8 migrates to S3/GCS)

### Part 4.5 — API Routes

**File:** `api/routes/train.py`

```
POST /train
  Request:
    {
      "rows": [...],              # Dataset rows
      "columns": [...],           # Column names
      "target_column": "churn",   # Target (optional for clustering)
      "task_type": "auto",        # auto | classification | regression | clustering
      "algorithms": "auto",       # auto | ["xgboost_clf", "random_forest_clf", ...]
      "cv_folds": 5,              # Cross-validation folds
      "tune_trials": 50,          # Optuna trials per model
      "test_size": 0.2,           # Hold-out test ratio
      "model_id": "gpt-4o-mini"   # LLM model for AI narrative
    }
  Response:
    {
      "model_id": "uuid",
      "best_algorithm": "xgboost_clf",
      "best_algorithm_display": "XGBoost Classifier",
      "task_type": "classification",
      "target_column": "churn",
      "metrics": { "accuracy": 0.95, "f1": 0.94, "auc_roc": 0.98 },
      "comparison_table": [
        { "algorithm": "XGBoost", "accuracy": 0.95, "f1": 0.94, "auc": 0.98, "train_time": 2.3 },
        ...
      ],
      "charts": [
        { "name": "Confusion Matrix", "plotly_json": {...} },
        { "name": "ROC Curve", "plotly_json": {...} },
        { "name": "Feature Importance", "plotly_json": {...} },
        { "name": "Learning Curve", "plotly_json": {...} },
      ],
      "feature_importance": [
        { "feature": "balance", "importance": 0.23 },
        ...
      ],
      "classification_report": "...",
      "ai_summary": "The XGBoost model achieved the best performance...",
      "download_url": "/train/models/{model_id}/download"
    }

GET /train/models/{conversation_id}
  Response: List of trained models for the conversation

GET /train/models/{model_id}/download
  Response: .joblib file download
```

### Part 4.6 — Frontend: `/train` Slash Command

**Flow:**
```
1. User types "/train"
2. Frontend shows TrainConfigPanel (inline in chat):
   ┌──────────────────────────────────────────────┐
   │  Train a Model                                │
   │                                               │
   │  Target Column    [▼ dropdown of columns   ]  │
   │  Task Type        (●) Auto  ( ) Classification│
   │                   ( ) Regression  ( ) Cluster  │
   │  Algorithms       (●) Auto (AI picks best)    │
   │                   ( ) Select manually...       │
   │  CV Folds         [5  ]                       │
   │  Tuning Trials    [50 ]                       │
   │                                               │
   │             [ Start Training ]                │
   └──────────────────────────────────────────────┘

3. User clicks "Start Training"
4. Show TrainProgressCard with status updates
5. Display TrainResultCard when complete
```

**Natural language detection (via AI planner):**
- "train a model to predict price" → task=regression, target=price
- "เทรนโมเดลทำนาย churn" → task=classification, target=churn
- "classify customers into segments" → task=clustering
- "which algorithm is best for this data?" → run /train with auto

**Frontend files:**

| File | Purpose |
|------|---------|
| `components/TrainConfigPanel.tsx` | Training configuration form with dropdowns and radio buttons |
| `components/TrainResultCard.tsx` | Best model card + metrics + embedded Plotly charts + download |
| `components/ModelComparisonTable.tsx` | Algorithm comparison table (sortable by metric) |
| `components/TrainProgressCard.tsx` | Training progress with step indicator |
| `api/train/route.ts` | Next.js proxy to FastAPI `/train` |
| `api/train/models/route.ts` | Next.js proxy to GET trained models |
| `api/train/models/[id]/download/route.ts` | Next.js proxy for model file download |

**New types in `types.ts`:**

```typescript
interface TrainConfig {
  targetColumn: string;
  taskType: "auto" | "classification" | "regression" | "clustering";
  algorithms: string[] | "auto";
  cvFolds: number;
  tunTrials: number;
  testSize: number;
}

interface TrainModelMetrics {
  [key: string]: number;
  // classification: accuracy, precision, recall, f1, auc_roc, log_loss
  // regression: rmse, mae, r2, adjusted_r2, mape
  // clustering: silhouette, calinski_harabasz, davies_bouldin
}

interface TrainComparisonRow {
  algorithm: string;
  algorithmDisplay: string;
  metrics: TrainModelMetrics;
  trainTime: number;
  rank: number;
}

interface TrainFeatureImportance {
  feature: string;
  importance: number;
}

interface TrainResultArtifact {
  modelId: string;
  bestAlgorithm: string;
  bestAlgorithmDisplay: string;
  taskType: "classification" | "regression" | "clustering";
  targetColumn: string;
  metrics: TrainModelMetrics;
  comparisonTable: TrainComparisonRow[];
  charts: { name: string; plotly_json: object }[];
  featureImportance: TrainFeatureImportance[];
  classificationReport?: string;
  aiSummary: string;
  downloadUrl: string;
  datasetShape: [number, number];
  trainingDuration: number;
}

// Add to MessageArtifacts:
train_config?: TrainConfig;
train_result?: TrainResultArtifact;
```

### Sprint 4 Summary

| Metric | Value |
|--------|-------|
| New backend files | 5 (train_agent, model_registry, model_evaluator, model_storage, routes/train) |
| New frontend files | 6 (TrainConfigPanel, TrainResultCard, ModelComparisonTable, TrainProgressCard, 2 API routes) |
| Algorithms supported | 27 (12 classification + 11 regression + 4 clustering) |
| Evaluation charts | 12 (6 classification + 4 regression + 2 clustering) |
| New dependencies | `xgboost`, `lightgbm`, `optuna`, `shap` |

### Sprint 4 Acceptance Criteria

- [ ] `/train` command triggers TrainConfigPanel in chat
- [ ] "train model to predict X" routes through AI planner to training agent
- [ ] Auto task detection correctly identifies classification vs regression
- [ ] All 27 algorithms train successfully on sample datasets
- [ ] Optuna tuning produces measurably better results than defaults
- [ ] 5-fold cross-validation metrics shown in comparison table
- [ ] All evaluation charts render as interactive Plotly in TrainResultCard
- [ ] Model .joblib downloads successfully and loads in external Python
- [ ] Training on 10K-row dataset completes in < 5 minutes
- [ ] Multiple models can be trained in same conversation
- [ ] `tsc --noEmit` passes with zero errors

---

## Sprint 5: Model Prediction & Explainability

**Goal:** Users attach a new dataset and type `/predict` or "predict churn for this data". The system loads the trained model, applies the same preprocessing, generates predictions, and explains them — all in chat.

**Status:** 🔲 Not Started

### System Architecture

```
User: "/predict" or "ทำนาย churn ให้หน่อย" + attached dataset
  │
  ▼
┌──────────────────────────────────────────────────────┐
│  PredictAgent (api/agents/predict_agent.py)           │
│                                                       │
│  Step 1: RESOLVE MODEL                                │
│    - Find trained models in conversation              │
│    - If multiple → ask user to pick (ModelSelector)   │
│    - If one → use it automatically                    │
│                                                       │
│  Step 2: VALIDATE INPUT                               │
│    - Check new data has required feature columns      │
│    - Warn about missing/extra columns                 │
│    - Report schema differences                        │
│                                                       │
│  Step 3: PREPROCESS                                   │
│    - Load saved preprocessing pipeline                │
│    - Apply identical transforms to new data           │
│    - Handle unseen categories gracefully              │
│                                                       │
│  Step 4: PREDICT                                      │
│    - Run model.predict() on preprocessed data         │
│    - Get probability scores (classification)          │
│    - Get confidence intervals (regression, if avail)  │
│                                                       │
│  Step 5: BUILD OUTPUT                                 │
│    - Create predictions DataFrame                     │
│    - Add prediction + confidence columns              │
│    - Save as new dataset in conversation              │
│    - Generate summary statistics                      │
│    - Generate distribution chart                      │
│                                                       │
│  Step 6: RETURN                                       │
│    - PredictionResultCard artifact                    │
│    - Predictions saved as downloadable dataset        │
└──────────────────────────────────────────────────────┘
  │
  ▼
┌──────────────────────────────────────────────────────┐
│  PredictionResultCard (frontend)                      │
│                                                       │
│  ┌────────────────────────────────────────────────┐  │
│  │  Predictions Complete                          │  │
│  │  Model: XGBoost | Rows: 1,234                  │  │
│  │  Task: Classification | Target: churn           │  │
│  └────────────────────────────────────────────────┘  │
│                                                       │
│  ┌─ Preview ─────────────────────────────────────┐   │
│  │ ID | age | balance | ... | predicted | prob   │   │
│  │ 1  | 35  | 50000   | ... | Churn     | 0.87   │   │
│  │ 2  | 28  | 12000   | ... | No Churn  | 0.23   │   │
│  │ 3  | 45  | 80000   | ... | Churn     | 0.91   │   │
│  └───────────────────────────────────────────────┘   │
│                                                       │
│  Charts: [Prediction Distribution]                    │
│          [Confidence Histogram]                       │
│                                                       │
│  Actions: [Export CSV] [Export Excel]                  │
│           [Explain Predictions]                       │
└──────────────────────────────────────────────────────┘
```

### Part 5.1 — Prediction Engine

**File:** `api/agents/predict_agent.py`

Core logic:
- Load model + pipeline from `model_storage`
- Validate schema compatibility between training data and new data
- Apply preprocessing pipeline (same transforms, same order)
- Handle edge cases: unseen categories → "unknown" fallback, missing columns → error with clear message
- For classification: return predicted class + probability for each class
- For regression: return predicted value
- Save predictions as new dataset with `source: "prediction"`

### Part 5.2 — Model Explainability

**File:** `api/agents/model_explainer.py`

**Global explanations (whole dataset):**
- SHAP summary plot — beeswarm showing feature impact direction + magnitude
- SHAP bar plot — mean absolute SHAP values (feature importance)
- Permutation importance — model-agnostic importance scores

**Local explanations (single prediction):**
- SHAP waterfall — how each feature pushed prediction from base value
- SHAP force plot — compact single-row explanation
- "Why did the model predict row 5 as Churn?" → waterfall chart + natural language

**New dependency:** `shap>=0.45`

### Part 5.3 — `/predict` Slash Command

**Flow:**
```
1. User types "/predict"
2. If no trained model exists → show message "Train a model first with /train"
3. If one model exists → auto-select it
4. If multiple models → show ModelSelectorPanel:
   ┌──────────────────────────────────────────┐
   │  Select Model                             │
   │                                           │
   │  ● XGBoost (F1=0.94, trained 2h ago)     │
   │  ○ RandomForest (F1=0.92, trained 1d ago)│
   │                                           │
   │  Dataset: [currently attached dataset]    │
   │              [ Predict ]                  │
   └──────────────────────────────────────────┘
5. Run prediction
6. Display PredictionResultCard

User: "explain why row 5 was predicted as fraud"
  → ModelExplainer generates SHAP waterfall for row 5
  → Display ExplainCard with chart + natural language explanation
```

### Part 5.4 — Frontend Components

| File | Purpose |
|------|---------|
| `components/PredictionResultCard.tsx` | Predictions table + distribution chart + export buttons |
| `components/ModelSelectorPanel.tsx` | Choose which trained model to use |
| `components/ModelExplainCard.tsx` | SHAP plots + natural language explanation |
| `api/predict/route.ts` | Next.js proxy to FastAPI `/predict` |
| `api/predict/explain/route.ts` | Next.js proxy to FastAPI `/explain` |

**New types:**

```typescript
interface PredictionResultArtifact {
  modelId: string;
  modelName: string;
  taskType: "classification" | "regression";
  targetColumn: string;
  rowCount: number;
  predictions: Record<string, unknown>[];   // preview rows with predictions
  predictionColumn: string;                  // "predicted_churn"
  confidenceColumn?: string;                 // "confidence" or "probability"
  distributionChart: object;                 // Plotly JSON
  summary: {
    totalRows: number;
    classDistribution?: Record<string, number>;   // classification
    meanPrediction?: number;                       // regression
    stdPrediction?: number;                        // regression
  };
  datasetId: string;                         // saved predictions dataset
  aiSummary: string;
}

interface ExplainArtifact {
  modelId: string;
  explanationType: "global" | "local";
  shapSummaryChart?: object;        // Plotly JSON
  shapWaterfallChart?: object;      // Plotly JSON for single row
  permutationImportance?: { feature: string; importance: number; std: number }[];
  naturalLanguage: string;          // AI-generated explanation
}

// Add to MessageArtifacts:
prediction_result?: PredictionResultArtifact;
explain_result?: ExplainArtifact;
```

### Part 5.5 — Chat-Driven Intelligence

The AI planner should detect prediction and explanation intents:

| User Message | Detected Intent | Action |
|-------------|-----------------|--------|
| "/predict" | predict command | Show model selector → run prediction |
| "predict churn for this data" | predict intent | Auto-find model → predict |
| "ทำนายราคาให้หน่อย" | predict intent (Thai) | Auto-find regression model → predict |
| "explain row 5" | explain intent | SHAP waterfall for row 5 |
| "why is this customer predicted as churn?" | explain intent | SHAP waterfall + narrative |
| "which features matter most?" | global explain | SHAP summary + permutation importance |
| "show feature importance" | global explain | Feature importance chart |

### Part 5.6 — API Routes

```
POST /predict
  Request:
    {
      "rows": [...],                    # New data rows
      "columns": [...],                 # Column names
      "model_id": "uuid",              # Trained model to use
    }
  Response:
    {
      "predictions": [...],             # Rows with prediction columns added
      "prediction_column": "predicted_churn",
      "confidence_column": "probability",
      "summary": {...},
      "distribution_chart": {...},       # Plotly JSON
      "ai_summary": "The model predicted 234 (19%) customers as likely to churn..."
    }

POST /explain
  Request:
    {
      "model_id": "uuid",
      "rows": [...],                    # Data to explain
      "row_index": 5,                   # Optional: specific row to explain
      "explanation_type": "global"      # global | local
    }
  Response:
    {
      "shap_summary_chart": {...},       # Plotly JSON
      "shap_waterfall_chart": {...},     # Plotly JSON (if local)
      "permutation_importance": [...],
      "natural_language": "Row 5 was predicted as Churn primarily because..."
    }
```

### Sprint 5 Summary

| Metric | Value |
|--------|-------|
| New backend files | 3 (predict_agent, model_explainer, routes/predict) |
| New frontend files | 5 (PredictionResultCard, ModelSelectorPanel, ModelExplainCard, 2 API routes) |
| Explanation types | 2 (global SHAP summary, local SHAP waterfall) |
| New dependencies | `shap` |

### Sprint 5 Acceptance Criteria

- [ ] `/predict` loads model and generates predictions on new dataset
- [ ] Predictions saved as new dataset in conversation (exportable)
- [ ] Schema validation catches missing columns with clear error message
- [ ] Probability/confidence scores shown for classification predictions
- [ ] Prediction distribution chart renders correctly
- [ ] SHAP summary plot works for global explanation
- [ ] SHAP waterfall works for individual row explanation
- [ ] "explain row N" routes through AI planner to explainer
- [ ] Natural language explanation generated for each SHAP analysis
- [ ] Export predictions as CSV/Excel works
- [ ] `tsc --noEmit` passes with zero errors

---

## Sprint 6: Automated Tests

**Goal:** Comprehensive test coverage for all backend routes, agents, and frontend hooks. CI/CD pipeline that blocks PRs below 80% coverage.

**Status:** 🔲 Not Started

### Backend Tests (pytest)

| Test File | What It Tests |
|-----------|---------------|
| `tests/test_routes_chat.py` | POST /chat — routing, response format, error handling |
| `tests/test_routes_train.py` | POST /train — full training pipeline, model saving |
| `tests/test_routes_predict.py` | POST /predict — prediction, schema validation |
| `tests/test_routes_prepare.py` | POST /prepare — PrepConfig pipeline |
| `tests/test_routes_insights.py` | POST /insights — insights generation |
| `tests/test_routes_documents.py` | POST /documents — document generation |
| `tests/test_train_agent.py` | TrainAgent — task detection, algorithm selection, CV |
| `tests/test_predict_agent.py` | PredictAgent — model loading, preprocessing, prediction |
| `tests/test_model_evaluator.py` | Metrics calculation, chart generation |
| `tests/test_model_storage.py` | Save/load models, metadata |
| `tests/test_planner.py` | Two-stage routing, handler selection |
| `tests/test_sandbox.py` | Sandboxed exec — allowed/blocked imports |
| `tests/test_handlers_sample.py` | Sample test for each handler category (7 tests) |

### Frontend Tests (vitest)

| Test File | What It Tests |
|-----------|---------------|
| `__tests__/api/train.test.ts` | Train API route proxy |
| `__tests__/api/predict.test.ts` | Predict API route proxy |
| `__tests__/hooks/useChatPage.test.ts` | Slash commands, state management |
| `__tests__/components/TrainResultCard.test.tsx` | Train result rendering |
| `__tests__/components/PredictionResultCard.test.tsx` | Prediction rendering |

### CI/CD

```yaml
# .github/workflows/ci.yml
- Run pytest with coverage (fail < 80%)
- Run vitest with coverage (fail < 80%)
- Run tsc --noEmit (fail on type errors)
- Run ruff check (fail on lint errors)
```

### Sprint 6 Acceptance Criteria

- [ ] pytest coverage >= 80% for backend
- [ ] vitest coverage >= 80% for frontend
- [ ] GitHub Actions CI runs on every PR
- [ ] All existing features still pass after test infrastructure added
- [ ] Test fixtures include sample datasets (small CSV files)

---

## Sprint 7: Production Hardening

**Goal:** Security, rate limiting, resource limits, and error handling ready for public deployment.

**Status:** 🔲 Not Started

### Security & Limits

| Item | Implementation |
|------|---------------|
| Rate limiting | `slowapi` — 60 req/min per user for chat, 10 req/min for train |
| Upload cap | 50MB max file size (enforced in Next.js + FastAPI) |
| Sandbox hardening | Import blocklist (`os`, `subprocess`, `shutil`, `socket`), 30s timeout |
| CORS lockdown | Allow only production domain + localhost |
| Model size limit | 500MB max per saved model |
| Training timeout | 10 minute max training time |
| Input validation | Pydantic strict mode on all request models |
| Error format | Structured JSON errors on all routes: `{ "error": "...", "code": "..." }` |
| Secrets audit | Verify zero secrets in git history |

### Training Job Queue

For long-running training jobs:
- Background task with `asyncio` or Celery
- WebSocket or polling for progress updates
- Graceful cancellation support
- Training status: queued → running → completed/failed

### Sprint 7 Acceptance Criteria

- [ ] Rate limiting blocks excessive requests with 429 status
- [ ] Large file upload rejected with clear error message
- [ ] Sandbox blocks `import os`, `import subprocess`
- [ ] CORS rejects requests from unauthorized origins
- [ ] Training job runs in background, progress queryable
- [ ] All error responses are structured JSON

---

## Sprint 8: Production Deployment

**Goal:** Deploy PrepPilot to cloud with production database, file storage, and monitoring.

**Status:** 🔲 Not Started

### Infrastructure

| Component | Production |
|-----------|-----------|
| Database | MongoDB Atlas (migrate from SQLite) |
| File Storage | AWS S3 or GCP Cloud Storage (datasets + models) |
| Docker | Multi-stage build, optimized images |
| Hosting | GCP Cloud Run or AWS ECS |
| CDN | CloudFront or Cloud CDN for static assets |
| Monitoring | Sentry for errors, Prometheus + Grafana for metrics |
| Logging | Structured JSON logs → CloudWatch or Cloud Logging |

### Migration Checklist

- [ ] MongoDB Atlas cluster provisioned
- [ ] Prisma schema updated for MongoDB provider
- [ ] S3/GCS bucket created for file storage
- [ ] Dataset upload/download routes updated for object storage
- [ ] Model storage routes updated for object storage
- [ ] Docker images built and pushed to registry
- [ ] Environment variables configured in cloud
- [ ] SSL/TLS configured
- [ ] Domain name pointed to cloud service
- [ ] Health check endpoints verified
- [ ] Load testing passed (100 concurrent users)

---

## Complete Slash Command Reference (After Sprint 5)

| Command | Sprint | Purpose | Output |
|---------|--------|---------|--------|
| `/cleaning` | 2.7 ✅ | AI auto-clean dataset | Cleaned dataset + report |
| `/ml-prepare` | 2.7 ✅ | Auto ML data preparation | X_train, X_test, y_train, y_test folder |
| `/insights` | 2.95 ✅ | Technical analysis + weaknesses + action plan | InsightsReportCard |
| `/report` | 2.95 ✅ | Business EDA + use cases + market analysis | DocumentReportCard |
| `/train` | 4 🔲 | Auto ML model training + evaluation | TrainResultCard |
| `/predict` | 5 🔲 | Predict with trained model on new data | PredictionResultCard |

## End-to-End User Journey (After Sprint 5)

```
1. Upload dataset (drag & drop, 20+ formats)
      ↓
2. /insights → Understand data quality, weaknesses, what analyses are possible
      ↓
3. /cleaning → AI auto-cleans the data (nulls, duplicates, types)
      ↓
4. /report → Full business EDA report with charts and strategies
      ↓
5. /ml-prepare → Prepare train/test split with proper encoding/scaling
      ↓
6. /train → AI trains 5-8 models, tunes top 3, picks best → download .joblib
      ↓
7. Upload new dataset → /predict → Predictions in chat, export as CSV
      ↓
8. "explain why row 5 is predicted as churn?" → SHAP waterfall + explanation
```

## Dependency Summary

| Sprint | New Python Packages |
|--------|-------------------|
| 3 | `imbalanced-learn>=0.12`, `boruta>=0.3` |
| 4 | `xgboost>=2.0`, `lightgbm>=4.0`, `optuna>=3.6` |
| 5 | `shap>=0.45` |

| Sprint | New npm Packages |
|--------|-----------------|
| 4 | (none — reuses existing Plotly + UI components) |
| 5 | (none) |
| 6 | `vitest`, `@testing-library/react`, `msw` |

---

## File Map — New Files by Sprint

### Sprint 3 (27 files)

```
api/handlers/clean/smote_oversample.py
api/handlers/clean/random_oversample.py
api/handlers/clean/random_undersample.py
api/handlers/clean/adasyn_oversample.py
api/handlers/clean/auto_dtype_infer.py
api/handlers/stats/class_weight_calc.py
api/handlers/feature/rfe_select.py
api/handlers/feature/mutual_info_select.py
api/handlers/feature/select_k_best.py
api/handlers/feature/variance_threshold.py
api/handlers/feature/correlation_select.py
api/handlers/feature/boruta_select.py
api/handlers/feature/lag_features.py
api/handlers/feature/rolling_window.py
api/handlers/feature/expanding_window.py
api/handlers/feature/diff_features.py
api/handlers/feature/seasonal_features.py
api/handlers/transform/target_encode.py
api/handlers/transform/frequency_encode.py
api/handlers/transform/leave_one_out_encode.py
api/handlers/transform/stratified_sample.py
api/handlers/transform/systematic_sample.py
api/handlers/transform/datetime_decompose.py
api/handlers/analysis/schema_validate.py
api/handlers/analysis/data_drift_detect.py
api/handlers/analysis/constraint_check.py
api/handlers/analysis/imbalance_report.py
```

### Sprint 4 (11 files)

```
api/agents/train_agent.py
api/agents/model_registry.py
api/agents/model_evaluator.py
api/agents/model_storage.py
api/routes/train.py
src/app/chatpage/components/TrainConfigPanel.tsx
src/app/chatpage/components/TrainResultCard.tsx
src/app/chatpage/components/ModelComparisonTable.tsx
src/app/chatpage/components/TrainProgressCard.tsx
src/app/api/train/route.ts
src/app/api/train/models/[id]/download/route.ts
```

### Sprint 5 (8 files)

```
api/agents/predict_agent.py
api/agents/model_explainer.py
api/routes/predict.py
src/app/chatpage/components/PredictionResultCard.tsx
src/app/chatpage/components/ModelSelectorPanel.tsx
src/app/chatpage/components/ModelExplainCard.tsx
src/app/api/predict/route.ts
src/app/api/predict/explain/route.ts
```

### Sprint 6 (18+ test files)

```
ml-datascience/tests/test_routes_chat.py
ml-datascience/tests/test_routes_train.py
ml-datascience/tests/test_routes_predict.py
ml-datascience/tests/test_routes_prepare.py
ml-datascience/tests/test_routes_insights.py
ml-datascience/tests/test_routes_documents.py
ml-datascience/tests/test_train_agent.py
ml-datascience/tests/test_predict_agent.py
ml-datascience/tests/test_model_evaluator.py
ml-datascience/tests/test_model_storage.py
ml-datascience/tests/test_planner.py
ml-datascience/tests/test_sandbox.py
ml-datascience/tests/test_handlers_sample.py
ml-datascience/tests/fixtures/sample_data.csv
web-app/__tests__/api/train.test.ts
web-app/__tests__/api/predict.test.ts
web-app/__tests__/hooks/useChatPage.test.ts
web-app/__tests__/components/TrainResultCard.test.tsx
web-app/__tests__/components/PredictionResultCard.test.tsx
.github/workflows/ci.yml
```
