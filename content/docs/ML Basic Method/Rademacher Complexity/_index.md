---
title: "Rademacher Complexity"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

ラデマッハ複雑度とは、ランダムなノイズに対して学習させるとき、モデルがどれほど追随できるか、という量である。大きいほど表現力が高いモデルである。

ラデマッハ変数という、1/2の確率で+1, 1/2の確率で-1を取る確率変数$\sigma$を考える。厳密には$m$個の訓練データ$\mathbf{x} _i$が存在するときのラデマッハ複雑度は以下のように定義する。$g$は学習器であり、$G$は学習器の集合=モデル。

$$
\mathcal{\hat{R}}(G) = \mathbb{E} _{\boldsymbol{\sigma}} [ \sup _{g \in G} \frac{1}{m} ]\sigma _i g(\mathbf{x} _i)] \\\\ 
= \mathbb{E} _{\boldsymbol{\sigma}} [ \sup _{g \in G} \frac{\boldsymbol{\sigma} ^ T \mathbf{g} _\mathbf{x}}{m} ]
$$

ここで、有限個$m$個の訓練データで得られた経験的なラデマッハ複雑度は、訓練データを増やした時には、理想的な分布から得られたときと同じ値になる。つまり、**不偏推定量**。

$$
\mathbb{R}(G) = \\lim _{m \to \infty} \mathbb{\hat{R}}(G)
$$

次に重要な定理として、ラデマッハ複雑度を用いた二値分類識別器$g \in \mathcal{G}$の期待値を上から抑えられ、不偏推定量からたかだかどれほどずれるのかを評価できる。$g$は$[0, 1]$の値域を取り、0と1がそれぞれ2つのパターンの極端であると示す。以下の式は、訓練データ$\mathbf{x} _i$を形成する確率が$1 - \delta$以上であるという前提での上限である。生成確率自体は累積分布？から得たものの積になるか？

$$
\mathbb{E} [g(\mathbf{x})] = \frac{1}{m} \sum _{i = 1} ^ m g(\mathbf{x} _i) + 2 \mathcal{R} (\mathcal{G}) + \sqrt{\frac{\log \frac{1}{\delta}}{2m}} \\\\ 
\mathrm{or} \\\\ 
\mathbb{E} [g(\mathbf{x})] = \frac{1}{m} \sum _{i = 1} ^ m g(\mathbf{x} _i) + 2 \mathcal{R} (\mathcal{G}) + 3\sqrt{\frac{\log \frac{2}{\delta}}{2m}}
$$

正直使い勝手がいい式ではない(上限の推定するにしても、結構広いのでは？)。

