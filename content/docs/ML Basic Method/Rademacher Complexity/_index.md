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

参考書一覧

- Foundations of Machine Learning
- 統計的学習理論(MLPシリーズ)

## VC Dimension

モデル=仮説集合$\mathcal{H}$が存在し、その中で識別器=仮説$h \in \mathcal{H}$が存在し、これが二値分類をする。データ$\mathbf{x}$に対して、ラベル$y$があり、これに分類器$h(\mathbf{x})$の出力ができるだけ合致することが望ましく、それが学習である。モデル$\mathcal{H}$のなかで、1通りの$\mathbf{y}$に対応できることを+1として、その個数を$\Pi _{\mathcal{H}}(\mathbf{x} _1, \cdots, \mathbf{x} _n)$とおく。ここで、**パラメタは連続値であるので理論上は無限通りの$\mathcal{H}$のパラメタの取り方が存在するが、同じラベル付け$\mathbf{y}$は1通りとまとめる**。

データが$n$個存在するとして、$2 ^ n$パターンのラベルの付き方が存在する。これについて、$n$が小さければ$\Pi _{\mathcal{H}} (\mathbf{x} _1, \cdots, \mathbf{x} _n) = 2 ^ n$が成り立つ。すなわち、ありうるすべてのラベルの付け方$\mathbf{y}$に対して、パラメタを調節すれば必ずそれを実現する識別器$h \in \mathcal{H}$が存在するということである。

$n$が増加していくにあたり、この等式が満たされなくなる=表現力が足りなくなる。この時の**最大の$n$をVC次元**という。

### Sauerの補題

VC次元が$d$の$\mathcal{H}$について、$\forall n \leq d$について、

$$
\max _{\mathbf{x} _1, \cdots, \mathbf{x} _n} \Pi _{\mathcal{H}} (\mathbf{x} _1, \cdots, \mathbf{x} _n) \leq (\frac{en}{d}) ^ d
$$

これにより、VC次元が$d$の時、データ数が$n$ならばたかだか$d$次の多項式$O(n ^ d)$になるとわかる。

続く(まだ終わっていない...)

## Rademacher Complexity

ラデマッハ複雑度とは、ランダムなノイズに対して学習させるとき、モデルがどれほど追随できるか、という量である。大きいほど表現力が高いモデルである。

ラデマッハ変数という、1/2の確率で+1, 1/2の確率で-1を取る確率変数$\sigma$を考える。厳密には$m$個の訓練データ$\mathbf{x} _i$が存在するときのラデマッハ複雑度は以下のように定義する。$g$は学習器であり、$G$は学習器の集合=モデル。

$$
\mathcal{\hat{R}}(G) = \mathbb{E} _{\boldsymbol{\sigma}} [ \sup _{g \in G} \frac{1}{m} \sigma _i g(\mathbf{x} _i)] \\\\ 
= \mathbb{E} _{\boldsymbol{\sigma}} [ \sup _{g \in G} \frac{\boldsymbol{\sigma} ^ T \mathbf{g} _\mathbf{x}}{m} ]
$$

ここで、有限個$m$個の訓練データで得られた経験的なラデマッハ複雑度は、訓練データを増やした時には、理想的な分布から得られたときと同じ値になる。つまり、**不偏推定量**。

$$
\mathcal{R}(G) = \\lim _{m \to \infty} \mathcal{\hat{R}}(G)
$$

## Taragrandの補題

$\mathcal{R}(G)$のなかで、$G \to \phi(G)$と写像の合成となったときに使える補題。

$\phi : \mathbb{R} \to \mathbb{R}$であり、リプシッツ連続であるとする。この時、リプシッツ定数$L$であるとして、以下の不等式が成り立つ。

$$
\mathcal{R} (\phi(G)) \leq L \mathcal{R} (G)
$$

証明を書く気力はなかった。

## McDiarmidの不等式

$$
Pr( f(X _1, \cdots, X _N) - \mathbb{E} [ f(X _1, \cdots, X _N) ] \geq \epsilon) \leq \exp( -\frac{2 \epsilon ^ 2}{\sum _{i = 1} ^ N c _i ^ 2})
$$

$c _i$は次の条件を満たす。$X _i$を同じく分布に従う別の確率変数$Y$に変えた時、$f$の取る値との差の上界にあたる。

$$
c _i = \sup _{Y} \lbrace |f(X _1, \cdots, X _{i - 1}, X _i, X _{i + 1}, \cdots, X _n) - f(X _1, \cdots, X _{i - 1}, Y, X _{i + 1}, \cdots, X _n)| \rbrace
$$

