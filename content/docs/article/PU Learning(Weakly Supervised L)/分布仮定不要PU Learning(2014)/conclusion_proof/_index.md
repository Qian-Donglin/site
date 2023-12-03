---
title: "Conclusion Proof"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[元記事](../_index.md)

$$
\forall a, \forall b \in \mathbb{N} \\\\ 
\frac{1}{\sqrt{a}} + \frac{1}{\sqrt{b}} \geq k \frac{1}{\sqrt{a + b}}
$$

このような最小の$k$を求める。

まずは以降して$k$だけくくりだす。

$$
\sqrt{\frac{a + b}{a}} + \sqrt{\frac{a + b}{b}} \geq k
$$

両辺が共に正なので、2乗すると以下の式が得られる。

$$
1 + \sqrt{b}{a} + 2 \frac{a + b}{\sqrt{ab}} + 1 + \frac{a}{b} = 2 + \frac{a}{b} + \sqrt{b}{a} + 2 \frac{a + b}{\sqrt{ab}} \geq k ^ 2
$$

相加相乗平均の不等式$a + b \geq 2 \sqrt{ab}$を用いると、以下の事実が成り立つ。

$$
\frac{a}{b} + \frac{b}{a} \geq 2 \times \sqrt{\frac{a}{b} \cdot \frac{b}{a}} = 2 \\\\ 
a + b \geq 2 \sqrt{ab} \Rightarrow \frac{a + b}{\sqrt{ab}} \geq 2
$$

これを元の式に代入すると、

$$
2 + \frac{a}{b} + \sqrt{b}{a} + 2 \frac{a + b}{\sqrt{ab}} = 2 + 2 + 2 \cdot 2 = 8 \geq k ^ 2 \\\\ 
k \leq 2 \sqrt{2}
$$

よって、等号成立条件は$k = 2\sqrt{2}$であり、以下が成り立つ。

$$
\forall a, \forall b \in \mathbb{N} \\\\ 
\frac{1}{\sqrt{a}} + \frac{1}{\sqrt{b}} \geq 2 \sqrt{2} \times  \frac{1}{\sqrt{a + b}}
$$