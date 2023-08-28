---
title: "補ラベル学習(2017)"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# Introduction

**「○○ではない」という補ラベル**で学習はできないだろうか？

既存では、

- Partial Label　　いくつかのクラスの中の1つなのはわかるが、どれなのかは不明。
- Multi Labels Setup　　各データは1つのクラスのみならず、複数のクラスを持つ。

そしてこの補ラベル学習は、**普通のラベル付けされた学習と組み合わせることもできる**と。

# 既存のmulti-class分類についてのおさらい

例によって定式化する。

データは$\mathbf{x} \in \mathbb{R}^n$、labelは$y \in 1, 2, \cdots, K$。いつも通り、未知の分布$p(\mathbf{x}, y)$に従う。

最終目的は、$f(\mathbf{x} : \mathbb{R}^d \to 1, 2, \cdots, K)$の識別器$f(\mathbf{x})$を作る事。$f(\mathbf{x})$の決定の中身は以下のように書くとする。

$$
f(\mathbf{x}) = \argmax _{y \in 1, \cdots, K} g_y (\mathbf{x})
$$

$g_y (\mathbf{x})$は、$\mathbf{x}$がクラス$y$であるか否かの二値分類器。(二値分類器なら、いくつかの1があったらそれらがすべてラベルってこと？連続した$[0, 1]$を出すのじゃなかったのか？by me)つまり、**全てのクラスにおいて、もっともらしさを計算して一番もっともらしいものを答え**としている。

例によって、リスク関数$R(f)$を以下のように、損失関数$l(f(\mathbf{x}), y)$で定義(二値分類ではないので、1つの引数にまとめるのはできない)。この$l(m)$は、$m$が小さいほど大きな値を取る。

$$
R(f) = \mathbb{E} _{p(\mathbf{x}, y)} [ L(f(\mathbf{x}), y) ]
$$

## 損失関数

以下の2つの損失関数について着目する。いずれも**普通の損失関数$l(m)$からの線形合成からなる**。

### One Versus All Loss

$$
L _{OVA} (f(\mathbf{x}), y) = l(g_y (\mathbf{x})) + \frac{1}{K - 1} \sum _{y \prime \neq y } l(- g _{y \prime} (\mathbf{x}))
$$

いかに自信満々にクラス$y$であれはあるほど、このロスは小さくなる。逆に、その決定したクラスと他のクラスとの差があまりない場合、2つ目の項でそれなりに損失関数が大きくなってしまう。

### Pairwise Comparison

$$
L _{PC} (f(\mathbf{x}), y) = \sum _{y \prime \neq y} l(g_y (\mathbf{x}) - g _{y \prime} (\mathbf{x}))
$$

上と同じように自信満々にクラス決定できれば強い。このロスの場合、`0.2, 0.1, 0,1, 0,1, 0.1, 0.1, 0.1, 0.1, 0.1`のような分布だと、上の方法より良く出る。

# どうやって補ラベルからやるのか？

まずは普通の分類とは別に、**補ラベル(〇〇ではない！)**だけで分類器をどう作るかを書く。

$\bar{p}(\mathbf{x}, \bar{y})$が以下の分布に従ってるとする。定義からしたら、それはそうなんだけど。

$$
\bar{p}(\mathbf{x}, \bar{y}) = \frac{1}{K - 1} \sum _{y \neq \bar{y}} p(\mathbf{x}, y)
$$

つまり、**○○以外のすべてのラベルに対しての確率の平均**。

## リスク関数の表示

補ラベルに対して、損失関数$\bar{L}(f(\mathbf{x}), y)$を考える。これが意味のするものは、**予測結果$f(\mathbf{x})$と、補ラベル=$y$じゃない、との相違度**。

こんなに風に表せる。

$$
R(f) = (K - 1) \mathbb{E} _{\bar{p} (\mathbf{x}, \bar{y})} [\bar{L} (f(\mathbf{x}), \bar{y})] - M_1 + M_2
$$

$M1, M2$の増減はあれど、$K-1$倍された、補ラベル$\bar{p}(\mathbf{x}, \bar{y})$全体においての、損失関数の期待値。

$$
M_1 = \sum _{\bar{y} = 1}^{K} \bar{L}(f(\mathbf{x}), \bar{y}) 
$$