ここで、$f$がN変数関数であるが、とりわけ$\mathbf{x} _i$を引数に取って、識別器による経験平均$\sum _{i = 1} ^ N g(\mathbf{x} _i)$である場合を考える。そして、$\forall \mathbf{x}, g(\mathbf{x}) \in [a, b]$と識別器の関数の上界が決まっているのであれば、この式は次のように書くことができる。

$$
Pr( \sum _{i = 1} ^ N g(\mathbf{x} _i) - \mathbb{E} [ g(\mathbf{x}) ] \geq \epsilon) \leq \exp( -\frac{2 \epsilon ^ 2 N }{ (b - a) ^ 2})
$$

そして、一般的には$1 - \delta$以上の確率で○○が成り立つ、という形なので、それに式変形する。

$$
\delta = \exp( -\frac{2 \epsilon ^ 2 N}{(b - a) ^ 2}) \\\\ 
\log \delta = - \frac{2 \epsilon ^ 2 N }{(b - a) ^ 2} \\\\ 
(b - a) ^ 2 \log (1 / \delta) = 2 \epsilon ^ 2 N \\\\ 
\frac{(b - a) ^ 2}{2N} \log (1 / \delta) = \epsilon ^ 2 \\\\ 
(b - a) \sqrt{\frac{\log (1 / \delta)}{2N}} = \epsilon
$$

このことから、$\forall \delta, 1 - \delta$以上の確率で、下式が成り立つ。

$$
\sum _{i = 1} ^ N g(\mathbf{x} _i) - \mathbb{E} [ g(\mathbf{x}) ] \leq (b - a) \sqrt{\frac{\log (1 / \delta)}{2N}}
$$

特に、写す空間が$[0, 1]$である場合は、以下のように更に略して×。

$$
\sum _{i = 1} ^ N g(\mathbf{x} _i) - \mathbb{E} [ g(\mathbf{x}) ] \sqrt{\frac{\log (1 / \delta)}{2N}} = \sqrt{\frac{\log (1 / \delta)}{2N}}
$$

## 一様大数の法則

次に重要な定理として、ラデマッハ複雑度を用いた二値分類識別器$g \in \mathcal{G}$の期待値を上から抑えられ、不偏推定量からたかだかどれほどずれるのかを評価できる。これは**一様大数の法則**と呼ばれている。$g$は集合$\mathcal{Z} \to [b, a]$への写像。なお、$\forall Z \in \mathcal{Z}$に所属している。また、$Z _i$はそれぞれ$Z$と独立同分布に従う。$\forall \delta, 1 - \delta$以上の確率で以下の式が成り立つ。

$$
\sup _{g \in \mathcal{G}} \lbrace \mathbb{E} [g(Z)] - \sum _{i = 1} ^ N g(Z _i) \rbrace \leq 2 \mathcal{R} _N (\mathcal{G}) + (b - a) \sqrt{\frac{\log(1 / \delta)}{2N}}
$$

これを絶対誤差で評価した版は以下である。

$$
\sup _{g \in \mathcal{G}} \lbrace | \mathbb{E} [g(Z)] - \sum _{i = 1} ^ N g(Z _i) | \rbrace \leq 2 \mathcal{R} _N (\mathcal{G}) + (b - a) \sqrt{\frac{\log(2 / \delta)}{2N}}
$$

### 証明

$A(z _1, \cdots, z _n) = \sup _{g \in \mathcal{G}} \lbrace \mathbb{E} [ g(Z) ] - \frac{1}{N} \sum _{i = 1} ^ N g(z _i) \rbrace$とする。

ここで、$z _n$が$z ^ \prime$になるとする(同様に$\mathcal{Z}$に従う値)。この時、1つだけ変更したとき、上界は以下のようになる。中でのsupは、外の符号が負なので、一番外に出すと最小値のinfになる。infに関しては$f$でなくても、$g$にするとinfじゃなくなって少し大きくなるので、符号は$\leq$となれる。

