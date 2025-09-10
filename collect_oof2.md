---

OOF予測を収集する際に考慮すべき重要なポイント¶
学習中にテストデータを使用しないこと

OOF予測は、学習フォールドからのみ取得する必要があります。
テストデータを間接的（例えば、データ漏洩など）であっても使用すると、OOFは無効になります。
適切なフォールド分離を維持すること

各フォールドは、そのフォールドの学習部分のみを使用する必要があります。
あるフォールド内の検証データは、そのフォールドの学習に使用してはなりません。
一貫した前処理

スケーリング、エンコード、または変換は、学習フォールドにのみ適用する必要があります。
適合した変換を検証フォールドに適用し、再適合は行わないでください。
スタッキングの前にOOF予測を収集すること

スタッキングの特徴量としてOOFを使用する場合は、まずOOFを収集してください。
下流のモデルやアンサンブルステップからのデータ漏洩がないことを確認してください。
ランダム性を慎重に扱うこと

OOFを再現可能にするために、可能な限りランダムシードを設定してください。
確率的アルゴリズムは一貫性のないOOF値を生成する可能性があるため、注意が必要です。
形状とアライメントを監視する

最終的なOOF配列は、トレーニングセットと同じ長さである必要があります。
各予測は、正しいトレーニングサンプルに対応している必要があります。
OOFのメトリクスを確認する

メトリクス（RMSE、loglossなど）を計算して、OOF予測を検証します。
これにより、未知のデータに対するモデルのパフォーマンスを信頼性の高い推定値として得ることができます。
グループ化されたデータまたは時系列データに注意する

フォールドがグループ化されている場合、または時間ベースである場合は、漏れを防ぐためにグループ化を尊重してください。
これらのシナリオでは、標準的なKフォールドは適切ではない可能性があります。
OOFの収集と漏れ/重要度のチェック
1. OOF予測を収集するための重要なポイント
厳密なフォールド分離：各フォールドの検証セットは、そのフォールドのトレーニングに使用してはなりません。
適切な前処理：スケーラー、エンコーダー、またはその他の変換は、各フォールドのトレーニング部分にのみ適用します。再フィッティングせずに検証に適用します。
スタッキング前にOOFを収集：リークを避けるため、OOF予測値はアンサンブルの特徴量として使用する前に取得する必要があります。
アライメントと形状を確認：OOF配列は、長さと順序がトレーニングセットと一致する必要があります。
再現性：ランダムシードを設定し、確率的効果を監視します。
グループ化または時系列制約を尊重：該当する場合は、グループ化されたフォールドまたは時間ベースのフォールドを使用します。
2. 潜在的にリークするOOFの検出
OOFとフルトレーニング予測値を比較：過度の類似性はリークを示している可能性があります。
異常に高いOOFメトリクス：極端に低い誤差またはほぼ完璧な精度は疑わしいものです。
特徴量とターゲットの相関：検証フォールドで予想外に高い相関は、リークを示している可能性があります。
順列/シャッフルテスト：ターゲットをシャッフルするとパフォーマンスが低下するはずです。そうでない場合は、リークが存在します。
フォールドの一貫性チェック：フォールド間でパフォーマンスに大きな差がある場合は、問題のある分割を示している可能性があります。
視覚的な検査：OOFと真のターゲットをプロットします。「完璧すぎる」パターンは危険信号です。
3. OOFを用いた特徴量の重要度チェック
OOFベースの重要度とフルトレイン全体の重要度：フォールドごとの重要度とフルトレイン全体の重要度を比較します。異常な値はリーク（漏れ）の兆候となる可能性があります。
異常に強いシグナルの確認：OOFで予測力が強すぎるように見える特徴量、特に弱い特徴量は、間接的なターゲットリークを反映している可能性があります。
クロスフォールドの一貫性：真の重要度は、フォールド間でほぼ一貫している必要があります。
OOFメトリクスとの組み合わせ：高いOOFパフォーマンスと歪んだ重要度は、リークの強力な指標です。
はじめに
このノートブックでは、Out-of-Fold（OOF）予測を、BPM予測のための元のデータセットに加えて追加の特徴量として使用する方法について調査します。私たちの主な目標は、オリジナルの特徴量とOOFから得られた特徴量の両方について、その重要度を評価することです。これにより、以下のことが可能になります。

