```

AtCoder Library (ACL) には、競技プログラミングでよく使われる**高度なデータ構造とアルゴリズム**が多数含まれています。主なカテゴリをご紹介します：

## 1. 数学関連 (math)
- `floor_sum(n, m, a, b)` - 先ほどの関数
- `pow_mod(x, n, m)` - 冪乗のmod
- `inv_mod(x, m)` - 逆元
- `crt(r, m)` - 中国剰余定理
- `floor_sum(n, m, a, b)` - 床関数の和

## 2. データ構造 (dsu, fenwicktree, segtree, etc.)
- `dsu` - Union-Find (Disjoint Set Union)
- `fenwicktree` - Fenwick Tree (BIT)
- `segtree` - セグメント木
- `lazysegtree` - 遅延評価セグメント木
- `scc` - 強連結成分分解
- `twosat` - 2-SAT

## 3. グラフ関連 (graph)
- `mf_graph` - 最大流 (Dinic法)
- `mincostflow` - 最小費用流
- `scc_graph` - 強連結成分
- `maxflow` - 最大流

## 4. 文字列処理 (string)
- `suffix_array(s)` - 接尾辞配列
- `lcp_array(s, sa)` - LCP配列
- `z_algorithm(s)` - Z-algorithm

## 5. 多項式と畳み込み (convolution)
- `convolution(a, b)` - 畳み込み (FFT/NTT)
- `convolution_ll(a, b)` - 整数畳み込み

## 6. 幾何 (geometry)
- 点や線分の基本操作
- 凸包アルゴリズムなど

## 使用例

```python
from atcoder import *
from atcoder.math import *
from atcoder.dsu import *
from atcoder.fenwicktree import *
from atcoder.segtree import *

# 例: Union-Find
uf = DSU(100)
uf.merge(1, 2)
print(uf.same(1, 2))  # True

# 例: Fenwick Tree
fw = FenwickTree(int, 100)
fw.add(0, 10)
fw.add(1, 20)
print(fw.sum(0, 2))  # 30

# 例: セグメント木
def op(x, y): return max(x, y)
def e(): return 0
seg = SegTree(op, e, 100)
seg.set(0, 10)
seg.set(1, 20)
print(seg.prod(0, 2))  # 20
```

## 特徴

1. **高速で信頼性が高い** - 競技プログラミング向けに最適化
2. **C++版がベース** - パフォーマンス重視
3. **簡単に使用可能** - importするだけ
4. **ドキュメント充実** - 公式ドキュメントあり

AtCoderのコンテストではこれらのライブラリを自由に使用できるので、複雑なデータ構造を自作する必要がなく、問題解決に集中できます！

どのライブラリが使えるかは `from atcoder import *` して確認するか、[公式ドキュメント](https://atcoder.github.io/ac-library/production/document_ja/)を参照すると良いでしょう。


```