$$
A(z _1, \cdots, z _n) - A(z _1, \cdots, z ^ \prime)
= \sup _{g \in \mathcal{G}} \lbrace \mathbb{E} [g(Z)] - \frac{1}{n} \sum _{i = 1} ^ n g(z _i) - \inf _{f \in \mathcal{G}} \lbrace \mathbb{E} [f(Z)] - \frac{1}{n} \sum _{i = 1} ^ {n - 1} f(z _i) - \frac{1}{n} f(z ^ \prime) \rbrace \rbrace \\\\ 
= \sup _{g \in \mathcal{G}} \inf _{f \in \mathcal{G}} \lbrace \mathbb{E} [g(Z)] - \frac{1}{n} \sum _{i = 1} ^ n g(z _i) - \mathbb{E} [f(Z)] + \sum _{i = 1} ^ {n - 1} f(z _i) + \frac{1}{n} f(z ^ \prime) \rbrace \\\\ 
\leq \sup _{g \in \mathcal{G}} \lbrace \mathbb{E} (g(Z)) - \frac{1}{n} \sum _{i = 1} ^ n g(z _i) - \mathbb{E} [ g(Z) ] - \frac{1}{n} \sum _{i = 1} ^ {n - 1} g(z _i) + \frac{1}{n} g(z ^ \prime) \rbrace \\\\ 
= \sup _{g \in \mathcal{G}} \lbrace \frac{g(z ^ \prime) - g(z _n)}{n} \rbrace = \frac{b - a}{n}
$$

これにより、マルチンゲールは$(b - a) / n$となる。これはどの$i$に対しても成り立ち、しかも$A(z _1, \cdots, z ^ \prime) - A(z _1, \cdots, z ^ \prime) \leq \frac{b - a}{n}$も成り立つので、全体で言うと

$$
|\frac{g(z ^ \prime) - g(z _n)}{n}| \leq \frac{b - a}{n}
$$

このことから、先ほどのMcDiarmidの不等式を用いると以下のような集中不等式を得られる。$\forall \delta, 1 - \delta$以上の確率で、以下が成り立つ。**$A$自体に対して不等式を適用している**ということに**注目**。$A(z _1, \cdots, z _n) = \sup _{g \in \mathcal{G}} \mathbb{E} [g(Z)] - \frac{1}{n} \sum _{i = 1} ^ n g(z _i)$であり、**この中の$g$に対してMcDiarmidの不等式を適用するのではない**！なお、普通に$g$で不等式を使うと、$\mathbb{E} [g]$の評価ができなくなるので、このような評価できるかたちで不等式を使った。

$$
A(Z _1, \cdots, Z _n) - \mathbb{E} [A] \leq (b - a) \sqrt{\frac{\log(1 / \delta)}{2n}}
$$

次に、$Z _1, \cdots, Z _n$のみならず、$Z _1 ^ \prime, \cdots, Z _n ^ \prime$も$\mathcal{Z}$に従う確率変数とする。これを使えば、$\mathbb{E} [g]$を$n$個の確率変数に分割できる。つぎに、期待値を外に出すことで、$\leq$の形を作れる。

$$
A(Z _1, \cdots, Z _n) = \sup _{g \in \mathcal{G}} \lbrace \mathbb{E} [ g(Z) ] - \frac{1}{n} \sum _{i = 1} ^ n g(Z _i) \rbrace \\\\ 
= \sup _{g \in \mathcal{G}} \lbrace \mathbb{E} _{Z _1 ^ \prime, \cdots, Z _n ^ \prime} [\frac{1}{n} \sum _{i = 1} ^ n g(Z _i ^ \prime)] -\frac{1}{n} \sum _{i = 1} ^ n g(Z _i) \rbrace \\\\ 
\leq \mathbb{E} _{Z _1 ^ \prime, \cdots, Z _n ^ \prime} [ \sup _{g \in \mathcal{G}} \lbrace \frac{1}{n} \sum _{i = 1} ^ n g(Z _i ^ \prime) - \frac{1}{n} \sum _{i = 1} ^ n g(Z _i)  \rbrace ] \\\\ 
= \mathbb{E} _{Z _1 ^ \prime, \cdots, Z _n ^ \prime} [ \sum _{i = 1} ^ n \sup _{g \in \mathcal{G}} \lbrace \frac{1}{n} (g(Z _i ^ \prime) - g(Z _i) )\rbrace ]
$$

ここで、対称性により、$g(Z _i) - g(Z _i ^ \prime)$も$g(Z _i ^ \prime) - g(Z _i)$も同じ分布を持っている(同じ$\mathcal{Z}$から来ているし、$g$も1つである)。だから、**符号を自由にflipさせてもいい**ということになる！**符号を自由にflipさせるというのならば、まさにラデマッハ変数じゃないか**！あとは三角不等式で評価すればラデマッハ複雑度が出てくる！

