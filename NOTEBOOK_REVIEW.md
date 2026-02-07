# Notebook Review: Thapar_UCS654_2026_Hack_01.ipynb

## Overall assessment
The notebook provides a compact baseline pipeline (loading data, imputing, cross-validated model training, and submission export) but it is currently optimized for a quick leaderboard submission rather than a reliable, reproducible modeling workflow. Several methodological risks (split strategy, feature handling, and missing workflow steps) limit confidence in reported performance and the ability to iterate safely.

## What is working well
- Clear, minimal end-to-end flow from data load to submission export.
- Uses cross-validation and out-of-fold predictions instead of a single holdout.
- Employs a strong tree-based model (ExtraTrees) that can handle non-linearities.
- Uses median imputation to address missing numeric values.

## Key issues and risks
1. **Categorical encoding risk / LabelEncoder misuse**: The notebook assumes all features are numeric. If categorical columns exist, using `LabelEncoder` on features would be inappropriate because it imposes an arbitrary ordinal relationship; this can distort model behavior. Proper categorical handling is missing.
2. **Potential temporal leakage from random splits**: `StratifiedKFold(shuffle=True)` can leak future information if the data is time-ordered or grouped by entity/time. Without time-aware or group-aware splits, performance estimates can be overly optimistic.
3. **Incomplete modeling workflow**: There is no baseline model for comparison, no hyperparameter search, no calibration or error analysis, and no feature importance review. This limits learning about failure modes and model reliability.
4. **Colab path and execution reproducibility**: The workflow uses `files.upload()`/`files.download()` which is interactive and not reproducible in non-Colab environments. Paths and data sources are not parameterized.
5. **Missing feature engineering**: No feature transformations, encoding, or domain-driven features are explored. If the dataset includes categorical or time-related fields, this likely leaves performance on the table.

## Prioritized recommendations
1. **Fix feature handling with a proper preprocessing pipeline**
   - Use `ColumnTransformer` with `OneHotEncoder(handle_unknown="ignore")` for categorical features and `SimpleImputer` for numerics.
   - Avoid `LabelEncoder` for feature columns; reserve it for target labels only (if needed).
2. **Use time-aware or group-aware validation where appropriate**
   - If the data is temporal: use `TimeSeriesSplit` or a custom time-based split.
   - If there are grouped entities (e.g., users, sessions): use `GroupKFold`.
3. **Add baseline models and a minimal evaluation report**
   - Train a simple baseline (e.g., logistic regression or majority class) to establish a floor.
   - Report fold-wise scores and variance to understand stability.
4. **Enhance reproducibility**
   - Replace `files.upload()` with parameterized file paths or a data download step.
   - Save metrics and model artifacts (e.g., `joblib.dump`) with versioned outputs.
5. **Iterate on feature engineering and model selection**
   - Add basic transformations (e.g., counts, date parts, interactions).
   - Compare tree-based models (RandomForest, XGBoost/LightGBM if allowed) against baseline.

## Quick scorecard (out of 5)
- **Clarity & organization**: 3
- **Methodological soundness**: 2
- **Reproducibility**: 2
- **Feature engineering**: 1
- **Validation strategy**: 2

**Overall score**: 2/5
