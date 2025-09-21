AutoGluon の `presets` に指定できる選択肢と、それぞれがどんな特徴を持っているかを以下にまとめます。使い分けるときの参考にどうぞ。

---

## ✅ Presets の選択肢一覧

AutoGluon の `TabularPredictor.fit(..., presets=…)` で使える既定のプリセットは以下です。 ([auto.gluon.ai][1])

| プリセット名                    | 説明（主な特徴）                                                                                       |
| ------------------------- | ---------------------------------------------------------------------------------------------- |
| `extreme_quality`         | 最も高精度を目指す設定。サンプル数が小さいデータセットで優れ、スタッキングや高度なハイパーパラメータ探査も含む。 GPU があると尚良い。 ([auto.gluon.ai][1])     |
| `best_quality`            | 精度重視。 inference（推論）時間やディスク使用量は重視しない。スタッキング・バギングなどを使ってモデルの性能を最大化。 ([auto.gluon.ai][1])          |
| `high_quality`            | `/ best_quality` よりは軽く、でもかなり精度を出したいとき。 inference を高速にする工夫あり。ディスク容量も少し抑える。 ([auto.gluon.ai][1]) |
| `good_quality`            | 実用的な精度と高速さのバランス重視。推論速度も考慮する。モデルサイズも moderate。 ([auto.gluon.ai][1])                             |
| `medium_quality`          | デフォルト設定。精度はそこそこ、でも訓練時間／推論時間を短くする。プロトタイピングや素早く結果を見たい時に向く。 ([auto.gluon.ai][1])                  |
| `experimental_quality`    | 新しい機能やモデル、未検証の技術を取り入れて試すプリセット。安定性より実験性重視。 ([auto.gluon.ai][1])                                 |
| `optimize_for_deployment` | 本番運用向け。不要なモデルやアーティファクトを落として、推論やディスク使用を最適化。機能の一部が制限されることあり。 ([auto.gluon.ai][1])                |
| `interpretable`           | 解釈可能なモデルだけを使う設定（例：ルールベース、決定木など）。モデルがブラックボックス寄りでなく、可視性／説明性が必要な場合に。 ([auto.gluon.ai][1])         |
| `ignore_text`             | テキスト特徴量を無視する設定。テキスト処理が重かったりノイズが予想されるとき。 ([auto.gluon.ai][1])                                   |

---

## ⚠️ 補足事項・使い分けのポイント

* 複数のプリセットを **リスト形式**で渡すことも可能です。例えば `presets=['good_quality', 'optimize_for_deployment']` のように。後のプリセットが前のものを上書きする設定もあります。 ([auto.gluon.ai][1])
* 「quality」系（精度を追うもの） のプリセットは相互に互換性が低いので、同時に複数の quality プリセットを混ぜるのはあまり推奨されていません。 ([auto.gluon.ai][1])
* デフォルトは `medium_quality`。プロトタイプや時間制限があるときはこちらが基本。 ([auto.gluon.ai][1])

---

もしよければ、あなたのデータ量（サンプル数、特徴量数）、利用可能な計算リソース（CPU / GPU / メモリ）、どのくらいの時間をかけたいか、などを教えてもらえれば、どのプリセットがベストか一緒に考えましょうか？

[1]: https://auto.gluon.ai/dev/api/autogluon.tabular.TabularPredictor.fit.html?utm_source=chatgpt.com "TabularPredictor.fit - AutoGluon 1.4.1 documentation"
