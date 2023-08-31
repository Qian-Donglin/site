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

p. 45まで