了解です。では、先ほどの関数呼び出しチェーンを 矢印付きの可視化図 に変換します。ここでは PLY → GVRM と GVRM 読み込み 両方を示します。

flowchart TD
    %% UI
    A[UI: file-input / select-sample-button / view-sample-avatars-button] --> B[main.js: setupPathsFromUrlOrSelect()]

    %% 分岐: PLY または GVRM
    B -->|gsPath が存在| C[preprocess(vrmPath, gsPath, ...)]
    B -->|gvrmPath が存在| G[GVRM.load(gvrmPath, scene, camera, renderer, fileName)]

    %% PLY 処理チェーン
    C --> C1[ply.js: readPLYFile(gsPath)]
    C1 --> C2[preprocess.js: createGVRMFromPLY(parsedData)]
    C2 --> C3[preprocess.js/utils.js: applyTexturesAndMaterials()]
    C3 --> C4[return { gvrm, promise2 }]
    C4 --> D[main.js then(): gvrm.changeFBX('./assets/Idle.fbx')]
    D --> D1[updateStatusList()]
    D1 --> D2[showControlButtons()]

    %% GVRM ファイル読み込みチェーン
    G --> G1[gvrm.js: parseVRMFile(gvrmPath)]
    G1 --> G2[gvrm.js: setupCharacter()]
    G2 --> G3[return gvrm]
    G3 --> H[main.js then(): gvrm.changeFBX('./assets/Idle.fbx')]
    H --> H1[updateStatusList()]
    H1 --> H2[showControlButtons()]

💡 解説
* 左側が PLY → GVRM のフロー
* 右側が GVRM 読み込み のフロー
* main.js が UI と外部モジュールの橋渡し役になっていることがわかる
* gvrm.changeFBX() と UI 更新は両方のフローで共通

希望であれば、この図をさらに 「モジュール別に色分け」 して、どのファイルがどの処理を担うかも一目でわかるようにできます。
作りますか？
