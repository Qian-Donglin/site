---
title: "2017-NIPS-Learning from Complementary Labels"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
bookCollapseSection: true
# bookComments: false
# bookSearchExclude: false
---

[元論文](https://arxiv.org/abs/1705.07541)

## Introduction

**「○○ではない」という補ラベル**で学習はできないだろうか？

既存では、

- Partial Label　　いくつかのクラスの中の1つなのはわかるが、どれなのかは不明。
- Multi Labels Setup　　各データは1つのクラスのみならず、複数のクラスを持つ。

そしてこの補ラベル学習は、**普通のラベル付けされた学習(1つの正解ラベルを指定するPN Learning)と組み合わせることもできる**と。そうすると性能がさらに上がる。

## 既存のmulti-class分類について

例によって定式化する。

データは$\mathbf{x} \in \mathbb{R}^n$、labelは$y \in 1, 2, \cdots, K$。いつも通り、未知の分布$p(\mathbf{x}, y)$に従う。

最終目的は、$f(\mathbf{x} : \mathbb{R}^d \to 1, 2, \cdots, K)$の識別器$f(\mathbf{x})$を作る事。$f(\mathbf{x})$の決定の中身は以下のように書くとする。

$$
f(\mathbf{x}) = \argmax _{y \in 1, \cdots, K} g_y (\mathbf{x})
$$

$g_y (\mathbf{x})$は、$\mathbf{x}$がクラス$y$であるか否かの二値分類器。(二値分類器なら、いくつかの1があったらそれらがすべてラベルってこと？連続した$[0, 1]$を出すのじゃなかったのか？by me)つまり、**全てのクラスにおいて、もっともらしさを計算して一番もっともらしいものを答え**としている。

例によって、リスク関数$R(f)$を以下のように、損失関数$l(f(\mathbf{x}), y)$で定義

$$
R(f) = \mathbb{E} _{p(\mathbf{x}, y)} [ L(f(\mathbf{x}), y) ]
$$

既存の多クラス分類の手法では、以下の2つの損失関数を持つ。いずれも**普通の損失関数$l(m)$からの線形合成からなる**。

### One Versus All Loss

こちらは、選ばれた1つとそれ以外、それぞれの損失の和。それ以外では、$1 / K - 1$で割ることで平均を得ている。

$$
L _{OVA} (f(\mathbf{x}), y) = l(g_y (\mathbf{x})) + \frac{1}{K - 1} \sum _{y \prime \neq y } l(- g _{y \prime} (\mathbf{x}))
$$

いかに自信満々にクラス$y$であれはあるほど、このロスは小さくなる。逆に、その決定したクラスと他のクラスとの差があまりない場合、2つ目の項でそれなりに損失関数が大きくなってしまう。

### Pairwise Comparison

判定したラベル$y$とそれ以外$y ^ \prime$との識別器出力値の差が大きいとペナルティが出る。$y$は負で、$y ^ \prime$は正という状態で一番ペナルティが発生する。

$$
L _{PC} (f(\mathbf{x}), y) = \sum _{y \prime \neq y} l(g_y (\mathbf{x}) - g _{y \prime} (\mathbf{x}))
$$

上と同じように自信満々にクラス決定できれば強い。このロスの場合、`0.2, 0.1, 0,1, 0,1, 0.1, 0.1, 0.1, 0.1, 0.1`のような分布だと、上の方法より損失関数の値が小さくなる。

## 補ラベルでの学習

まずは普通の分類とは別に、**補ラベル(〇〇ではない、という意味のラベル)**だけで分類器をどう作るかを書く。

$\bar{p}(\mathbf{x}, \bar{y})$が以下の分布に従ってるとする。定義からしたら、それはそうなんだけど。

$$
\bar{p}(\mathbf{x}, \bar{y}) = \frac{1}{K - 1} \sum _{y \neq \bar{y}} p(\mathbf{x}, y)
$$

つまり、**○○以外のすべてのラベルに対しての確率の平均**。

### リスク関数の表示

補ラベルに対して、損失関数$\bar{L}(f(\mathbf{x}), y)$を考える。これが意味のするものは、**予測結果$f(\mathbf{x})$と、補ラベル=$y$じゃない、との相違度**。

こんなに風に表せる。

$$
R(f) = (K - 1) \mathbb{E} _{\bar{p} (\mathbf{x}, \bar{y})} [\bar{L} (f(\mathbf{x}), \bar{y})] - M_1 + M_2
$$

$M _1, M _2$の増減はあれど、$K-1$倍された、補ラベル$\bar{p}(\mathbf{x}, \bar{y})$全体においての、損失関数の期待値。

$$
M_1 = \sum _{\bar{y} = 1}^{K} \bar{L}(f(\mathbf{x}), \bar{y}) 
$$

$M_1$は、$\mathbf{x}$について、**すべてのありえる補ラベルに対して、予測結果との損失関数$\bar{L}$の和**である。

$$
M_2 = \bar{L}(f(\mathbf{x}), y) + L(f(\mathbf{x}), y)
$$

証明は[こちら](./theorem1/_index.md)。

$M_2$は、普通の損失関数と、補ラベルの損失関数の和、みたいな。和が定数という仮定に従うなら、すべての$\mathbf{x}$について、**$\mathbf{x}$がクラス$y$に属してる可能性が高ければ高いほど、$\mathbf{x}$がクラス$y$に属してない可能性が低い**。

実は先ほど提案した$L _{OVA}, L _{PC}$はちゃんと定数になる。


### 実際の使われ方

$$
R(f) = (K - 1) \mathbb{E} _{\bar{p} (\mathbf{x}, \bar{y})} [\bar{L} (f(\mathbf{x}), \bar{y})] - M_1 + M_2
$$

実際にはこの式を経験的な値で代用をするしかない。

$$
\hat{R}(f) = (K - 1)\frac{1}{N} \sum _{i = 1} ^ N \bar{L} (f(\mathbf{x} _i), \bar{y_i}) 
$$

### 損失関数の実際の使われ方

先ほど定義したOVA損失とPC損失の補ラベルにおける場合を書き直してみる。いずれの損失も、定義上$y$と$y$以外のその他$= \bar{y}$なので、補ラベルと相性が良くそのまま書き換えられる。

$$
\bar{L}_{OVA} (f(\mathbf{x}), \bar{y})= \frac{1}{K - 1} (\sum _{y \neq \bar{y}} l(g _{y} (\mathbf{x}_i))) + l( -g _{\bar{y}} (\mathbf{x_i}))
$$

$$
\bar{L}_{PC} (f(\mathbf{x}), y) = \sum _{y \neq \bar{y}} l(g _{y}(\mathbf{x}_i) - g _{\bar{y}}(\mathbf{x}_i))
$$

### 点対称$l(z) + l(-z)$の時ならば

もし、$l(z) + l(-z) = 1$と点対称だとすると、[具体的に式変形](./ova-and-pc-transistion/_index.md)の結果以下のようになる。

OVAの場合、$M _1 = K, M _2 = 2$。PCの場合、$M _1 = K (K - 1) / 2, M _2 = K - 1$。

## リスク関数の評価

例によって、ラデマッハ複雑度を考え、McDiarmidの不等式や一様大数の法則で評価する。

具体的には[こちら](./estimation-error-bound/_index.md)に書いてある。

## 通常のラベル付けと補ラベルの融合

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

