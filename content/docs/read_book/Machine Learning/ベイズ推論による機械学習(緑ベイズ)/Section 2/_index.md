---
title: "第二章"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# 期待値

$$
\mathbb{E} [f(\mathbf{x})] = \in f(\mathbf{x}) p(\mathbf{x}) d\mathbf{x}
$$

と定義される。純粋に$p(\mathbf{x})$の取る平均は$\mathbb{E} [\mathbf{x}] = \int \mathbf{x} p(\mathbf{x}) d\mathbf{x}$

期待値は独立かそうじゃないか関わらず、線形性を持つ。

分散は、
$$
\mathbb{E} [(\mathbf{x} - \hat{\mathbf{x}}) ^ T (\mathbf{x} - \hat{\mathbf{x}})]
= \mathbb{E} [\mathbf{x} ^ T \mathbf{x}] - \mathbb{E} [\mathbf{x} ^ T] \mathbb{E} [\mathbb{x}]
$$

## 2変数以上の期待値

$$
\mathbb{E} _{p(\mathbf{x}, \mathbf{y})} [\mathbf{x} ^ T \mathbf{y}]
$$

を計算する。もし$\mathbf{x}, \mathbf{y}$が**お互いに独立**であるなら、$p(\mathbf{x}, \mathbf{y}) = p(\mathbf{x}) p(\mathbf{y})$であり、

$$
\int \int \mathbf{x} ^ T \mathbf{y} p(\mathbf{x, y})d\mathbf{x} d\mathbf{y} 
= \int \mathbf{x} ^ T p(\mathbf{x}) d\mathbf{x} + \int \mathbf{y} p(\mathbf{y}) d \mathbf{y}
= \mathbb{E} [\mathbf{x} ^ T] \mathbb{E} [\mathbf{y}]
$$

が成り立つ。
しかし、独立ではない場合は、上の積分を丁寧に行うしかない。これは条件付期待値。

$$
\int \int \mathbf{x} ^ T \mathbf{y} p(\mathbf{x, y})d\mathbf{x} \mathbf{y} 
= \mathbb{E}_{\mathbf{y}} [ \mathbb{E} _{\mathbf{x | y}} [\mathbf{x} ^ T \mathbf{y}] ]
$$

# エントロピー

エントロピー$H(\mathbf{x})$を定める。大きいほど乱雑で、0が一番整っている。ここで$\log 0 = 0$と定義。

$$
H(\mathbf{x}) = -\mathbb{E} [\log p(\mathbf{x})] = -\int p(\mathbf{x}) \log p(\mathbf{x}) d \mathbf{x}
$$

# KLダイバージェンス

2つの分布$p(\mathbf{x}), q(\mathbf{x})$について、**疑似的に分布の距離っぽいなにか(距離の公理は満たさない！！厳密にやるなら最適輸送)**。これをKLダイバージェンス。

$$
\mathrm{KL} [p(\mathbf{x}) || q(\mathbf{x})] = \mathbb{E}_{p(\mathbb{x})} [\log p(\mathbf{x})] - \mathbb{E} _{p(\mathbb{x})} [\log q(\mathbf{x})]
= -\int q(\mathbf{x}) \log \frac{q(\mathbf{x})}{p(\mathbf{x})} d\mathbf{x}
$$

# サンプリングによる期待値の推定

分散はそうでもないけど期待値はサンプリングの平均は不偏推定量なんで、$\frac{1}{n} \sum _{i = 1} ^ {n} f(\mathbf{x}_i)$

# 離散確率分布

|            | 事象2つ    | 事象3つ以上 |
| ---------- | ---------- | ----------- |
| 1回試行    | ベルヌーイ | カテゴリ    |
| 複数回試行 | 二項       | 多項        |

## ベルヌーイ分布

1回のコイン投げで、表の確率が$\mu$、裏が$1 - \mu$の時、$x$を取る確率は以下のようにまとめられる。
$x = 0$なら$1 - \mu$, $x = 1$なら$\mu$になるのをまとめている。

$$
\mathrm{Bern}(x | \mu) = \mu ^ x (1 - \mu) ^ {1 - x}, x \in \{0, 1\}
$$

なお、エントロピーは$- \mathbb{E} [\log p(\mathbf{x})] = -(1 - \mu)\log (1 - \mu) - \mu \log \mu$となり、この形は明らかに$\mu = 0.5$で最大値を取る。

## 二項分布

複数回のベルヌーイ分布をやる。$M$回の試行を繰り返し、表が出るのは$\mu$の確率である。
この時、**$x$回表が出る確率**は以下のようになる。

$$
\mathrm{Bin | M, \mu} = _M C _x \mu ^ x (1 - \mu) ^ {M - x} 
$$

明らかに上式は$M = 1$において、ベルヌーイ分布と同じようになる。$x = 0$ならば表が出ない、$x = 1$ならば表が出る。$M > 1$だと$x$は回数という意味だが、$x = 1$である限りベルヌーイ分布と同じ。

主な期待値は以下のようになる。

$$
\mathbb{E} [x] = M \mu \\\\ 
\mathbb{E} [x ^ 2] = M \mu ((M - 1) \mu + 1)
$$

## カテゴリ分布

ベルヌーイ分布は表と裏の2通りしかないが、これを3通り以上の状態に拡張する。
$\mathbf{s}$はカテゴリを示すone-hotベクトル。$\mathbf{\pi}$は各カテゴリにおける出現確率。
以下の式では総乗であるが、$s_i = 1$は一つだけなので、そこの$\pi_i$だけ出てそれ以外は$1$となる。掛け合わせると$\pi_i$が出てくる。

$$
\mathrm{Cat} (\mathbf{s}, \mathbf{\pi}) = \prod_{i = 1} ^ {N} \pi_i ^ {s_i}
$$

同様にエントロピーを計算すると、

$$
-\mathbb{E} [\log \prod_{i = 1} ^ {N} \pi _i ^ {s_i}] = - \sum _{i = 1} ^ {N} \pi _i \log \pi _i
$$

## 多項分布

二項分布においてカテゴリを3つ以上にも拡張したもの。3つ以上の種類の$N$個の球を1列に並べる並び方が$\frac{N!}{a!b!c!\cdots}, a + b + c + \cdots = N$であるので、同様に分布の式も以下のようになる。$\mathbf{x}$は各カテゴリにおける出現数。

$$
\mathrm{Mult}(\mathbf{x} | \mathbf{\pi}, M) = M! \prod _{i = 1} ^ {N} \frac{\pi_i ^ {x_i}}{x_i !}
$$

これの期待値は、

$$
\mathbb{E} [ x_i ] = M \pi_i \\\\ 
\mathbb{E} [ x_i x_j ] = M \pi_i ((M - 1)\pi_i + 1) \\:\\: \mathrm{if} j = k \\\\ 
= M(M - 1) \pi_i \pi_j \\: \\: \mathrm{otherwise}
$$

## ポアソン分布

