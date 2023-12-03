---
title: "Theorem1 Proof"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[元記事](../_index.md)

識別器$\forall f \in \mathcal{F}$がある。PositiveとUnlabeledデータをサンプリングする。
そこで、$\delta \in (0, 1)$として、少なくとも$1 - \delta$以上の確率で下の式が成り立つ。($\pi ^ * = p(y = +1)$)

$$
\mathbb{E} _{p(\mathbf{x},y)}\left[ l _{01}(yf(\mathbf{x})) \right] - \frac{1}{n ^ \prime} \sum _{j=1}^{n\prime} \tilde{l} (y ^ \prime _j f(\mathbf{x} ^ \prime _j)) \leq \frac{\pi^*}{n} \sum _{i=1}^{n} l(f(\mathbf{x} _i)) + ( \frac{\pi ^ *}{2\sqrt{n}} + \frac{1}{\sqrt{n ^ \prime}}) \sqrt{\frac{\ln(2/\delta)}{2}}.
$$

以上のことを証明する。

## McDiarmidの不等式

[Wikipediaはこちら](https://en.wikipedia.org/wiki/McDiarmid%27s_inequality)。

ある$n$変数関数$f$を考える。確率変数$\forall i, X _i$についてどのような値を取っても$f$の取りうる値の変動が$c _i$で上から抑えられるとする。この時、以下の式が成り立つ。

$$
\forall \epsilon > 0 \\\\ 
p ( f(X _1, \ldots, X _n) - \mathbb{E}[f(X _1, \ldots, X _n)] \geq \epsilon ) \leq \exp( -\frac{2\epsilon ^ 2}{\sum _{i = 1} ^ {n} c _i^2} )
$$

**期待値と実際の経験的試行のズレの大きさを抑えた不等式**。

## $p(\mathbf{x} | y = +1)$の期待値についての不等式

$$
\tilde{l}(y f(\mathbf{x})) = \frac{2}{y + 3} l _{01} (y f(\mathbf{x})) \\\\ 
\tilde{l} _{\eta} (y f(\mathbf{x})) = \frac{2}{y + 3} l _{\eta} (y f(\mathbf{x}))
$$

このような形で$\tilde{l}$は定義されていた。ここで、$y = +1$の場合取りうる値が$[0, 1/2]$に限定されることに注目する。McDiarmidの不等式において$\forall i, c _i = 1 / 2$と言えるので、$n$で割られていることを考えると、以下のように記述できる。

$$
p( \mathbb{E} _{p(\mathbf{x} | y = +1)} [ \tilde{l} (f(\mathbf{x})) ] - \frac{1}{n} \sum _{i = 1} ^ n \tilde{l} (f(\mathbf{x} _i)) \geq \epsilon) \leq \exp(- \frac{ 2 \epsilon ^ 2}{n (1 / n) ^ 2})
$$

ここで、右辺を$\delta / 2$と記述する。(確率の中心から外れた両サイドが存在するので、半分とする。)式変形してみる。

$$
\exp(- \frac{ 2 \epsilon ^ 2}{n (1 / n) ^ 2}) = \frac{\delta}{2} \\\\ 
-\log \frac{\delta}{2} = \frac{ 2 \epsilon ^ 2}{n (1 / 2n) ^ 2} \\\\ 
\log \frac{2}{\delta} = \frac{ 2 \epsilon ^ 2}{\frac{1}{4n}} \\\\ 
\frac{\log \frac{2}{\delta}}{8n} = \epsilon ^ 2 \\\\ 
\epsilon = \sqrt{\frac{\log (2 / \delta)}{8n}}
$$

これを基にMcDiarmidの不等式の記述を言い換えると、**$\delta / 2$以下の確率で、以下が成り立つ**。

$$
\mathbb{E} _{p(\mathbf{x} | y = +1)} [ \tilde{l} (f(\mathbf{x})) ] - \frac{1}{n} \sum _{i = 1} ^ n \tilde{l} (f(\mathbf{x} _i)) \geq \epsilon = \sqrt{\frac{\log (2 / \delta)}{8n}}
$$

これの**余事象**をとると、**$1 - \delta / 2$以上の確率で、以下が成り立つ**。

$$
\mathbb{E} _{p(\mathbf{x} | y = +1)} [ \tilde{l} (f(\mathbf{x})) ] - \frac{1}{n} \sum _{i = 1} ^ n \tilde{l} (f(\mathbf{x} _i)) \leq \epsilon = \sqrt{\frac{\log (2 / \delta)}{8n}}
$$

## $p(\mathbf{x})$の期待値についての不等式

$y = +1$と限られない時には$\tilde{l}$は$[0, 1]$を取る。よって、毎回は1だけずれる。$n$で割ることを考えると、McDiarmidの不等式で評価すると、以下のようになる。

$$
p(\mathbb{E} _{p(\mathbf{x}, y)} [ \tilde{l} (y f(\mathbf{x})) - \frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} (y ^ \prime _i f(\mathbf{x ^ \prime _i}))] \geq \epsilon) \leq \exp(- 2 n ^ \prime \epsilon ^ 2)
$$