モデル予測に最も貢献する特徴量を特定する。

OOF予測における潜在的な漏れを検出する。

OOF特徴量のスタッキングやアンサンブルでの使用が信頼できるかどうかを評価する。
OOF予測をフォールドワイズで計算し、オリジナルの特徴量と組み合わせ、LightGBMモデルを学習させます。そして、下流のスタッキングを行う前の健全性チェックとして、得られた特徴量の重要度を可視化します。

---


Key Points to Consider When Collecting OOF Predictions¶
Never Use Test Data During Training

OOF predictions must come strictly from training folds.
Using test data even indirectly (e.g., via leakage) invalidates the OOF.
Maintain Proper Fold Separation

Each fold should only use the training portion of that fold.
Validation data in a fold must not be used for training in that fold.
Consistent Preprocessing

Any scaling, encoding, or transformation must be fitted only on the training fold.
Apply the fitted transformation to the validation fold without refitting.
Collect OOF Predictions Before Any Stacking

If you plan to use OOF as features in stacking, collect them first.
Ensure no data leakage from downstream models or ensemble steps.
Handle Randomness Carefully

Set random seeds where possible to make OOF reproducible.
Be cautious of stochastic algorithms that might give inconsistent OOF values.
Monitor Shape and Alignment

The final OOF array must have the same length as the training set.
Each prediction must correspond to the correct training sample.
Check Metrics on OOF

Validate your OOF predictions by computing metrics (e.g., RMSE, logloss).
This gives a reliable estimate of model performance on unseen data.
Be Aware of Grouped or Time Series Data

If folds are grouped or time-based, respect the grouping to prevent leakage.
Standard K-Fold may not be appropriate in these scenarios.
OOF Collection and Leakage/Importance Check
1. Key Points for Collecting OOF Predictions
Strict fold separation: Each fold’s validation set must never be used in training for that fold.
Proper preprocessing: Fit scalers, encoders, or other transformations only on the training portion of each fold; apply them to validation without refitting.
Collect OOF before stacking: OOF predictions should be obtained before using them as features in ensembles to avoid leakage.
Check alignment and shape: OOF arrays must match the training set in length and order.
Reproducibility: Set random seeds and monitor stochastic effects.
Respect grouping or time-series constraints: Use grouped or time-based folds if applicable.
2. Detecting Potentially Leaky OOF
Compare OOF vs full-train predictions: Excessive similarity may indicate leakage.
Unusually high OOF metrics: Extremely low error or near-perfect accuracy is suspicious.
Feature-target correlations: Unexpectedly high correlations in validation folds may signal leakage.
Permutation/shuffle tests: Shuffling targets should degrade performance; if not, leakage exists.
Fold consistency check: Large performance differences across folds can indicate a problematic split.
Visual inspection: Plot OOF vs true target; “too perfect” patterns are a red flag.
3. Feature Importance Check Using OOF
OOF-based importance vs full-train importance: Compare per-fold importances with full-train importances; anomalies can signal leakage.
Check unusually strong signals: Features that appear too predictive in OOF, especially weak features, may reflect indirect target leakage.
Cross-fold consistency: Genuine importance should be roughly consistent across folds.
Combine with OOF metrics: High OOF performance + distorted importance is a strong indicator of leakage.
Introduction
This notebook investigates how Out-of-Fold (OOF) predictions can be used as additional features alongside the original dataset for BPM prediction. Our main goal is to assess the feature importance of both original and OOF-derived features to:

Identify which features contribute most to model predictions,
Detect potential leakage in OOF predictions, and
Evaluate whether stacking or using OOF features in ensembles is reliable.
We will compute OOF predictions fold-wise, combine them with original features, train LightGBM models, and visualize the resulting feature importances as a sanity check before any downstream stacking.



---
