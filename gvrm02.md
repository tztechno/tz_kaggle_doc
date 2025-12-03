なるほど、JS のファイル構造が見えました。これで PLY / GVRM の処理の流れを追いやすくなります。
ポイントは以下です：
1. main.js
    * ここで UI（ボタンやファイル入力）を監視
    * 選択された .ply / .gvrm を gsPath / gvrmPath にセット
    * その後 setupPathsFromUrlOrSelect() で外部関数を呼ぶ
2. 外部連携
    * preprocess/preprocess.js
        * .ply ファイルはここで処理され、GVRM に変換される
        * preprocess(vrmPath, gsPath, ...) が呼ばれ、戻り値として gvrm オブジェクトと promise2 を返す
    * gvrm-format/gvrm.js
        * .gvrm ファイルの読み込みとアバター生成
        * GVRM.load(gvrmPath, scene, camera, renderer, fileName) がここで実行される
    * gvrm-format/ply.js
        * PLY 形式の読み込みや変換に関わる可能性が高い
3. 流れ（簡略）
ボタン押下 / ファイル選択
        │
        ▼
main.js: fileInput change / select-button click
        │
        ▼
setupPathsFromUrlOrSelect()
        │
        ├─ PLY → preprocess(preprocess.js) → gvrm オブジェクト生成
        └─ GVRM → GVRM.load(gvrm.js) → gvrm オブジェクト生成
        │
        ▼
gvrm.changeFBX(Idle.fbx)
        │
        ▼
表示 / 操作可能

もし希望なら、私は この 28 ファイルの中で PLY / GVRM に直接関わる関数と呼び出し元をマッピングしたフロー図 を作れます。
作りますか？
