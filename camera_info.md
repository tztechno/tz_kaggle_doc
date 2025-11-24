このテキストは、3つのマルチビューデータセット（Neural 3D Dataset, Technicolor Dataset, Google Immersive Dataset）のカメラ情報の管理方法をまとめたものです。以下にそれぞれのデータセットの特徴を整理します。

## 各データセットのカメラ情報管理方法

### 1. Neural 3D Dataset
**特徴**：
- COLMAPを使用した構造復元ベース
- 各フレームごとに`colmap_<番号>`ディレクトリを作成
- 時系列のカメラ姿勢情報を管理

**ディレクトリ構造**：
```
<location>
└── cook_spinach
    ├── colmap_0
    ├── colmap_1
    ├── ...
    └── colmap_299
```

### 2. Technicolor Dataset
**特徴**：
- 事前に歪み補正済みの画像を使用
- ファイル名にカメラIDとフレーム番号を含む（例：`Fabien_undist_<00257>_<08>.png`）
- 静的なカメラ配置（光フィールドビデオ）

**ディレクトリ構造**：
```
<location>
└── Fabien
    ├── Fabien_undist_00257_08.png
    ├── Fabien_undist_00257_09.png
    └── ...
```

### 3. Google Immersive Dataset
**特徴**：
- **3段階の処理パイプライン**：
  1. 元の動画ファイル（`camera_0001.mp4`）
  2. 歪み補正前（`*_dist`）
  3. 歪み補正後（`*_undist`）

**ディレクトリ構造**：
```
<location>
└── 02_Flames
    ├── 02_Flames/              # 元の動画
    │   ├── camera_0001.mp4
    │   └── camera_0002.mp4
    ├── 02_Flames_dist/         # 歪み補正前
    │   ├── colmap_0
    │   └── ...
    └── 02_Flames_undist/       # 歪み補正後
        ├── colmap_0
        └── ...
```

## 共通点と相違点

**共通点**：
- すべてCOLMAPを使用してカメラ姿勢を推定
- シーンごとにディレクトリを分離
- アクティベーション環境は`colmapenv`

**相違点**：
- **入力形式**：Neural 3Dは動画、Technicolorは画像、Immersiveは動画
- **歪み補正**：Technicolorは事前補正、Immersiveは補正前後の両方を保持
- **カメラ配置**：Technicolorは静的なマルチカメラ、他は動的シーン

この整理から、各データセットが異なるユースケース（動的シーン vs 静的光フィールド）と処理パイプラインに対応していることがわかります。
