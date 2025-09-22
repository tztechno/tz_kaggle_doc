**焼きなまし法（Simulated Annealing）** と **ビームサーチ（Beam Search）** は、どちらも最適化問題や探索問題で使用されるアルゴリズムですが、アプローチと特性が大きく異なります。

## 焼きなまし法（Simulated Annealing）
**メタヒューリスティックな確率的探索アルゴリズム**

### 特徴：
- **単一の解**からスタート
- **温度パラメータ**を使用して探索を制御
- **悪い解へも一定確率で移動**（局所最適解脱出のため）
- 温度が下がるにつれて、探索が貪欲に（確率が低く）なる

### プロセス：
1. 初期解と高温からスタート
2. 近傍解を生成
3. 改善解なら必ず移動、悪化解でも確率で移動
4. 温度を徐々に下げる（冷却スケジュール）
5. 終了条件まで繰り返し

### 擬似コード：
```python
current_solution = initial_solution
temperature = high_temperature

while temperature > low_temperature:
    new_solution = get_neighbor(current_solution)
    cost_diff = cost(new_solution) - cost(current_solution)
    
    if cost_diff < 0 or random() < exp(-cost_diff / temperature):
        current_solution = new_solution
    
    temperature = cool(temperature)
```

## ビームサーチ（Beam Search）
**ヒューリスティックな並列探索アルゴリズム**

### 特徴：
- **複数の解**を並行して保持（ビーム幅 k）
- 各ステップで**最も有望なk個の解**だけを保持
- **確率的要素なし**（貪欲な選択）
- 幅優先探索のメモリ効率版

### プロセス：
1. ビーム幅 k を設定
2. 各ステップで、現在のk個の解から次の候補を生成
3. 生成された候補の中から最も良いk個を選択
4. 終了条件まで繰り返し

### 擬似コード：
```python
beam = [initial_solution]  # サイズkのリスト

while not termination_condition:
    candidates = []
    
    for solution in beam:
        candidates.extend(generate_next_steps(solution))
    
    # 評価値が高い順にソートし、上位k個を選択
    beam = sorted(candidates, key=score_function)[:k]
```

## 主な違い

| 特徴 | 焼きなまし法 | ビームサーチ |
|------|-------------|-------------|
| **探索方式** | 単一解ベース | 複数解ベース |
| **確率性** | あり（悪い解へ移動） | なし（貪欲選択） |
| **メモリ使用** | 少ない | ビーム幅に比例 |
| **局所最適** | 脱出可能 | 脱出困難 |
| **パラメータ** | 温度スケジュール | ビーム幅 |
| **適用分野** | 連続・離散最適化 | 系列生成、自然言語処理 |

## 具体的な使用例
- **焼きなまし法**: 巡回セールスマン問題、スケジューリング問題
- **ビームサーチ**: 機械翻訳、音声認識、構文解析

どちらも問題の性質に応じて選択される、強力な探索アルゴリズムです。
