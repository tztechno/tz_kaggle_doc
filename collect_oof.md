「ノートを改造してOOFを取れるようにする」ための **最低限のコードパターン集** を整理します。
基本的には「クロスバリデーションのどこに `oof_preds[val_idx] = ...` を入れるか？」が鍵です。

---

# 🔑 OOF改造パターン集

## ① KFold / StratifiedKFold を使っている場合

よくある典型パターンです。

```python
from sklearn.model_selection import KFold
from sklearn.metrics import mean_squared_error
import numpy as np

kf = KFold(n_splits=5, shuffle=True, random_state=42)
oof_preds = np.zeros(len(train_x))

for fold, (train_idx, val_idx) in enumerate(kf.split(train_x, train_y)):
    model = lgb.LGBMRegressor()
    model.fit(train_x.iloc[train_idx], train_y.iloc[train_idx])
    val_pred = model.predict(train_x.iloc[val_idx])
    
    oof_preds[val_idx] = val_pred
    print(f"Fold {fold+1} RMSE:", mean_squared_error(train_y.iloc[val_idx], val_pred, squared=False))

np.save("oof.npy", oof_preds)
```

### ポイント

* `val_idx` に fold ごとのインデックスが入っている
* `oof_preds[val_idx] = val_pred` が **必須行**

---

## ② cross\_val\_score / cross\_val\_predict を使っている場合

Scikit-learn流に簡略化して書いているノートに遭遇することがあります。

```python
from sklearn.model_selection import cross_val_predict
from sklearn.linear_model import Ridge

model = Ridge()
oof_preds = cross_val_predict(model, train_x, train_y, cv=5)
np.save("oof.npy", oof_preds)
```

### ポイント

* `cross_val_predict` を使えば **一発でOOFが取れる**
* ただし LightGBM や XGBoost では使えないことが多い（scikit-learn APIで包んでいないと無理）

---

## ③ train\_test\_split を手作業でループしている場合

クロスバリデーションでなく「手書きで分割」しているノート。
この場合も考え方は同じです。

```python
from sklearn.model_selection import train_test_split

oof_preds = np.zeros(len(train_x))

for seed in [0, 1, 2, 3, 4]:
    X_train, X_val, y_train, y_val, idx_train, idx_val = train_test_split(
        train_x, train_y, np.arange(len(train_x)), test_size=0.2, random_state=seed
    )
    model = lgb.LGBMRegressor()
    model.fit(X_train, y_train)
    val_pred = model.predict(X_val)
    
    oof_preds[idx_val] = val_pred
```

### ポイント

* `train_test_split` はインデックスを返さないので、自分で作る必要がある
  → `np.arange(len(train_x))` を一緒にsplitして対応

---

## ④ TimeSeriesSplit（時系列CV）の場合

Kaggleでは時系列データでよく登場します。

```python
from sklearn.model_selection import TimeSeriesSplit

tscv = TimeSeriesSplit(n_splits=5)
oof_preds = np.zeros(len(train_x))

for fold, (train_idx, val_idx) in enumerate(tscv.split(train_x)):
    model = lgb.LGBMRegressor()
    model.fit(train_x.iloc[train_idx], train_y.iloc[train_idx])
    val_pred = model.predict(train_x.iloc[val_idx])
    
    oof_preds[val_idx] = val_pred
```

### ポイント

* `shuffle=True` が使えない
* 未来情報を漏らさないように「train < val」という順序が保証されている

---

# ✅ まとめ

* **必ずあるべき行** → `oof_preds[val_idx] = model.predict(X_val)`
* どのCV手法でも、**検証インデックスに予測を埋め込む**のが共通原則
* 最後に `np.save("oof.npy", oof_preds)` して収集できる形にする

---


