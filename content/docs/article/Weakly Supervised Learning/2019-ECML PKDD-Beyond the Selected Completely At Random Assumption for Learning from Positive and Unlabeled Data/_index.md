---
title: "2019-ECML PKDD-Beyond the Selected Completely at Random Assumption for Learning From Positive and Unlabeled Data"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[元論文](https://arxiv.org/abs/1809.03207)

## Introduction

[Elkan Notoら](../PU%20Learning/_index.md)が定式化したsingle-training-setでは、明確に$p(s = 1)$がわかるものだった。そしてここではSCAR仮定を使っている=完全にランダムに選択する。この仮定の下では、$p(s = +1 | \mathbf{x}, y = +1) = p(s = +1 | y)$。しかし現実ではそうはうまく行かない。

この論文では、SCARではなく、**より弱いSAR、Selected At Random**という仮説を提言している。データ$\mathbf{x}$の**一部の成分**を引数に受け取って計算される関数によって、**選ばれる確率**が求まる。
因果推論(casual inference literature)で使われる、傾向スコア(propensity score)をPU Learningにも持ち込む。

なお、Preliminariesでは、[サーベイ論文における4つの仮定](../0-survey-2020/_index.md#クラス事前確率における仮定)の参考文献をそれぞれ引用している。

## PU Learningにおけるラベル付けのメカニズム

Elkan, Notoらの論文にもあるように、PU LearningのSCARの着想は、データが欠損しているときの条件から来ていた。これを援用して定義してみる。

- SCAR　Positive例ならば一定確率で選ぶ。
- SAR　Selected At Random. データ$\mathbf{x}$を引数に取る関数があり、その関数の出力値がラベル付けされる確率。
- SNAR　Selected Not Ar Random. 上の2ケース以外。さすがに一概に論じることができない。

## SARでのPU Learningのアルゴリズム

傾向スコアのやり方を参考にしている。

### 傾向スコアとは何か

共変量とは、因と果の両方に依存されてる変量のこと。これらを想定しないと偽相関を見抜けない。グラフィカルモデルでいうと以下のようになる。

```
共変量 ----> 因
|         /
|       /
|    /
|  /
v v
果
```

比較する2つグループで、**そもそも共変量が絡んで、キレイにバイアスなしの状態に分けられていないことが多い**。(例として、高血圧の人が多く投薬群、正常な人の多くがプラセボ群に入ってしまうとか)。そもそも全てのステータスが完全に同じ2人を見つけるなど理論上はあり得ない。

そこで、**人$i$が介入される(上で言う投薬群)確率を傾向スコアとして定義する**。そして、傾向スコアを参考にして2つの群に割り振りをすればいい。割り振り方としては近いペアをランダムに入れるetcなどがある。具体的な定義は以下の通り。

$Z _i$を1ならば介入あり、0なら介入なしとして、以下のものが傾向スコア。つまり、共変量を条件とした$Z _i = 1$の条件付確率である。

$$
e(\mathbf{x} _i) = p(Z _i = 1 | \mathbf{x} _i)
$$

具体的には、いかにして$\mathbf{x} _i$の計算をするかは決まっていない。簡単なものは対次元ロジスティック回帰、もっと表現力を上げるならばNNなど。

この論文で使われるのは、**IPW(Inverse-Propensity-Weighting)という割り当て手法**。
考え方としては、群への割り当て自体は適当でいい。しかし、これのバイアスをうまく除去したい。そこで、該当の全てに対して、**できるだけ全てが平等に現れてほしい**。そのためには**出現頻度が高いものの重みを小さく、低いものの重みを大きくしたい**。これにより、「**ランダムに選んでいるように見せかける**」ことができ、そこから$\mathbb{E} [A _{処置あり}] - \mathbb{E} [A _{処置なし}]$の値を計算できる。前半では$Z _i = 1$、後半では$Z _i = 0$だけ集めており、前半では$e(\mathbf{x} _i)$の逆数だが、後半の群は介入なしなので、$1 - e(\mathbf{x} _i)$の逆数を使う。

$$
\frac{1}{N} \sum _{i = 1} ^ N \frac{1}{e (\mathbf{x} _i)} (Z _i) Y _i - \frac{1} {N} \sum _{i = 1} ^ N \frac{1}{1 - e(\mathbf{x} _i)} (1 - Z _i) Y _i \\\\ 
\frac{\sum _{i = 1} ^ N \frac{1}{e(\mathbf{x} _i)} (Z _i) Y _i}{\sum _{i = 1} ^ N \frac{1}{e(\mathbf{x} _i)} (Z _i)} - \frac{\sum _{i = 1} ^ N \frac{1}{1 - e(\mathbf{x} _i)} (1 - Z _i) Y _i}{\sum _{i = 1} ^ N \frac{1}{1 - e(\mathbf{x} _i)} (1 - Z _i)}
$$

2つの式があるが、**下の方がより分散が小さくなる**。また、**いずれの式でも$N \to \infty$とすれば、真の値へと収束することが保証されている**。

### この論文での応用

$e(\mathbf{x}) = p(s = +1 | \mathbf{x}, y = +1)$

とすると、共変量$\mathbf{x}, y$のもとでのラベルが付く確率を傾向スコアとみなせる。

先ほどのIPWをそのまま使えるならば、一番幸せであるが、ここでは**Negative例が1つのもないため、$1 - e(\mathbf{x} _i) = p(s = 0 | y, \mathbf{x})$の$y$の情報がないためそのままでは使えない**。

各例の傾向スコアが$e(\mathbf{x} _i)$とすると、ラベル付き例を$\frac{1}{e(\mathbf{x} _i)}$の重みのPositive例と、$\frac{1}{e(\mathbf{x} _i)} - 1$のNegative例として扱う。(具体的には下の例を参照　ようわからん)

真の傾向スコアを知っている、知らないで2通りの設定を考える。

### 真の傾向スコアを知っている

どれほどずれているのかの評価を次のように行える。(ここの経験的な式への変形あってる？)

$$
\mathbb{E} _{\mathbf{x}} [l(y, \hat{y})]
\frac{1}{n} \sum _{i = 1} ^ n l(y _i, \hat{y _i})
$$

損失関数としては、1乗誤差、2乗誤差、対数誤差$l(y=1, \hat{y}) = - \log \hat{y}, l(y=1, \hat{y}) = - \log (1 - \hat{y})$がある。

そして、$s _i = e _i y _i$という等式が成り立つ。これはなぜ成り立つか。以下のようなものである。

- $s _i, y _i$は具体的な値であるので、**経験分布であると考える**。両辺を$n$で割る。$s _i / n = e _i y _i / n$
  - 経験分布とは、具体的な試行から得られる分布。もし試行で現れてない例では0、現れてるものは$n$回の試行では$1 / n$という確率となる=**最尤推定**。
- $s _i$は$\hat{p}(\mathbf{x} _i, s _i = +1)$というかたちで経験分布に現れる。(分布から確率を計算する時は具体的な値を引数に渡すので)
  - $s _i / n$に関して、$s _i = 0, 1$である。今回は$s _i = 1$という条件で考えるが、$s _i = 1$が成り立つ時は$1 / n$、$s _i = 0$が成り立つときは$0 / n = 0$である。よって、$s _i / n = \hat{p}(\mathbf{x} _i, s _i = +1)$
  - このように、**本来は(経験)分布の中に、変数に入れることができる。このように出すことで、該当する経験分布に書き換えることができ、そこから経験じゃない分布(後述)にしてベイズの定理などを適用できる**ので、非常に有用なテク。 
- $s _i$の時と同様に、$y _i$についても、$\hat{p}(\mathbf{x} _i, y _i = +1)$という経験分布について$y _i = +1$であれば$1/n$、そうでなければ$0/n$により、同様に$y _i / n = \hat{p}(\mathbf{x} _i, y _i)$と、分布に書き直せる。
- 書き直したのちは、以下のように**経験分布の$\hat{p}$は試行を無限回にすることで、$p$に近づけられる**。これを利用して式変形する。
$$
s _i / n = e _i y _i / n \Leftrightarrow \hat{p}(\mathbf{x} _i, s _i = +1) = e _i \hat{p}(\mathbf{x} _i, y _i = +1) \\\\ 
= \hat{p}(\mathbf{x} _i, s _i = +1) = p(s = +1 | \mathbf{x}, y = +1) \hat{p}(\mathbf{x} _i, y _i = +1) \\\\ 
\Leftrightarrow p(\mathbf{x}, s = +1) = p(s = +1 | \mathbf{x}, y = +1) p(\mathbf{x}, y = +1) \\\\ 
\Leftrightarrow p(\mathbf{x}, s = +1) = p(s = +1, \mathbf{x}, y = +1)
$$

なお、SCARの仮定では$\forall e _i = const$であるので、下式の最小化の式変形は不要である。[2014年のDu Plessisら](../分布仮定不要PU%20Learning(2014)/_index.md)はSCARという仮定を前提としてここを飛ばして、cost-sensitiveの式変形へとつないだ。

それを踏まえると、**$s _i = e _i y _i$としても、正しい答えへと収束する**とわかる。これを代入すると、最終的に最小化をしたいのは、以下の式。

$$
\mathbb{E} [ \hat{R}(\hat{\mathbf{y}} | \hat{\mathbf{e}}, \mathbf{s}) ] = \frac{1}{n} \sum _{i = 1} ^ n l(y _i, \hat{y _i}) \\\\ 
= \frac{1}{n} \sum _{i = 1} ^ n y _i l _{y = 1} (\hat{y _i}) + (1 - y _i) l _{y = 0} (\hat{y _i}) \\\\ 
= \frac{1}{n} \sum _{i = 1} ^ n e(\mathbf{x} _i) y _i (\frac{1}{e(\mathbf{x} _i)} l _{y = 1}(\hat{y _i}) + (1 - \frac{1}{e(\mathbf{x} _i)})l _{y = 0}(\hat{y _i})) + (1 - e(\mathbf{x} _i) y _i) l _{y = 0}(\hat{y _i}) \\\\ 
= \frac{1}{n} \sum _{i = 1} ^ n s _i (\frac{1}{e(\mathbf{x} _i)} l _{y = 1}(\hat{y _i}) + (1 - \frac{1}{e(\mathbf{x} _i)})l _{y = 0}(\hat{y _i})) + (1 - s _i) l _{y = 0}(\hat{y _i})
$$

この式では

- $y _i$と$1 - y _i$のPN Learningの式から、$s _i$だけを使った式にしたい。
  - **モチベとして$s _i$と$1 - s _i$のかたちにしてみるとどうなるか**？($s _i$しか知らないので、$y _i$のようなかたちにしたい。)
  - 既存のdu PlessisらはSCARなのでこれを行わず、$p(y = +1)$を使うかたちで変形をし、この手法は行わなかった。
  - SCARであれば、$e _i$は定数であるので$s _i$と$y _i$を簡単に定数倍でよかった。
- Unlabeledの$\mathbf{x} _i$では、$y = 0$とみなした時の損失で評価する(**cost-sensitive特有のnegativeに偏る問題はここでもある**)
- 式変形の結果論として以下のことが言える。
  - Labeled(Positiveの一部)の$\mathbf{x} _i$では、重みが$1 / e _i$のPositiveとしての損失と、$(1 / e _i) - 1$のNegativeとしての損失を持つ。
  - Unlabeledの$\mathbf{x}$では、重みが$1 - y _i e _i$のNegativeとしてとして扱う。

### 上界評価

McDiarmidの不等式で評価していた。あとでかく。

### 傾向スコアが未知の場合

傾向スコアの推定を行うしかない。推定したのが$\hat{e _i}$としたとき、真のリスク関数との差の上界？は以下のようになる。

$$
R(\hat{\mathbf{y}}) - \mathbb{E} [ \hat{R} (\hat{\mathbf{y}} | \hat{\mathbf{e}}, \mathbf{s}) ] \\\\ 
=\frac{1}{n} \sum _{i = 1} ^ n y _i (1 - \frac{e _i}{\hat{e _i}}) (l _{y = 1}(\hat{y} _i) - l _{y = 0}(\hat{y} _i))
$$

ここからわかることとして、

- $\hat{e _i}, e _i$が完全に一致すれば0になる。
- $l _{y = 1}(\hat{y} _i) = l _{y = 0}(\hat{y} _i)$でも成り立つが、これは非現実的では？
- Positiveの傾向スコアの**正確性のみ**が影響する。
- 傾向スコアが間違っているとき、予測したクラス$\hat{y _i}$が0か1かに近い、自信満々であるほど大きく響く。
- 傾向スコアが小さいとき、マイナスに式は行くし絶対値は大きくなる。

## Learning under the SAR Assumption

予測するときに、任意のデータが無規則にpropensityスコアを取るのは本質的には予測できない。依って、追加の仮定が必要であり、$\mathbf{x}$のうちの一部の成分によるベクトル$\mathbf{x} _e$を選ぶ。

$$
p(s = +1 | y = +1, \mathbf{x}) = p(s = +1 | y = +1, \mathbf{x} _e), e(\mathbf{x}) = e(\mathbf{x} _e)
$$

### SARをSCARに落とし込むには

複数の層にデータが分けられ、その層の中では各データはSCARであるとするならば、既存の手法で学習することはできる。しかし、これは連続値が$\mathbf{x} _e$に入る場合は使えないし、離散値だとしても指数関数的に層が増えてしまうので非現実的ではある。$\mathbf{x} _e$の次元は低い方が望ましい。

### EMアルゴリズムによる推定

データ$\mathbf{x} _i$、ラベルがついている$s _i$、$\hat{f}$を分類器、$\hat{e}$を傾向スコアの出力する関数モデルとする。

$$
y _i = p(y _i = +1 | \mathbf{x} _i, s _i, \hat{f}, \hat{e}) = s _i + (1 - s _i)\frac{\hat{f}(\mathbf{x} _i)(1 - \hat{e}(\mathbf{x} _i))}{1 - \hat{f}(\mathbf{x} _i) \hat{e}(\mathbf{x} _i)}
$$

- ラベル付きの時は$s _i = +1$であるため、必ずpositiveである。
- ラベルなしの場合、もし傾向スコアが0(Negative)つまり$1 = p(s = +0 | \mathbf{x})$の時は、$\hat{f}(\mathbf{x} _i)$と予測器の出力を出す。
- ラベルなしの場合、もし傾向スコアが1(Positive)つまり$1 = p(s = +1 | \mathbf{x})$の時は、これは0になる。ラベルがついてないのにPositiveであるということは、圧倒的な自信でNegativeになる。
- ラベルなしの場合、もし傾向スコアが0.5(ちょうど真ん中)