わかりました。では、あなたのプロジェクト構造に基づき、PLY / GVRM 処理の呼び出しフロー図を作ります。主要ファイルと関数の関係を整理します。

PLY / GVRM ファイル処理フロー
[UI: ボタン / file input]
        │
        ▼
main.js
├─ setupPathsFromUrlOrSelect()
│    ├─ fileInput change → 選択ファイル判定
│    │      ├─ 拡張子が .ply → gsPath にセット
│    │      └─ 拡張子が .gvrm → gvrmPath にセット
│    └─ selectSampleButton / viewSampleAvatarsButton
│           └─ サンプル選択 → gsPath / gvrmPath にセット
│
├─ gvrmPath がある場合
│     └─ GVRM.load(gvrmPath, scene, camera, renderer, fileName)
│            (gvrm-format/gvrm.js)
│            └─ gvrm オブジェクト生成 → scene に配置
│
└─ gsPath (.ply) がある場合
      └─ preprocess(vrmPath, gsPath, scene, camera, renderer, ...)
             (apps/preprocess/preprocess.js)
             ├─ PLY 読み込み (ply.js)
             ├─ GVRM 生成
             └─ gvrm オブジェクト返却
                     └─ gvrm.changeFBX(Idle.fbx)
                     └─ scene に配置

ファイルごとの役割
ファイル	主な役割
main.js	UI 管理、選択されたファイルの判定、GVRM / PLY 処理呼び出し
preprocess/preprocess.js	PLY を GVRM に変換するメイン処理。非同期処理もここで管理
gvrm-format/gvrm.js	GVRM 読み込みと生成。GVRM.load() がここ
gvrm-format/ply.js	PLY ファイルのパース・変換。preprocess.js 内で呼び出される
gvrm-format/utils.js / vrm.js / gs.js	補助関数、GVRM 内部処理、スプラットメッシュ生成等
処理のポイント
1. ファイル選択後の分岐
    * .ply → preprocess → gvrm 生成
    * .gvrm → GVRM.load → gvrm 生成
2. gvrm オブジェクト生成後
    * gvrm.changeFBX(Idle.fbx) で初期アニメーション
    * scene に配置され、レンダリング開始
3. 非同期
    * preprocess → promise を返す
    * .then() 内で gvrm を操作

💡 補足
* main.js は UI と呼び出しの仲介が主で、実際のファイル処理は外部モジュールに委任しています。
* つまり .ply / .gvrm の中身を追いたい場合は preprocess.js と gvrm.js / ply.js が重要です。

もしよければ、次のステップとして PLY → GVRM 変換の具体的な関数呼び出し順 をさらに細かく追跡した「関数呼び出しチェーン図」を作れます。
作りますか？
