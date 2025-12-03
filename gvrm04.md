了解です。では、先ほど整理したフローをさらに細かく、関数呼び出しチェーンとして図示します。ここでは .ply → GVRM 生成の場合を中心に追います。

PLY → GVRM 関数呼び出しチェーン
[UI: file-input or select-sample-button]
        │
        ▼
main.js → setupPathsFromUrlOrSelect()
        │
        ├─ fileInput change / selectSampleButton click
        │        └─ 選択ファイルを gsPath / gvrmPath にセット
        │
        ▼
gsPath が存在
        │
        └─ preprocess(vrmPath, gsPath, scene, camera, renderer, stage, useGPU, noBG, noCheck, fileName, savePly)
               (apps/preprocess/preprocess.js)
               │
               ├─ readPLYFile(gsPath)
               │       (gvrm-format/ply.js)
               │       └─ PLY ファイルをパースして頂点・法線・色情報を取得
               │
               ├─ createGVRMFromPLY(parsedData)
               │       (preprocess.js 内)
               │       └─ 頂点・メッシュ情報を THREE.js のオブジェクトに変換
               │
               ├─ applyTexturesAndMaterials()
               │       (preprocess.js 内 / utils.js)
               │       └─ マテリアル・テクスチャ設定
               │
               └─ return { gvrm, promise2 }
                       │
                       └─ main.js で then() 内で受け取り
                               ├─ gvrm.changeFBX('./assets/Idle.fbx')
                               └─ updateStatusList()
                               └─ showControlButtons()

GVRM ファイル読み込みの場合 (.gvrm)
[UI: file-input or view-sample-avatars-button]
        │
        ▼
main.js → setupPathsFromUrlOrSelect()
        │
        ├─ fileInput change / viewSampleAvatarsButton click
        │        └─ gvrmPath にセット
        │
        ▼
gvrmPath が存在
        │
        └─ GVRM.load(gvrmPath, scene, camera, renderer, fileName)
               (gvrm-format/gvrm.js)
               │
               ├─ parseVRMFile(gvrmPath)
               │       └─ VRM データを THREE.js オブジェクトに変換
               │
               ├─ setupCharacter()
               │       └─ GVRM キャラクター初期化
               │
               └─ return gvrm
                       │
                       └─ main.js で then() 内で受け取り
                               ├─ gvrm.changeFBX('./assets/Idle.fbx')
                               └─ updateStatusList()
                               └─ showControlButtons()

💡 ポイントまとめ
1. main.js は「UI → ファイルパス判定 → 外部関数呼び出し」の仲介役。
2. PLY 処理は preprocess.js → ply.js → utils.js を経て GVRM オブジェクト化。
3. GVRM 読み込みは gvrm.js が VRM ファイルを THREE.js に変換。
4. 共通の後処理は
    * 初期 FBX アニメーション設定 (gvrm.changeFBX)
    * ステータスリスト更新
    * UI コントロールボタン表示
5. 非同期処理なので then() 内で gvrm が使える状態になる。

もし希望なら、私はこのチェーンを 「図式化した可視化図（矢印付き）」 にして、どのモジュールの関数がどこに流れるか一目でわかるように作ることもできます。
作りますか？