$$
\mathbb{E} [ A(z _1, \cdots, z _n) ] = \mathbb{E} _{Z _1, \cdots, Z _n} [ \sup _{g \in \mathcal{G}} \lbrace \mathbb{E} [ g(Z) ] - \frac{1}{n} \sum _{i = 1} ^ n g(z _i) \rbrace ] \\\\ 
\leq \mathbb{E} _{Z _1, \cdots, Z _n} [ \mathbb{E} _{Z _1 ^ \prime, \cdots, Z _n ^ \prime} [ \sup _{g \in \mathcal{G}} \lbrace \frac{1}{n} (g(Z _i ^ \prime) - g(Z _i) )\rbrace ] ] \\\\ 
\leq \mathbb{E} _{\sigma _1, \cdots, \sigma _n} [\mathbb{E} _{Z _1, \cdots, Z _n} [ \mathbb{E} _{Z _1 ^ \prime, \cdots, Z _n ^ \prime} [ \sup _{g \in \mathcal{G}} \lbrace \frac{\sigma _i}{n} (g(Z _i ^ \prime) - g(Z _i) )\rbrace ] ] ] \\\\ 
= \mathbb{E} _{Z _1, \cdots, Z _n} [ \mathbb{E} _{Z _1 ^ \prime, \cdots, Z _n ^ \prime} [ \mathbb{E} _{\sigma _1, \cdots, \sigma _n} [\sup _{g \in \mathcal{G}} \lbrace \frac{\sigma _i}{n} (g(Z _i ^ \prime) - g(Z _i) )\rbrace ] ] ] \\\\ 
\leq \mathbb{E} _{Z _1 ^ \prime, \cdots, Z _n ^ \prime} [ \mathbb{E} _{\sigma _1, \cdots, \sigma _n} [\sup _{g \in \mathcal{G}} \lbrace \frac{\sigma _i}{n} g(Z _i ^ \prime) \rbrace ] ] + \mathbb{E} _{Z _1, \cdots, Z _n} [\mathbb{E} _{\sigma _1, \cdots, \sigma _n} [\sup _{g \in \mathcal{G}} \lbrace \frac{-\sigma _i}{n} g(Z _i )\rbrace ] ] \\\\ 
\leq \mathcal{R} _n(\mathcal{G}) + \mathcal{R} _n(\mathcal{G}) = 2 \mathcal{R} _n(\mathcal{G})
$$

このように、上から評価できた！

これを使えば、先ほどのMcDiarmidの不等式から以下の結果が得られる。これこそが、一様大数の法則。

$$
A(Z _1, \cdots, Z _n) = \sup _{g \in \mathcal{G}} \lbrace \mathbb{E} [g(Z)] - \sum _{i = 1} ^ n g(Z _i) \rbrace \leq 2 \mathcal{R} _n (\mathcal{G}) + (b - a) \sqrt{\frac{\log(1 / \delta)}{2n}}
$$

絶対値版に対しては、ここまでの議論で$\delta$としていたものを$\delta / 2$に置き換えると、以下の2つの式が得られる。いずれも$\forall \delta, 1 - \delta / 2$以上の確率で成立する。

$$
\sup _{g \in \mathcal{G}} \lbrace \mathbb{E} [g(Z)] - \sum _{i = 1} ^ n g(Z _i) \rbrace \leq \sup _{g \in \mathcal{G}} \lbrace |\mathbb{E} [g(Z)] - \sum _{i = 1} ^ n g(Z _i)| \rbrace \leq 2 \mathcal{R} _n (\mathcal{G}) + (b - a) \sqrt{\frac{\log(1 / \delta)}{2n}} \\\\ 
\sup _{g \in \mathcal{G}} \lbrace \sum _{i = 1} ^ n g(Z _i) - \mathbb{E} [g(Z)] \rbrace \leq \sup _{g \in \mathcal{G}} \lbrace |\mathbb{E} [g(Z)] - \sum _{i = 1} ^ n g(Z _i)| \rbrace \leq 2 \mathcal{R} _n (\mathcal{G}) + (b - a) \sqrt{\frac{\log(1 / \delta)}{2n}}
$$

この2つの式を足し合わせる事で、2で割ると$(1 - \delta / 2) ^ 2 = 1 - \delta + (\delta ^ 2 / 4) \geq 1 - \delta$以上の確率で、

$$
\sup _{g \in \mathcal{G}} \lbrace |\mathbb{E} [g(Z)] - \sum _{i = 1} ^ n g(Z _i)| \rbrace \leq 2 \mathcal{R} _n (\mathcal{G}) + (b - a) \sqrt{\frac{\log(1 / \delta)}{2n}} 
$$