---
title: "Theorem1 Proof"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookComments: false
# bookSearchExclude: false
---

[元記事](../_index.md)

補ラベルに対して、損失関数$\bar{L}(f(\mathbf{x}), y)$を考える。こんなに風に表せる。

$$
R(f) = (K - 1) \mathbb{E} _{\bar{p} (\mathbf{x}, \bar{y})} [\bar{L} (f(\mathbf{x}), \bar{y})] - M_1 + M_2
$$

$M _1, M _2$はここで、定数となるという仮定を行う。

$$
M_1 = \sum _{\bar{y} = 1}^{K} \bar{L}(f(\mathbf{x}), \bar{y}) 
$$

$M_1$は、$\mathbf{x}$について、**すべてのありえる補ラベルに対して、予測結果との損失関数$\bar{L}$の和**である。

$$
M_2 = \bar{L}(f(\mathbf{x}), y) + L(f(\mathbf{x}), y)
$$

証明は以下の通り。

## 証明の式変形

$$
(K - 1) \mathbb{E} _{\bar{p} (\mathbf{x}, \bar{y})} [\bar{L} (f(\mathbf{x}), \bar{y})] = (K - 1) \int \sum _{\bar{y} = 1} ^ {K} \bar{L} (f(\mathbf{x}), \bar{y}) p(\mathbf{x}, \bar{y}) d\mathbf{x}
$$

前に定義した、$\bar{p}(\mathbf{x}, \bar{y})$を代入する。

$$
= (K - 1) \int \sum _{\bar{y} = 1} ^ {K} \bar{L} (f(\mathbf{x}), \bar{y}) (\frac{1}{K - 1} \sum _{y \neq \bar{y}} p(\mathbf{x}, y)) d\mathbf{x}
$$

ここで、Σの2つの順番を、次のようにしてもいい。**全体をイテレーションしてることには変わりがないからだ**。

$$
= \int \sum _{y = 1} ^ {K} \sum _{\bar{y} \neq y} \bar{L} (f(\mathbf{x}), \bar{y}) p(\mathbf{x}, y) d\mathbf{x}
$$

よく見ると$p(\mathbf{x}, y)$についての期待値のかたちである。

$$
= \mathbb{E} _{p(\mathbf{x}, y)} [ \sum _{\bar{y} \neq y} \bar{L} (f(\mathbf{x}), \bar{y}) ] = \mathbb{E} _{p(\mathbf{x}, y)} [ M_1 - \bar{L}(f(\mathbf{x}), y) ]
$$

$M_1$には$\mathbf{x}$が入ってるけど、定数と仮定するので、

$$
= M_1 - \mathbb{E} _{p(\mathbf{x}, y)} [\bar{L}(f(\mathbf{x}), y) ]
$$

そして、ここで、

$$
(K - 1) \mathbb{E} _{\bar{p}(\mathbf{x}, \bar{y})} [ \bar{L}(f(\mathbf{x}), \bar{y}) ] - \mathbb{E} _{\bar{p}(\mathbf{x}, \bar{y})}[L(f(\mathbf{x}), y)]
$$

次に、突然だが、引いてみる。

$$
= M_1 - \mathbb{E} _{p(\mathbf{x}, y)} [ \bar{L}(f(\mathbf{x}), y) + L(f(\mathbf{x}), y)] = M_1 - \mathbb{E} _{p(\mathbf{x}, y)} [ M_2 ] = M_1 - M_2
$$

ここで、$R(f) = \mathbb{E} _{p(\mathbf{x}, y)} [ L(f(\mathbf{x}), y) ]$であることから、

$$
R(f) = \mathbb{E} _{p(\mathbf{x}, y)} [ L(f(\mathbf{x}), y) ] = M_2 - \mathbb{E} _{p(\mathbf{x}, y)} [ \bar{L}(f(\mathbf{x}), y) ] \\\\ 
= M_2 - (M_1 - (K - 1) \mathbb{E} _{\bar{p} (\mathbf{x}, \bar{y})} [\bar{L} (f(\mathbf{x}), \bar{y})])
$$