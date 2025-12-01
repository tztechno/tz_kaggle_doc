リポジトリを確認しました。ファイル間の情報フローを解析します。リポジトリの構造を確認できました。提供いただいた`preprocess.js`とGitHubの情報から、**ファイル間の情報フロー**を解析します。

## 📊 ファイル間のデータフロー図

```
[入力ファイル]
├─ VRM file (.vrm) ────────┐
└─ Gaussian Splat (.ply) ──┤
                           ↓
┌──────────────────────────────────────────┐
│         preprocess.js (今回の分析対象)      │
├──────────────────────────────────────────┤
│ 1. cleanSplats()                         │
│    ├─ PLYParser (ply.js)                │
│    └─ → 人物/背景分離                     │
│                                          │
│ 2. GVRM.initVRM() / initGS()            │
│    ├─ gvrm.js                           │
│    └─ → character, gs オブジェクト生成    │
│                                          │
│ 3. findBestAngleInRange()               │
│    ├─ PoseDetector (pose.js)           │
│    └─ → 最適カメラ角度検出                │
│                                          │
│ 4. GVRMUtils.getPointsMeshCapsules()    │
│    ├─ utils.js                          │
│    └─ → pmc, capsuleBoneIndex 生成      │
│                                          │
│ 5. assignSplatsToBones()                │
│    ├─ preprocess_gl.js (GPU版)         │
│    └─ → splatBoneIndices 生成           │
│                                          │
│ 6. assignSplatsToPoints()               │
│    ├─ preprocess_gl.js (GPU版)         │
│    └─ → splatVertexIndices 生成         │
│                                          │
│ 7. finalCheck()                         │
│    ├─ check.js                          │
│    └─ → 姿勢検証                         │
│                                          │
│ 8. gvrm.save()                          │
│    └─ → .gvrm ファイル出力               │
└──────────────────────────────────────────┘
                           ↓
                   [出力ファイル]
                   .gvrm (圧縮済みアバター)
```

## 🔄 主要データ構造の流れ

### **1. 初期化フェーズ**
```
VRM → GVRM.initVRM() → character {
  currentVrm: VRM object
  skinnedMeshIndex: number
  ground: number (床の高さ)
}

PLY → GVRM.initGS() → gs {
  viewer: GaussianSplats3D
  splatMesh: THREE.Mesh
  centers0: Float32Array (元の位置)
  colors: Uint8Array
  splatCount: number
}
```

### **2. クリーニングフェーズ**
```
PLY → PLYParser.parsePLY() → {
  vertices: Array<{x, y, z, ...}>
  header: Array<string>
}

↓ cleanSplats()

{
  urls: [人物PLY, 背景PLY]
  centroid: {x, z}
  centroidHead: {x, z}
  heights: {min, max}
  distXZ: number
}
```

### **3. ボーン構造生成**
```
character → GVRMUtils.getPointsMeshCapsules() → {
  pmc: {
    points: THREE.Points
    mesh: THREE.SkinnedMesh
    capsules: THREE.Group (各ボーンのカプセル)
  }
  capsuleBoneIndex: Map<capsuleIndex → boneIndex>
}
```

### **4. スプラット割り当て**
```
gs + pmc → assignSplatsToBones() → {
  gs.splatBoneIndices: Array<boneIndex>
}

↓

gs + character + pmc → assignSplatsToPoints() → {
  gs.splatVertexIndices: Array<vertexIndex>
  gs.splatRelativePoses: Array<x, y, z>
}
```

### **5. 最終出力**
```
GVRM {
  character: character
  gs: gs
  pmc: pmc
  modelScale: number
  boneOperations: Array
} → gvrm.save() → .gvrm file (ZIP)
```

## 📁 依存ファイルの役割

| ファイル | 主な役割 | 提供する機能 |
|---------|---------|------------|
| **gvrm.js** | コアクラス | GVRM.load/save, initVRM/initGS, update |
| **utils.js** | ユーティリティ | getPointsMeshCapsules, resetPose, visualizePMC |
| **ply.js** | PLY解析 | parsePLY, createPLYFile |
| **pose.js** | 姿勢検出 | PoseDetector (MediaPipe Pose) |
| **preprocess_gl.js** | GPU処理 | assignSplatsToBonesGL, assignSplatsToPointsGL |
| **check.js** | 検証 | finalCheck (膝位置等の検証) |

## 🔑 重要な情報の伝播パターン

1. **スケール情報**: `heights` → `vrmScale` → `character` → `gvrm.modelScale`
2. **位置情報**: `centroid` → `gsScene.position` → `gs`
3. **角度情報**: `findBestAngleInRange()` → `gsScene.rotation` → `gs`
4. **ボーン情報**: `capsuleBoneIndex` → `splatBoneIndices` → `splatVertexIndices`
5. **相対位置**: `splatRelativePoses` → GVRM → アニメーション時に使用

この構造により、各モジュールが独立しながらも、必要な情報が適切に伝播する設計になっています。