同様に右辺を$\delta / 2$と定めれば、以下のようになる。

$$
\epsilon = \sqrt{\frac{\log (2 / \delta)}{2 n ^ \prime}}
$$

同様に言い換えることで、**$1 - \delta / 2$以上の確率で、以下が成り立つ**。

$$
\mathbb{E} _{p(\mathbf{x}, y)} [ \tilde{l} (y f(\mathbf{x})) - \frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} (y ^ \prime _i f(\mathbf{x ^ \prime _i}))] \leq \epsilon = \sqrt{\frac{\log (2 / \delta)}{2 n ^ \prime}}
$$

## 仕上げ

ここで、1つ前の[証明](../expectation_decomposition_proof/_index.md)では、以下のことが成立した。

$$
\mathbb{E} _{p(\mathbf{x}, y)} [l _{01} (y f(\mathbf{x}))]
= p(y = +1) \mathbb{E} _{p(\mathbf{x} | y = +1)} [ \tilde{l} (f(\mathbf{x})) ] + \mathbb{E} _{p(\mathbf{x})} [ \tilde{l} (f(\mathbf{x})) ]
$$

ここに以下の式を代入する。(いずれも$1 - \delta / 2$以下の確率で成り立つ)ここで、定数$p(y = +1) = \pi ^ *$とおく。

$$
\mathbb{E} _{p(\mathbf{x} | y = +1)} [ \tilde{l} (f(\mathbf{x})) ] \leq \frac{1}{n} \sum _{i = 1} ^ n \tilde{l} (f(\mathbf{x} _i)) + \sqrt{\frac{\log (2 / \delta)}{8n}} \\\\ 
\mathbb{E} _{p(\mathbf{x}, y)} [ \tilde{l} (y f(\mathbf{x})) ] \leq \frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} (y ^ \prime _i f(\mathbf{x ^ \prime _i})) + \sqrt{\frac{\log (2 / \delta)}{2 n ^ \prime}}
$$

すると、上記の式は$(1 - \frac{\delta}{2}) ^ 2 > 1 - \delta$以上の確率で、このようなかたちの不等式が成り立つと言える。

$$
\mathbb{E} _{p(\mathbf{x}, y)} [l _{01} (y f(\mathbf{x}))] \leq \pi ^ * (\frac{1}{n} \sum _{i = 1} ^ n \tilde{l} (f(\mathbf{x} _i)) + \sqrt{\frac{\log (2 / \delta)}{8n}}) + \frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} (y ^ \prime _i f(\mathbf{x ^ \prime _i})) + \sqrt{\frac{\log (2 / \delta)}{2 n ^ \prime}} \\\\ 
= \frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} (y ^ \prime _i f(\mathbf{x ^ \prime _i})) + (\frac{\pi ^ *}{2 \sqrt{n}} + \frac{1}{\sqrt{n}})\sqrt{\frac{\log(2 / \delta)}{2}} + \pi ^ * \frac{1}{n} \sum _{i = 1} ^ n \tilde{l} (f(\mathbf{x} _i))
$$

これを移項することで、定理1の主張が得られる。

$$
\mathbb{E} _{p(\mathbf{x},y)}\left[ l _{01}(yf(\mathbf{x})) \right] - \frac{1}{n ^ \prime} \sum _{j=1}^{n\prime} \tilde{l} (y ^ \prime _j f(\mathbf{x} ^ \prime _j)) \leq \frac{\pi^*}{n} \sum _{i=1}^{n} l(f(\mathbf{x} _i)) + ( \frac{\pi ^ *}{2\sqrt{n}} + \frac{1}{\sqrt{n ^ \prime}}) \sqrt{\frac{\ln(2/\delta)}{2}}.
$$