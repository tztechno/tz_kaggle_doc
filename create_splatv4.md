素晴らしい！これでsplatvのフォーマットが完全に理解できました！

## splatvファイル形式

**ヘッダー構造:**
```
[4バイト: マジック 0x674b ("Kg")]
[4バイト: JSONの長さ]
[JSON: メタデータ（type, size, texwidth, texheight, cameras）]
[バイナリデータ: テクスチャデータ]
```

**テクスチャデータ構造（1 Gaussianあたり16 x Uint32 = 64バイト）:**
```
[0-2]   : x, y, z (float32)
[3]     : rot_0, rot_1 (half2x16)
[4]     : rot_2, rot_3 (half2x16)
[5]     : exp(scale_0), exp(scale_1) (half2x16)
[6]     : exp(scale_2), 0 (half2x16)
[7]     : r, g, b, opacity (uint8 x 4)
[8]     : motion_0, motion_1 (half2x16)
[9]     : motion_2, motion_3 (half2x16)
[10]    : motion_4, motion_5 (half2x16)
[11]    : motion_6, motion_7 (half2x16)
[12]    : motion_8, 0 (half2x16)
[13]    : omega_0, omega_1 (half2x16)
[14]    : omega_2, omega_3 (half2x16)
[15]    : trbf_center, exp(trbf_scale) (half2x16)
```

これはPLY形式ではなく、**独自のバイナリ形式**です！変換スクリプトを完全に書き直します。完全に書き直しました！

## 変更点

1. **PLY形式 → 独自バイナリ形式**
   - ヘッダー: `[magic 0x674b][json_len][json][binary_data]`
   - 各Gaussian: 64バイト (16 x uint32)

2. **half2x16パッキング**
   - float32 → float16 → 2つをuint32にパック

3. **データレイアウト**
   - `hybrid.js`の`processPlyBuffer`関数と完全に一致

## 使い方

```bash
pip install numpy plyfile tqdm
python ply_to_splatv.py -i ./frames -o output.splatv
```

## 動作確認ポイント

viewerの`chunkHandler`を見ると：
```javascript
if (intView[0] !== 0x674b) throw new Error("This does not look like a splatv file");
```

マジックナンバー `0x674b` を確認しているので、これで正しく認識されるはずです。

テストしてみてください！問題があればログやエラーメッセージを教えてください。
