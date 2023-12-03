---
title: "Estimation Error Bounds"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookComments: false
# bookSearchExclude: false
---

[元記事](../_index.md)

$p(\mathbf{x})$から$n$個のデータを取り出して、$X$とする。補ラベルの分布からも同様に、$\bar{p}(\mathbf{x})$から$\bar{X}$とする。
これらのラデマッハ複雑度は以下のように定められる。

$$
R _n(\mathcal{G}) = \mathbb{E} _{X} \mathbb{E} _{\sigma _1, \cdots, \sigma _n} [ \sup _{g \in \mathcal{G}} \frac{1}{n} \sum _{\mathbf{x} _i \in X} \sigma _i g(\mathbf{x} _i)] \\\\ 
\bar{R _n}(\mathcal{G}) = \mathbb{E} _{\bar{X}} \mathbb{E} _{\sigma _1, \cdots, \sigma _n} [ \sup _{g \in \mathcal{G}} \frac{1}{n} \sum _{\mathbf{x} _i \in \bar{X}} \sigma _i g(\mathbf{x} _i)]
$$

この2つは等しい。なぜならば、$p(\mathbf{x}) = \bar{p}(\mathbf{x})$である。$\bar{p}$は、周辺確率を考えるとnot犬not猫...=サル　ということなので、これも成り立つ。

次に[Talagrandの補題](../../../../ML%20Basic%20Method/Rademacher%20Complexity/_index.md#Taragrandの補題)を適用できるように、損失関数をシフトして0の時には0を返すようにする。$\tilde{k}(z) = l(z) - l(0)$

## 一様大数の法則

そして、$K$個の分類がある以上、

これを踏まえたうえで、一様大数の法則を使うことで、以下の事実がわかる。

$\forall \delta > 0$で、$1 - \delta$以上の確率で以下の事実が成り立つ。

OVA損失の場合、

$$
\sup _{g _1, \cdots, g _K \in \mathbf{G}} |\hat{R}(f) - R(f)| \leq 2 K (K - 1) L _l \mathcal{R} _n (\mathcal{G}) + (K - 1) \sqrt{\frac{2 \log (2 / \delta)}{n}}
$$

PC損失の場合、

$$
\sup _{g _1, \cdots, g _K \in \mathbf{G}} |\hat{R}(f) - R(f)| \leq 4 K (K - 1) ^ 2 L _l \mathcal{R} _n (\mathcal{G}) + (K - 1) ^ 2 \sqrt{\frac{2 \log (2 / \delta)}{n}}
$$

OVA損失の方から証明する。

McDiarmidの不等式で、下式が成り立つ。

$$
Pr(\sup _{g _1, \cdots, g _K \in \mathcal{G}} (\hat{R}(f) - R(f)) - \mathbb{E}[ \sup _{g _1, \cdots, g _K \in \mathcal{G}} (\hat{R}(f) - R(f)) ] \geq \epsilon) \leq \exp(- \frac{2 \epsilon ^ 2}{n (2(K - 1) / n) ^ 2})
$$

ここで、$\delta / 2 = \exp(- \frac{2 \epsilon ^ 2}{n (2(K - 1) / n) ^ 2})$とすると、

$$
\epsilon = (K - 1) \sqrt{\frac{2 \log (2 / \delta)}{n}}
$$

これが成り立つので、$1 - \frac{\delta}{2}$以上の確率で、以下が成り立つ。

$$
\sup _{g _1, \cdots, g _K \in \mathcal{G}} (\hat{R}(f) - R(f)) \leq \mathbb{E}[ \sup _{g _1, \cdots, g _K \in \mathcal{G}} (\hat{R}(f) - R(f)) ] + \epsilon + (K - 1) \sqrt{\frac{2 \log (2 / \delta)}{n}}
$$

次に、$\mathbb{E}[ \sup _{g _1, \cdots, g _K \in \mathcal{G}} (\hat{R}(f) - R(f)) ]$を評価する。一様大数の法則と同じ流れで証明していくことができる。これは$n$個のデータから計算されているので、これを展開していくと、