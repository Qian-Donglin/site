---
title: "Weakly Supervised Learning"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
bookCollapseSection: true
# bookComments: false
# bookSearchExclude: false
---

# PUの損失関数の定式化

PUについての定式化の1種類 by [du Plessisの論文](https://proceedings.neurips.cc/paper_files/paper/2014/file/35051070e572e47d2c26c241ab88307f-Paper.pdf)

PNをまず考える。$R _{fp}$偽陽性=negativeなのにpositiveにされる**率**。$R _{fn}$偽陰性=positiveなのにnegativeにされる**率**。この2つが誤分類のエラーの起源であり、事前確率として$\pi = p(y = +1)$を知っている時に以下のように定式化できる。

$$
R _{fp}(g) = \mathbb{E} _{y = -1} [l(g(\mathbf{x}), -1)] \\\\ 
R _{fn}(g) = \mathbb{E} _{y = +1} [l(g(\mathbf{x}), +1)] \\\\ 
R(g) = \pi R _{fp} (g) + (1 - \pi) R _{fn} (g)
$$

しかし、**PUではNegative例は存在しないので、$R _{fp}(g)$をなくさないといけない**から、Unlabeledでうまく代替しなければならない。**Unlabeledは$\pi$の割合でpositive、$1 - \pi$の割合でnegativeであると考えられる**。

ここで、Unlabeledに対して、**全部positive**であるとしたときの期待値を考える。以下のように分割したものの和とみなせる。

- NegativeがPositiveに判定される率。これは$R _{fp}(g)$である。これの割合は$1 - \pi$
- PositiveがPositiveに判定される率。これは、偽陰性=pがnになるの余事象なので、$1 - R _{fn}(g)$であり、これの割合は$\pi$

$$
R _{X}(g) = \pi (1 - R _{fn}(g)) + (1 - \pi) R _{fp}(g) \\\\ 
$$

$R(g) = \pi R _{fp} (g) + (1 - \pi) R _{fn} (g)$に$(1 - \pi) R _{fp}(g)$を代入して式変形すると最終的には、以下の式となる。

$$
R(g) = 2\pi R _{fp}(g) + R _{X}(g) - \pi
$$

ここで、$\eta$を全体の中のラベル付きの割合とする(これは経験的に推定できる)。すると、$R(g)$は以下のように書き直せる。

$$
c _1 = \frac{2 \pi}{\eta}, c _X = \frac{1}{1 - \eta} \\\\ 
R(g) = 2\pi R _{fp}(g) + R _{X}(g) - \pi = \frac{2 \pi}{\eta} \eta R _1(g) + \frac{1}{1 - \eta} (1 - \eta) R _X(g) - \pi \\\\ 
= c _1 \eta R _1(g) + c _C (1 - \eta) R _X(g) - \pi
$$

Cost-Sensitive Classificationは確率$\pi$でクラスA、$1 - \pi$でクラスBが得られるとするとき、損失関数が

$$
R(g) = \pi R _A(g) + (1 - \pi) R _B(g)
$$

となるのみならず、お互いにコスト$c _A, c _B$まで定義されているというもの。

$$
R(g) = c _A \pi R _A(g) + c _B (1 - \pi) R _B(g)
$$

