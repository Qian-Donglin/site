---
title: "PU LearningでなんでSVMのヒンジ関数は精度悪いか(2014)"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[元論文](https://proceedings.neurips.cc/paper_files/paper/2014/file/35051070e572e47d2c26c241ab88307f-Paper.pdf)

[参考したスライド](https://www.slideshare.net/Quasi_quant2010/quasi-quant2010-2)

[参考にした記事](https://nnkkmto.hatenablog.com/entry/2020/12/23/000000)

[**親戚論文の中国語解説　とっても良い**](https://blog.csdn.net/crazy_scott/article/details/88993441)

これも東大杉山研じゃないか、たまげたな。

[前にやったPU Learning]()は、強い分布に対しての仮定が必要だった。つまり、**PositiveなDataのうちから一様に抽出して、ラベル付きになっているという強い分布への仮定が必要**だった。今回のやつはそこへのカウンターも兼ねた理論的な証明。

# 問題設定

Positive(+1)とNegative(-1)の誤分類を最小化する関数は以下のように書ける。これは、**誤分類の割合**を表す関数。

まず、あるデータ$X$について、ラベル付けする識別器$f(X) \in 1, -1$があるとする。また、

- $R_{-1}(f) = P_{-1} (f(X) \neq -1)$　　**本来Negative=-1なのに、識別器にPositive=1に分類されてしまう確率**。
- $R_{1}(f) = P_{1} (f(X) \neq 1)$　　**本来Positive=1なのに、識別器にNegative=-1に分類されてしまう確率**
- $\pi$は**全sampleのうちのpositiveの割合**であり、$\frac{n_{pos}}{n_{pos} + n_{neg}}$で推定する。

$$
R(f) = \pi R_1(f) + (1-\pi) R_{-1}(f)
$$

また、ここから更にcost-sensitive=重み付き分類は、上記の式にcostの$c_1, c_{-1}$をつけたもの。これは以下のもの

$$
R(f) = \pi c_1 R_1(f) + (1-\pi) c_{-1} R_{-1}(f)
$$

# PU分類

上の式はPositive=1とNegative=-1だったが、PU分類ではUnknownにNegativeが全部入ってるし、Positiveも一部ある。$\pi$は前述のとおり、全sampleでのPositiveな割合。(**Positiveなラベルしかつけないというけどさすがにsampleを見る限りNegativeが何個あったかも把握はしておくので、πが求まる**。これがラベルなしのデータでも同じ割合であると推定)この時、↓の式のように、**ラベル付けされてないものの確率を求めることができる**。

$$
P_X = \pi P_1 + (1-\pi) P_{-1}
$$

次に、$R_X(f)$=**分類器f(X)が、$P_X$に対してその中でPositiveの確率**として、求めてみる。つまり$\pi$では？と思うが、$\pi$は**全体のPositiveな真の割合であり、$R_X(f)$は推定してる**といえる。
これ、ある訓練データから一部をPositiveとしてラベル付けしておくかたちだが、**多分ラベルなしは、訓練データでのNegativeを除く。排除しないと、割合が$\pi$で推定できないから(Negativeが予想以上に混入するので)**。

$$
R_X (f) = P_X (f(X) = 1) = \pi P_1(f(X) = 1) + (1 - \pi) P_{-1} (f(X) = 1)
$$

$$
= \pi(1 - R_1(f)) + (1 - \pi) R_{-1}(f)
$$

となる。この$R_X$、**ラベルなしのクラス$P_X$のうち、識別器にpositiveと認識される確率**である(再掲)だが、これを使って$R(f)$も表してみる。なんせPU分類には$R_{-1}$は分からないから、できるだけ消したい。重みは一旦なしで見る。

$$
R(f) = \pi R_1(f) + (1-\pi) R_{-1}(f) 
$$

$$
= \pi R_1(f) + R_X(f) - \pi(1 - R_1(f))　= 2\pi R_1(f) + R_X(f) - \pi
$$

そして、$\mu$**を$P_x$に対して$P_1$が占める割合**とする。今までは$\pi$で、Negativeも入れた中でのSampleのPositiveの割合だった。しかし、$\mu$は、ラベル付き=Positiveとラベルなし(**Negativeの割合は使わない**！)ということ。

この$\mu$という量を使って、$R(f)$を無理やり式変形してみる。

$$
R(f) = \frac{2 \pi}{\mu} \cdot \mu R_1(f) + \frac{1}{1 - \mu} \cdot (1 - \mu) R_X(f) - \pi
$$

このように、はじめにいった重み付きのPN分類に帰着できるとわかるね。

# 損失関数について

ヒンジ損失関数を考える。

$$
\max (0, 1 - y_i(\mathbf{w}^T \mathbf{x}_i + \mathbf{\theta}))
$$

ランプ、ReLUの損失関数を考える。

$$
\max (0, \min(2, 1 - y_i(\mathbf{w}^T \mathbf{x}_i + \mathbf{\theta})))
$$

## ヒンジ関数ではどうなるか。

$$
R(f) = 2\pi R_1(f) + R_X(f) - \pi
$$

これに、ヒンジ関数を適用する。

$$
R(f) = 2 \pi P_{1} (f(X) \neq 1) + 
$$

続く

結果論、ヒンジ関数はダメだけど、ReLUがいいって。

# PU分類の誤差 in 同一分布じゃない

注意：他の論文を読んでたら、ここでは前提として「**Positiveでラベル付けされてるものとされてないものは同じ分布に従う**」とあるらしい。**じゃあこれはなんだ**...?

PU分類は、同一分布に従う必要がある。つまり、ラベル付けでPositiveになってるのは、Positive全体のデータからランダムに抽出させないとアカン。
では、同一分布に従ってない時(=ランダムに抽出してない？)はどうする？**その時の誤差はどうなるのか**、を解析したのがこの論文。

関数$f(\mathbf{x})$を次のように定める。

$$
f(\mathbf{x}) = \sum _{i = 1}^{n} \alpha_i k(\mathbf{x}_i, \mathbf{x}) + \sum _{j = 1}^{n ^\prime} \alpha_j ^\prime k(\mathbf{x}_j^\prime, \mathbf{x})
$$

- $\alpha_1, \cdots, \alpha_n, \alpha_1^\prime, \cdots, \alpha_{n^\prime}^\prime$は実数。
- $\mathbf{x}_1, \cdots, \mathbf{x}_n$は、$p(\mathbf{x} | y = +1)$。**Positiveの分布に従い、ランダムに$\mathbf{x}_i$を$n$個抽出**した。
- $\mathbf{x}_1^\prime, \cdots, \mathbf{x} _{n^{ \prime }} ^ \prime$は、$p(\mathbf{x})$。つまり、これは**Positive、Negative関係なくランダムに$\mathbf{x}_i^\prime$を$n^\prime$個抽出**した。

つまり、$n$個のPositiveと、$n^\prime$個のラベルなしがあるとして、それらと引数$\mathbf{x}$のカーネル内積の一定係数の線形結合。この関数での誤差を考えてみる。

これをごにょごにょすると(中略...)

一定分布に従わないという最悪な状況でも、誤差は

$$
O(\frac{1}{\sqrt{n}} + \frac{1}{\sqrt{n^\prime}})
$$

のオーダーになる。これは$n$個が独立同分布に従って得られ、$n^\prime$個がまた別の独立同分布に従ってる場合に最適である。

もし、いずれも同じ独立同分布に従ってるなら、

$$
O(\frac{1}{\sqrt{n + n^\prime}})
$$

だが、これは非現実的。同じ独立同分布に従ってるなら、PU Learningする意味ないので。

ただ、これを見る限り、**どれほど分布が違っていようが**、完全に一致の分布から$n$と$n^\prime$を取ってるのと比べて、**たかだか$2 \sqrt{2}$倍までしか悪くならない！**

$$
\frac{1}{\sqrt{n}} + \frac{1}{\sqrt{n ^ \prime}} = = \frac{\sqrt{n} + \sqrt{n ^ \prime}}{\sqrt{n n^\prime}}
$$