$M_1$は、**すべてのありえる補ラベルに対して、予測結果との損失関数$\bar{L}$の和**である。

$$
M_2 = \bar{L}(f(\mathbf{x}), y) + L(f(\mathbf{x}), y)
$$

$M_2$は、普通の損失関数と、補ラベルの損失関数の和、みたいな。和が定数という仮定に従うなら、**$\mathbf{x}$がクラス$y$に属してる可能性が高ければ高いほど、$\mathbf{x}$がクラス$y$に属してない可能性が低い**。

**ここで、$M_1, M_2$は定数であると仮定する。実際はそうじゃない場合も多々あるが、ここで考えて評価するには仮定する**！

### 変形

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
R(f) = \mathbb{E} _{p(\mathbf{x}, y)} [ L(f(\mathbf{x}), y) ] = M_2 - \mathbb{E} _{p(\mathbf{x}, y)} [ \bar{L}(f(\mathbf{x}), y) ]
$$

$$
= M_2 - (M_1 - (K - 1) \mathbb{E} _{\bar{p} (\mathbf{x}, \bar{y})} [\bar{L} (f(\mathbf{x}), \bar{y})])
$$

このように、式変形できるのだ。

### 実際の使われ方

$$
R(f) = (K - 1) \mathbb{E} _{\bar{p} (\mathbf{x}, \bar{y})} [\bar{L} (f(\mathbf{x}), \bar{y})] - M_1 + M_2
$$

実際にはこの式を経験的な値で代用をするしかない。

$$
\hat{R}(f) = (K - 1)\frac{1}{N} \bar{L} \sum _{i = 1} ^ N (f(\mathbf{x} _i), \bar{y_i}) 
$$

### 損失関数の実際の使われ方

先ほど定義したOVA損失とPC損失を、経験損失で書き直してみる。**補ラベルなので、$l(m)$の符号は逆である**。

$$
\bar{L}_{OVA} (f(\mathbf{x}), \bar{y})= \frac{1}{N - 1} \sum _{y \neq \bar{y}} l(g _{y} (\mathbf{x}_i)) + l( -g _{\bar{y}} (\mathbf{x_i}))
$$

$$
\bar{L}_{PC} (f(\mathbf{x}), y) = \sum _{y \neq \bar{y}} l(g _{y}(\mathbf{x}_i) - g _{\bar{y}}(\mathbf{x}_i))
$$

あれ、PCのlの中身の符号はお互い逆では？？？ようわからんのだ。

## 点対称$l(z) + l(-z)$

もし、$l(z) + l(-z) = 1$と点対称ならば、

$$
\bar{L} _{OVA} (f(\mathbf{x}), \bar{y})= \frac{1}{N - 1} \sum _{y \neq \bar{y}} l(g _{y} (\mathbf{x}_i)) + l( -g _{\bar{y}} (\mathbf{x_i})) = 1
$$

この時定数の$M_1 = N, M_2 = 2$

$$
\bar{L}_{PC} (f(\mathbf{x}), y) = \sum _{y \neq \bar{y}} l(g _{y}(\mathbf{x}_i) - g _{\bar{y}}(\mathbf{x}_i))
$$

この場合は求まらないが、$M_1 = \frac{K (K - 1)}{2}$であり、$M_2 = K - 1$となる。

$l(z) + l(-z) = 1$となる関数はシグモイド関数、ランプ損失、01損失などがある。シグモイドとランプは識別器の訓練に、0-1はハイパーパラメタの訓練に使われる。

# リスク関数の抑え方

ここはスキップで...

# 通常のラベル付けと補ラベルの融合

通常のラベル付けのリスク関数は

$$
\mathbb{E} _{p(\mathbf{x}, y)} [ L(f(\mathbf{x}), y) ]
$$

なので、組み合わせるだけならば、以下の関数の最適化を考えればよい。以下の関数は融合できてると言える。$\alpha$は0から1の間の定数。

$$
\hat{R} (f) = \alpha \frac{1}{N_1} \sum _{i = 1} ^ {N_1} L(f(\mathbf{x}_i), y_i) + (1 - \alpha) (K - 1)\frac{1}{N_2} \bar{L} \sum _{i = 1} ^ {N_2} (f(\mathbf{x} _i), \bar{y_i}) 
$$

これの最小化という最適化問題に落ち着く。

# 実験結果

