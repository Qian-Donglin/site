---
title: "2014-NIPS-Analysis of Learning from Positive and Unlabeled Data"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
bookCollapseSection: true
# bookComments: false
# bookSearchExclude: false
---

[元論文](https://proceedings.neurips.cc/paper_files/paper/2014/file/35051070e572e47d2c26c241ab88307f-Paper.pdf)

[参考したスライド](https://www.slideshare.net/Quasi_quant2010/quasi-quant2010-2)

[参考にした記事](https://nnkkmto.hatenablog.com/entry/2020/12/23/000000)

[**親戚論文の中国語解説　とっても良い**](https://blog.csdn.net/crazy_scott/article/details/88993441)

[SVMの総集編論文](https://www.csie.ntu.edu.tw/~cjlin/papers/libsvm.pdf)

In any way, this article is the only one to explain the generalization error bounds for PU classification in the paper. I suggest everyone who tries to do research for PU Learning to read this article. 

PU Learningは結局、条件の下でのコスト最小化による最適化なので、SVMと同じようにやることができる。

この論文では以下の2点が判明した。

1. 凸関数(ヒンジ損失など)は、たとえ2つのクラスが完璧に分割できる場合でも、上手くいかないことがある。解決策は非凸の損失関数(ランプ損失など)を使うといい。
2. 事前分布$p(y = +1)$を推定するとき、分類誤差は真の$p(y = +1)$と推定クラス自演分布の両方に依存する有効クラス事前分布というものに依存する(これなに？)
3. 一般的な分類誤差の上界をMcDirmidの不等式で抑え、完全な教師ありと比べてもたかだか$2 \sqrt{2}$倍までしか悪くならないと示した。

## PU Learningはいかにして損失最小化となるのか。

[ここにある説明](../_index.md/#puの損失関数の定式化)を読むとわかりやすい。

## 非凸な損失関数がなぜ必要か

### 通常の分類で使われる損失関数

一番いいのは01損失だが、傾きは全くないので**代理損失関数**を使わないといけない。よく使われるのはランプ損失

$$
l _R (z) = \frac{1}{2} \max(0, \min(2, 1 - z))
$$

ランプ損失は非凸関数であるが、凸関数にしたかったらヒンジ損失

$$
l _H (z) = \frac{1}{2} \max(1 - z, 0)
$$

が候補となる。**完全に分類できる前提**では、非凸のramp損失によるリスク関数が0であるなら凸のhinge損失によるリスク関数も0になる。
そのうえで、凸性のあるhingeを使った方が勾配降下法による最適化にいいよね、というかたちなのでhinge損失が使われてきた。

### PU Learningにおける損失関数

PU Learningのリスク関数から損失関数の形を代入すると、以下のような形に。損失関数$l$は正の値を与えられると0だが、負になると大きくなるという性質を持つ。

ここで、リスク関数は最初から確率と定義されていることが重要である。**損失に01損失を使う限り、損失関数の期待値は確率(or条件付確率)と等しいからである**。このことから$1 - R _{fn}(g)$は、1から擬陰性の確率を引いたもので、**真にpositiveが分類器$g$で正しくpositiveに分類される確率**に該当する。
そして、$\mathbb{E} _{+1}[ l(-g(\mathbf{x})) ]$は、**正しく分類できたものが損失関数で正値を出しそれの期待値**に該当するので、**まさに$1 - R _{fn}(g)$に該当する**のである！

$$
R(g) = 2\pi R _{fn}(g) + R _{X}(g) - \pi \\\\ 
= 2 \pi \mathbb{E} _{+1} [ l(g(\mathbf{x})) ] + \pi (1 - \mathbb{E} _{+1} [ l(g(\mathbf{x})) ]) + (1 - \pi) \mathbb{E} _{-1}[ l(-g(\\mathbf{x})) ] - \pi \\\\
= 2 \pi \mathbb{E} _{+1} [ l(g(\mathbf{x})) ] + \pi \mathbb{E} _{+1} [ -l(g(\mathbf{x})) ] + (1 - \pi) \mathbb{E} _{-1}[ l(-g(\\mathbf{x})) ] - \pi \\\\ 
= \pi \mathbb{E} _{+1} [l(g(\mathbf{x}))] + (1 - \pi) \mathbb{E} _{-1}[ l(-g(\\mathbf{x})) ] - \pi + \pi \mathbb{E} _{+1}[ l(g(\mathbf{x})) + l(-g(\mathbf{x}))]
$$

ここで、最後の項の$l(g(\mathbf{x})) + l(-g(\mathbf{x}))$について、ramp損失とhinge損失それぞれの性質を考えてみる。
ここで、**うまいこと定数となった項は削除することができ、項の減少は最適化の局所最適解に陥る確率が同等かそれ以上に改善される**ことを意味している。

- ramp損失は$l(z) + l(-z) = 1$
- hinge損失は$l(z) + l(-z) = 1, z \in [-1, 1]$ それ以外は**定数ではない**！

このように、**PU Learningのリスク関数の定式化では、凸最適化がしやすいHinge損失では逆に項を削ることができずに、最適化が難しくなっていることを示唆している**。

なお、ランプ損失は、Convex-Concave Procedureという凸関数と凹関数によって構成される式の最適化を行う手法で無事に計算できる。

## 事前確率$\pi = p(y = +1)$の不正確な推定が及ぼす影響

cost-sensitiveでPU Learningを行うにあたり、$\pi = p(y = +1)$の正確な推定が必要。これが$\hat{\pi}$に推定された時どれほど誤差が起きうるのかを評価する。ラベルがすべて明確なPN Learningとみなしたとき、リスク関数は偽陰性$R _{fn}$と偽陽性$R _{fp}$を用いて、以下のように書けた。この時、モデル$\mathcal{G}$においてリスクを最小化する$g$がある。

$$
R(\pi, g) = \pi R _{fn}(g) + (1 - \pi) R _{fp}(g) \\\\ 
R ^ {*} (\pi) = \min _{g \in \mathcal{G}} R(\pi, g)
$$

これを経験的に予測して、ハットをつけると以下のようになる。

$$
\hat{R}(\hat{\pi}, g) = \pi R _{fn}(g) + (1 - \pi) R _{fp}(g) \\\\ 
R ^ {*} (\pi) = \min _{g \in \mathcal{G}} R(\pi, g)
$$

PU Learningでのリスク関数は以下のように評価できる。ここでは、推定した$\hat{\pi}$を導入する。

$$
R(g) = 2 \hat{\pi} R _{fn}(g) + R _X(g) - \hat{\pi}
$$

$R _X(g)$自体はこの論文で$R _{fn}, R _{pn}$で、$R _X(g) = \pi(1 - R _{fn}(g)) + (1 - \pi) R _{fp}(g)$と定義できるとわかっているので、代入すると以下のようになる。この定義の部分では、推定した$\hat{\pi}$を使う余地がない($R _X$は理想的な状況から計算する)。

$$
R (g) = 2 \hat{\pi} R _{fn}(g) L \pi(1 - R _{fn}(g)) L (1 - \pi) R _{fp}(g) - \hat{\pi} \\\\ 
= (2 \pi - \pi) R _1(g) + (1 - \pi) R _{fp} (g) + \pi - \hat{\pi}
$$

このことから、$\hat{\pi} \leq \frac{1}{2} \pi$の時は、$R _{fn}$(g)の係数が0以下になるので、**PU Learningができなくなる**とわかる。

また、上の誤差入り版の$R _{fn}, R _{pn}$の係数を同様に$\tilde{\pi}, 1 - \tilde{\pi}$の$R(g)$の形と見てみると、以下のようになる。この$\tilde{\pi}$は理想的には$\pi$に近づくことになる。

$$
\tilde{\pi} = \frac{2 \hat{\pi}}{2 \hat{\pi} - \pi + 1 - \pi} = \frac{2 \hat{\pi}}{2 \hat{\pi} - 2\pi + 1}
$$

グラフにすると以下のようになる。より$\pi$が大きいほど、$\hat{\pi}$の推定の範囲が広まっていてもだいたい平坦であるとわかる。

![](effect-of-estimation-pi.png)

結論として、**$\pi$が大きいとき、$\hat{\pi}$の推定がガバガバでも大して問題は起きづらいが、$\pi$が小さいときはすぐ響いてくる**。

## PU Learningの誤差境界の理論的解析

$\mathbf{x} _1 \cdots \mathbf{x} _n \sim p(\mathbf{x} | y = +1), \mathbf{x} ^ \prime _ 1, \cdots, \mathbf{x} ^ \prime _ {n ^ \prime} \sim p(\mathbf{x})$がPositiveデータとUnlabeledデータと定義する。

分類器$f(\mathbf{x})$はカーネル法を使用したSVMだと想定して、係数$\alpha _ i, \alpha ^ \prime _ i$以下のような一般形で表す。

$$
f(\mathbf{x}) = \sum _{i = 1} ^ n \alpha _ i k(\mathbf{x}, \mathbf{x} _ i) + \sum _ {j = 1} ^ {n ^ \prime} \alpha ^ \prime _ j k(\mathbf{x}, \mathbf{x} ^ \prime _ j)
$$

そして、01損失の代理損失関数として、ランプ損失を以下のように定義する。

$$
l_n(z) = 
\begin{cases} 
0 & \text{if } z > \eta, \\\\ 
1 - \frac{z}{\eta} & \text{if } 0 \leq z \leq \eta, \\\\ 
1 & \text{if } z \leq 0.
\end{cases}
$$

その上、通常の損失関数$l _{01}, l _{\eta}$に対して、以下の損失関数を天下り的に定義する。

$$
\tilde{l}(y f(\mathbf{x})) = \frac{2}{y + 3} l _{01} (y f(\mathbf{x})) \\\\ 
\tilde{l} _{\eta} (y f(\mathbf{x})) = \frac{2}{y + 3} l _{\eta} (y f(\mathbf{x}))
$$

このような定義では、$y = -1$の時は1倍、$y = +1$の時は$1 / 2$倍されることを意味する。

そして、次のように$\mathbb{E} _{p(\mathbf{x}, y)} [l _{01} (y f(\mathbf{x}))]$を分解することができる、というのを利用する。証明は[こちら](./expectation_decomposition_proof/_index.md)。

$$
\mathbb{E} _{p(\mathbf{x}, y)} [l _{01} (y f(\mathbf{x}))]
= p(y = +1) \mathbb{E} _{p(\mathbf{x} | y = +1)} [ \tilde{l} (f(\mathbf{x})) ] + \mathbb{E} _{p(\mathbf{x})} [ \tilde{l} (f(\mathbf{x})) ]
$$

以下の定理1と定理2が存在する。**この2つの定理は01損失と代理損失(ランプ損失)を使用した時のそれぞれの理論的な誤差の上界である**。

### 定理1

先ほどの分解を使用すると、以下のことが得られる。

識別器$\forall f \in \mathcal{F}$がある。PositiveとUnlabeledデータをサンプリングする。
そこで、$\delta \in (0, 1)$として、少なくとも$1 - \delta$以上の確率で下の式が成り立つ。($\pi ^ * = p(y = +1)$)

$$
\mathbb{E} _{p(\mathbf{x},y)}\left[ l _{01}(yf(\mathbf{x})) \right] - \frac{1}{n ^ \prime} \sum _{j=1}^{n\prime} \tilde{l} (y ^ \prime _j f(\mathbf{x} ^ \prime _j)) \leq \frac{\pi^*}{n} \sum _{i=1}^{n} l(f(\mathbf{x} _i)) + ( \frac{\pi ^ *}{2\sqrt{n}} + \frac{1}{\sqrt{n ^ \prime}}) \sqrt{\frac{\ln(2/\delta)}{2}}.
$$

証明は[こちら](./theorem1_proof/_index.md)。

この定理は、損失関数が理想的な$l _{01}$であることを前提とおいている。ただ、現実には代理損失関数を使うことになる。今回の場合はランプ損失にあたる$\tilde{l}$を使用しているので、そちらでの評価もしなければならない。これは定理2である。

### 定理2

$\forall \eta > 0, \delta \in (0, 1)$であるとする。PositiveとUnlabeledデータをサンプリングする。少なくとも$1 - \delta$以上の確率で、識別器$f \in \mathcal{F}$において、以下の式が成り立つ。

$$
\mathbb{E} _{p(\mathbf{x}, y)} [ l _{01} (y f(\mathbf{x})) ] - \frac{1}{n ^ \prime} \sum _{i = 1} ^ {n ^ \prime} \tilde{l} _{\eta} (y _i ^ \prime f(\mathbf{x} _i ^ \prime)) \leq \frac{\pi ^ *}{n} \sum _{i = 1} ^ n \tilde{l} _{\eta} (f(\mathbf{x})) + (\frac{\pi ^ *}{\sqrt{n}} + \frac{2}{\sqrt{n ^ \prime}}) \frac{C _{\alpha} C _{k}}{\eta} + (\frac{\pi ^ *}{2 \sqrt{n}}) \sqrt{\frac{\log(2 / \delta)}{2}}
$$

証明は[こちら](./theorem2_proof/_index.md)。

この定理は一番欲しかったこと。代理損失との差を評価できた。

### 意味すること

この2つの定理はみな$O(\frac{1}{\sqrt{n}} + \frac{1}{n ^ \prime})$である。一方、完全にラベルがわかるPN Learningは1つのルートの中で収まるので、$O(\frac{1}{n + n ^ \prime})$が成り立つ。

$$
\frac{1}{\sqrt{n}} + \frac{1}{\sqrt{n ^ \prime}} \geq 2 \sqrt{2} \frac{1}{\sqrt{n + n ^ \prime}}
$$

つまり、上界をこのような形で評価でき、PU LearningはPN Learningに比べてもオーダーでは$2 \sqrt{2}$倍までしか悪くならないと示せた。

証明は[こちら](./conclusion_proof/_index.md)。