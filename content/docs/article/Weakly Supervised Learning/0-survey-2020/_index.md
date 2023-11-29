---
title: "0-Survey-2020"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[元論文のリンク](https://link.springer.com/article/10.1007/s10994-020-05877-5)

2020年のサーベイ論文。[弱教師学習本](../../../read_book/Machine%20Learning/ML%20from%20Weak%20Supervision/_index.md)よりも、とりわけPUについてまとめている。

## Introduction

PU Learningの現実での応用例

1. パーソナライズされた広告のクリック。押されなかったからと言って興味がないわけではない。
2. 診断されてないからといって、必ずそれにかかっていないことはない。
3. 知識ベースという知識のタプルがある。学習してないタプルだからと言って、その組み合わせは嘘であると言えない。

PU Learningの鍵となるResearch Question

1. PUのデータから学習するという問題にどう帰着するか。
2. 学習アルゴリズムを容易に設計するときには、どのような仮定が必要か。
3. PUのデータからクラス事前確率$p(y = +1)$を推定できるか？それは有用か？
4. PUのデータからどうして上手くモデルを学習できるか。
5. モデルの性能をどう評価すればいいか。
6. 実世界ではどのような応用があるか。
7. 他の機械学習分野へどう影響するか。

## PU Learningの準備

### Positive Negativeによる二値分類

一番理想的なケースである。$\mathbf{x}$の周辺確率分布は以下のように得られる。$\pi = p(y = +1)$

$$
\mathbf{x} \sim f(\mathbf{x}) = \pi f _+(\mathbf{x}) + (1 - \pi) f _- (\mathbf{x})
$$

### Positive Unlabeledによる二値分類

一般的には、PUデータは$(\mathbf{x}, y, s)$とラベル$s$を拡張。1ならばラベル付きである。ラベルが付いてるものは全てPositiveであるという仮定なので、$p(y = +1 | s = +1) = 1$。

### 学習のメカニズム

Positiveのデータは$\mathbf{x} \sim p(\mathbf{x} | y = +1)$から得られる。各Positiveのデータについて、選ばれる確率$p(s = +1 | \mathbf{x}, y = +1)$はPropensity SScoreと呼ばれる。[2008 Elkan, Noto](../PU%20Learning/_index.md)にも指摘しているように、(後述する)$p(s = +1)$がわかる、single-training-setの場合は、一様にPositiveのなかから選ぶのであれば$\mathbf{x}$に依らない定数倍が成立する。

$$
p(\mathbf{x} | s = +1) = p(\mathbf{x} | s = +1, y = +1) = \frac{p(s = +1 | \mathbf{x}, y = +1)}{p(s = +1 | y = +1)} p(\mathbf{x} | y = +1)
$$

### single-training-setシナリオとcase-controlシナリオ

Elkan, Notoらの論文での説明をそのまま持ってきた。

基本的には、いずれのデータセットも分布からデータを得るときはi.i.d.である。

基本的には、ランダムに訓練データは$p(\mathbf{x}, y, s)$の分布から選ばれる(もちろん$p(s = 1 | y = 0) = 0$というのが成り立つ分布)が、$y$の情報を落としてデータとしては$(\mathbf{x}, s)$のみを得る。

今回の場合はsingle-training-setシナリオというものであり、以下の手順で行う。

1. $p(\mathbf{x}, y, s)$の分布から1回、一定数サンプリングしてこれを$D$とする。
2. $D$の中で$s = 1$となる部分をPositiveとする。
3. $D$の中で$s = 0$となった、つまり残りのすべてをUnlabeledとして扱う。

このサンプリング方法では、一度だけのサンプリングして、そこから$p(s = 1)$は計算できる。

一方、case-controlシナリオでは、以下の手順で行う。

1. $p(\mathbf{x}, y, s)$の分布から1回、一定数サンプリングしてこれを$D _1$とする。
2. $D _1$の中で$s = 1$となる部分をPositiveとする。
3. $D _1$の$s = 0$の部分、すなわち残ったすべてを**捨てる**。
4. $p(\mathbf{x}, y, s)$の分布から**もう1回**、一定数サンプリング(さきほどと同数である必要もない)して、これを$D _2$とする。
5. $D _2$の全てをUnlabeledとする。(つまり、single-training-setのように$D _1$の$s = 0$なら全部ここに入るわけではない！)

こちらの方が一般的である。ただし、欠点としては2回のサンプリングの母数は同数ではないし、1回目のサンプリングの母数がいくつなのかもわからないことが通例なので、$p(s = 1)$の推定はそもそもできない。

この論文では**後者を論じる**。大は小を兼ねる！

### クラス事前確率とラベル付けられる確率の関係

[2008 Elkan, Notoら](../PU%20Learning/_index.md)の論文を見ると詳しく解説されている。

## PU Learningするにあたっての仮説

Unlabeledには、本来Negativeと本来Positiveだがラベル付けされなかったものの2つが混入している。故に、**ラベル付けのポリシーとクラスの分布**を仮定しなければならない。

### ラベル付けのポリシー

#### Selected Completely At Random (SCAR)

一様にランダムに選ぶ！これが一番前提として使われている。これはcase-controlシナリオに多く、分布からi.i.d.で選ばれている。
このようにラベル付けすると、通常の二値分類の手法をほぼそのまま転用できるという利点がある。

定式化すると、$\mathbf{x}$に関わらず、選ばれる確率は一定の値を取るということ。[2008 Elkan, Notoら](../PU%20Learning/_index.md)の論文にある通り。

$$
p(s = 1 | \mathbf{x}, y = +1) = p(s = 1 | y = +1) = c \\\\ 
p(s = 1 | \mathbf{x}) = c p(y = +1 | \mathbf{x})
$$

これに基づく重要な性質として以下のようなものがある。

1. $p(y = 1 | \mathbf{x} _1) > p(y = 1 | \mathbf{x} _2) \Leftrightarrow p(s = 1 | \mathbf{x} _1) > p(s = 1 | \mathbf{x} _2)$　順序は常に保たれる。
   1. ラベルが付く確率は一様であるが、そもそもPositiveでなければラベルが付かないということを考慮してこういう形ってことかな？
2. 定数倍なので、$p(s = 1 | \mathbf{x})$を訓練すれば、$p(y = +1 | \mathbf{x})$が得られるようになる。

これの発想は、欠損データは一様にランダムに発生するという仮定から着想されたらしい。

#### Selected At Random

データ$\mathbf{x}$の性質によって、**ラベル付けられる確率が変動する**、というものである。$p(s = 1 | \mathbf{x}, y = +1)$がそれぞれ異なる感じ。

#### Probabilistic Gap

SARの1つの典型例として、Negative例に近いPositive例ほど、選ばれにくくなると仮定する。ラベル付けのされやすさとしては、以下の式で考えられる。

$$
\triangle p(\mathbf{x}) = p(y = +1 | \mathbf{x}) - p(y = -1 | \mathbf{x})
$$

ラベルが付く確率は、狭義単調増加の関数$f$が存在するとして、以下の式から各点のラベル付けされる重みが算出されるとする。

$$
f(\triangle p(\mathbf{x})) = f(p(y = +1 | \mathbf{x}) - p(y = -1 | \mathbf{x}))
$$

そのうえで、**ラベル付けされたかどうかで判断する**観測されるProbabilistic Gapとして、以下のものだと**定義する？**。WHY?

$$
\triangle \tilde{p}(\mathbf{x}) = p(s = 1 | \mathbf{x}) - p(s = 0 | \mathbf{x}) 
=f(\triangle p(\mathbf{x})) p(\mathbf{x}) + f(\triangle p(\mathbf{x})) - 1
$$

重要な性質として以下の2つがある。

1. $\triangle \tilde{p}(\mathbf{x}) \leq \tilde{p}(\mathbf{x})$実際に観測するgapは真のgapより小さい。
2. $\triangle \tilde{p}(\mathbf{x} _1) = \triangle \tilde{p}(\mathbf{x} _2) \Leftrightarrow \triangle p(\mathbf{x} _1) = \triangle p(\mathbf{x} _2) \\\\ \triangle \tilde{p}(\mathbf{x} _1) \leq \triangle \tilde{p}(\mathbf{x} _2) \Leftrightarrow \triangle p(\mathbf{x} _1) \leq \triangle p(\mathbf{x} _2)$　順序性が同様に成り立つ。

これは[2018 Katoら](../Positiveでラベル有り無しで分布が違う時のPU学習(2019)/_index.md)の論文で使われた鍵アイディア。彼らは以下の仮定を使った。

$$
p(y = 1 | \mathbf{x} _1) > p(y = 1 | \mathbf{x} _2) \Leftrightarrow p(s = 1 | \mathbf{x} _1) > p(s = 1 | \mathbf{x} _2)
$$

さきほどのSCARと違うのは、あれは$y = +1, y = -1$のみで成立していた？のだが、これは同じ$y = +1$のデータの間でもこれが成り立つということだろう。

### データにおける仮定

基本はすべてのNegativeと一部のPositiveがUnlabeledであり、残ったPositievだけラベル付けされていること。そして、2つのクラスは分離可能であり、分布の確率分布はなめらかな確率密度関数を持つというもの。

#### Negativity

Unlabeledを全部Negativeとして扱うという非常に乱暴なテクがある。もうこれはさすがに語る余地もないだろ。それはそう。

#### Separability

**2つのクラスを完璧に分離可能にする分類器が存在する**という仮定。この仮定に基づけば、ラベル付けされた例のすべてをできるだけPositiveとして分類することを優先するという方法になる。後述の2-stepsテクニックに活用されるらしい。

#### Smoothness

お互いに近いデータ点は、同じような$y$を持つ(Positive or Negative)。数式で書くと、$\mathbf{x} _1, \mathbf{x} _2$が近いのであれば、$p(y = +1 | \mathbf{x} _1), p(y = +1 | \mathbf{x} _2)$は近い値を取る。

つまり、すべての$s = 1$のデータ点から遠い点こそが信頼できるNegativeの点だと考えられる(ほんとうにNegativeである保証はない)。これも2-stepsテクニックで使われるらしい。これは距離別で教師なし？的な手法を使う時でよく見られるかもしれない。

### クラス事前確率$p(y = +1)$における仮定

SCARにおける重要な役割を果たす。仮定で強い順から並べる。

1. 完全分離可能な分布たち　完全分離できるのならば、未ラベルのうちのPositive例と分布が理論上一致できる。
2. 完全に分離できる一部のドメインを持つ　データ$\mathbf{x}$の**成分の一部**では完全にPositiveだけを含むことができる、という例。のうちの一部が完全に$y = +1$であるとわかれば、そこにおいての$s = 1$の割合から、ラベルが付く頻度がわかる。(これは書いてないがこういうこと？　ラベルの付く頻度から逆算して、クラス事前確率がわかる)
3. Positive関数の仮定　　成分の一部をそのままではできないが、上手く変換する関数があるとして(カーネル法のように？)、変換した先では完全に分離可能な分布。
4. 非還元可能性　Negativeの分布がPositiveの分布を完全に含むことはない。もし、これが成り立つなら、Positiveを特定したとしてもNegativeのなかにPositiveがあることになり、そもそもの仮定が破綻。

これは[Jessarらの論文](../2019-ECML%20PKDD-Beyond%20the%20Selected%20Completely%20At%20Random%20Assumption%20for%20Learning%20from%20Positive%20and%20Unlabeled%20Data/_index.md)でもあった。

## PU Learningについての評価指標

