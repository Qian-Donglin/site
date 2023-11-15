---
title: "Theorem2 Proof"
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
\mathbb{E} _{p(\mathbf{x}, y)} [ l _{01} (y f(\mathbf{x})) ] - \frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} _{\eta} (y _i ^ \prime f(\mathbf{x} _i ^ \prime)) \leq \frac{\pi ^ *}{n} \sum _{i = 1} ^ n \tilde{l} _{\eta} (f(\mathbf{x})) + (\frac{\pi ^ *}{\sqrt{n}} + \frac{2}{\sqrt{n ^ \prime}}) \frac{C _{\alpha} C _{k}}{\eta} + (\frac{\pi ^ *}{2 \sqrt{n}}) \sqrt{\frac{\log(2 / \delta)}{2}}
$$

## 補題4

$\forall \eta > 0$とする。PositiveとUnlabeledデータをサンプリングする。$\forall \delta \in (0, 1)$として、$1 - \delta$以上の確率で、識別器$\forall f \in \mathcal{F}$に対して、以下が成り立つ。

$$
\mathbb{E} _{p(\mathbf{x}, y)} [ \tilde{l} (y f(\mathbf{x})) ] - \frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} _{\eta} (y _j ^ \prime f(\mathbf{x} _i ^ \prime)) \leq \frac{2}{\eta} \mathcal{R} _{n ^ \prime} (\mathcal{F}) + \sqrt{\frac{\log (1 / \delta)}{2n ^ \prime}}
$$

これを示す。

まず、$y$を限定しないとき、$\tilde{l}$は$[0, 1]$となるので、一様大数の法則を用いると以下の式が得られる。

$$
\mathbb{E} _{p(\mathbf{x}, y)} [ \tilde{l} (y f(\mathbf{x})) ] - \frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} _{\eta} (y _j ^ \prime f(\mathbf{x} _i ^ \prime)) \leq 2 \mathcal{R} _{n ^ \prime} (\mathcal{\tilde{l} _{\eta} (F)}) + \sqrt{\frac{\log(1 / \delta)}{2n ^ \prime}}
$$

[Talagrandの補題](../../../../ML%20Basic%20Method/Rademacher%20Complexity/_index.md##Taragrandの補題)により、以下の事実が成り立つ。$\tilde{l} _{\eta}$はリプシッツ連続であり、リプシッツ定数は$1 / \eta$である。よって、以下のことが成り立つ。

$$
\mathcal{R} _{n ^ \prime} (\tilde{l} _{\eta} (\mathcal{F})) \leq \frac{1}{\eta} \mathcal{R} _{n ^ \prime} (\mathcal{F})
$$

よって、これを代入すると、以下の補題が成り立つ。

$$
\mathbb{E} _{p(\mathbf{x}, y)} [ \tilde{l} (y f(\mathbf{x})) ] - \frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} _{\eta} (y _j ^ \prime f(\mathbf{x} _i ^ \prime)) \leq \frac{2}{\eta} \mathcal{R} _{n ^ \prime} (\mathcal{F}) + \sqrt{\frac{\log (1 / \delta)}{2n}}
$$

## 補題5

$\forall \eta > 0$として、PositiveとUnlabeledデータをサンプリングする。$\forall \delta \in (0, 1)$として、$1 - \delta$以上の確率で以下の式が成り立つ。

$$
\mathbb{E} _{p(\mathbf{x} | y = +1)} [ \tilde{l}(f(\mathbf{x})) ] - \frac{1}{n} \sum _{i = 1} ^ n \tilde{l} _{\eta} (f(\mathbf{x} _i)) \leq \frac{1}{\eta} \mathcal{R} _{n} (\mathcal{F}) + \sqrt{\frac{\log(1 / \delta)}{8n}}
$$

これを証明する。

補題4と同様に考えると、$y = +1$に限定したら$\tilde{l}$は$[0, 1/2]$を取るので、McDiarmidの不等式では1ステップ当たりの上界は$\frac{1}{2n}$となるので、$\sqrt{\frac{\log(1 / \delta)}{8n}}$となる。

また、同様にリプシッツ連続であると考えると、$y = +1$の時の$\tilde{l}$のリプシッツ定数は$1 / \eta$ではなく全体が$1 / 2$されたので、$1 / 2\eta$となる。

これを踏まえると補題5は成立する。

