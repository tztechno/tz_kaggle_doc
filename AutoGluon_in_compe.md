Here are effective strategies for using AutoGluon in competitions:

## 🚀 **Competition-Winning AutoGluon Strategies**

### 1. **Multi-Layered Time Strategy**
```python
# Phase 1: Quick exploration (30 min)
predictor1 = TabularPredictor(label='target').fit(
    train_data, time_limit=1800, presets='medium_quality'
)

# Phase 2: Refined training (2 hours)  
predictor2 = TabularPredictor(label='target').fit(
    train_data, time_limit=7200, presets='high_quality',
    hyperparameter_tune=True
)

# Phase 3: Final ensemble (4+ hours)
predictor3 = TabularPredictor(label='target').fit(
    train_data, time_limit=14400, presets='best_quality',
    auto_stack=True, num_bag_folds=5, num_stack_levels=2
)
```

### 2. **Advanced Feature Engineering + AutoGluon**
```python
# Create extensive features first, then let AutoGluon handle selection
feature_engineered_data = create_advanced_features(raw_data)

predictor = TabularPredictor(label='target').fit(
    feature_engineered_data,
    time_limit=10800,  # 3 hours
    presets='best_quality',
    feature_generator='auto',  # AutoGluon will select best features
    num_bag_folds=8,
    num_stack_levels=2
)
```

### 3. **Cross-Validation Optimization**
```python
from autogluon.core.utils import generate_train_test_split

# Custom CV strategy for competition data
predictor = TabularPredictor(label='target').fit(
    train_data,
    num_bag_folds=10,  # More folds for stable validation
    num_stack_levels=2,
    holdout_frac=0.1,  # Separate holdout for final stacking
    hyperparameters={
        'GBM': {'num_boost_round': 10000, 'early_stopping_rounds': 100},
        'CAT': {'iterations': 10000, 'early_stopping_rounds': 100},
        'XGB': {'n_estimators': 10000, 'early_stopping_rounds': 100}
    }
)
```

### 4. **Pseudo-Labeling Integration**
```python
# Step 1: Train initial model
predictor = TabularPredictor(label='target').fit(train_data, time_limit=7200)

# Step 2: Predict on test data with high confidence
test_predictions = predictor.predict_proba(test_data)
high_confidence_mask = (test_predictions.max(axis=1) > 0.95)  # 95% confidence threshold

# Step 3: Retrain with pseudo-labeled data
pseudo_labeled_data = create_pseudo_labels(test_data, high_confidence_mask)
augmented_train_data = pd.concat([train_data, pseudo_labeled_data])

# Step 4: Final training with expanded dataset
final_predictor = TabularPredictor(label='target').fit(
    augmented_train_data, time_limit=10800, presets='best_quality'
)
```

### 5. **Model Blending with External Models**
```python
# Train AutoGluon
autogluon_predictor = TabularPredictor(label='target').fit(
    train_data, time_limit=7200, presets='high_quality'
)

# Train manual models (LightGBM, XGBoost, etc.)
manual_models = train_manual_models(train_data)

# Blend predictions optimally
autogluon_proba = autogluon_predictor.predict_proba(test_data)
manual_proba = ensemble_manual_predictions(manual_models, test_data)

# Learn optimal blending weights (can use simple average or meta-model)
final_predictions = 0.7 * autogluon_proba + 0.3 * manual_proba  # Weighted blend
```

### 6. **Time-Series Competition Strategy**
```python
# For time-series competitions with tabular format
predictor = TabularPredictor(label='target').fit(
    train_data,
    time_limit=14400,  # 4 hours
    presets='best_quality',
    hyperparameters={
        'GBM': {'extra_trees': True, 'feature_fraction': 0.8},
        'CAT': {'l2_leaf_reg': 5, 'random_strength': 1},
    },
    num_bag_folds=5,
    time_limit=14400
)
```

### 7. **Memory and Speed Optimization**
```python
# For large datasets or memory constraints
predictor = TabularPredictor(label='target').fit(
    train_data,
    time_limit=3600,
    presets='optimize_for_deployment',  # Faster, lighter models
    hyperparameters={
        'GBM': {'num_leaves': 31, 'min_data_in_leaf': 20},
        'NN': {'num_epochs': 50, 'learning_rate': 1e-3},
    },
    ag_args_fit={'num_gpus': 1}  # Utilize GPU if available
)
```

## 🏆 **Pro Tips for Competitions:**

1. **Start with AutoGluon baseline immediately** - Get a strong benchmark quickly
2. **Use feature importance** from AutoGluon to guide manual feature engineering
3. **Experiment with different time limits** - Often 2-4 hours gives best ROI
4. **Combine with manual models** - AutoGluon + hand-tuned models often wins
5. **Monitor leaderboard during training** - Stop early if performance plateaus
6. **Use `refit_full=True`** for final submission to retrain on full data

## 📊 **Evaluation Strategy:**
```python
# Get detailed model analysis
leaderboard = predictor.leaderboard(extra_info=True)
best_model = predictor.get_model_best()

# Analyze feature importance
feature_importance = predictor.feature_importance(train_data)

# Cross-validate best model
cv_results = predictor.fit_summary()
```

AutoGluon provides an excellent foundation, and combining its automated power with competition-specific strategies often leads to top rankings!