## ラデマッハ複雑度の上界

本筋は[こちらを見る](../../../../read_book/Machine%20Learning/ML%20from%20Weak%20Supervision/section%203/_index.md#ラデマッハ複雑度での上界)とわかりやすい。形としては$\cdot / \sqrt{n}$に、線形分類器やNNは皆なる。

今回の場合、$k$の上界$C _k$やパラメタの上界$C _{\alpha}$を論文の中で定義している。これは有界なので(書くのがめんどくなった...)、ラデマッハ複雑度はそれぞれ以下のようになる。

$$
\mathcal{R} _{n} (\mathcal{F}) = \frac{C _\alpha C _x}{\sqrt{n}} \\\\ 
\mathcal{R} _{n} (\mathcal{F}) = \frac{C _\alpha C _x}{\sqrt{n ^ \prime}} 
$$

これを代入すると、

$$
\mathbb{E} _{p(\mathbf{x}, y)} [ \tilde{l} (y f(\mathbf{x})) ] - \frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} _{\eta} (y _j ^ \prime f(\mathbf{x} _i ^ \prime)) \leq \frac{2}{\eta} \frac{C _\alpha C _x}{\sqrt{n ^ \prime}} + \sqrt{\frac{\log (1 / \delta)}{2n ^ \prime}} \\\\ 
\mathbb{E} _{p(\mathbf{x} | y = +1)} [ \tilde{l}(f(\mathbf{x})) ] - \frac{1}{n} \sum _{i = 1} ^ n \tilde{l} _{\eta} (f(\mathbf{x} _i)) \leq \frac{1}{\eta} \frac{C _\alpha C _x}{\sqrt{n}} + \sqrt{\frac{\log(1 / \delta)}{8n ^ \prime}}
$$

## 最後に

[定理1](../theorem1_proof/_index.md)のように組み合わせることを考える。$\pi ^ * = p(y = +1)$

すると、以下の式が成り立つ。

$$
\mathbb{E} _{p(\mathbf{x}, y)} [l _{01} (y f(\mathbf{x}))]
= p(y = +1) \mathbb{E} _{p(\mathbf{x} | y = +1)} [ \tilde{l} (f(\mathbf{x})) ] + \mathbb{E} _{p(\mathbf{x})} [ \tilde{l} (f(\mathbf{x})) ] \\\\ 
= \pi ^ * (\frac{1}{n} \sum _{i = 1} ^ n \tilde{l} _{\eta} (f(\mathbf{x} _i)) + \frac{1}{\eta} \frac{C _\alpha C _x}{\sqrt{n}} + \sqrt{\frac{\log(1 / \delta)}{8n}}) \\\\ 
+\frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} _{\eta} (y _j ^ \prime f(\mathbf{x} _i ^ \prime)) + \frac{2}{\eta} \frac{C _\alpha C _x}{\sqrt{n ^ \prime}} + \sqrt{\frac{\log (1 / \delta)}{2n}} \\\\ 
= (\frac{\pi ^ *}{2 \sqrt{n}} + \frac{1}{\sqrt{n}})\sqrt{\frac{\log(1 / \delta)}{2}} + \frac{C _\alpha C _X}{\eta} (\frac{\pi ^ *}{\sqrt{n}} + \frac{2}{n ^ \prime}) \\\\ 
+\pi ^ * \frac{1}{n} \sum _{i = 1} ^ n \tilde{l} _{\eta} (f(\mathbf{x} _i)) + \sum _{i = 1} ^ {n ^ \prime} \tilde{l} _{\eta} (y _j ^ \prime f(\mathbf{x} _i ^ \prime))
$$

最後に移行すると、定理2が得られる。

$$
\mathbb{E} _{p(\mathbf{x}, y)} [ l _{01} (y f(\mathbf{x})) ] - \frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} _{\eta} (y _i ^ \prime f(\mathbf{x} _i ^ \prime)) \leq \frac{\pi ^ *}{n} \sum _{i = 1} ^ n \tilde{l} _{\eta} (f(\mathbf{x})) + (\frac{\pi ^ *}{\sqrt{n}} + \frac{2}{\sqrt{n ^ \prime}}) \frac{C _{\alpha} C _{k}}{\eta} + (\frac{\pi ^ *}{2 \sqrt{n}}) \sqrt{\frac{\log(2 / \delta)}{2}}
$$