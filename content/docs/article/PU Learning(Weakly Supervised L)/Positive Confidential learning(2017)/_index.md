---
title: "Positive Confidential Learning(2017)"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[元論文](https://arxiv.org/abs/1710.07138)

日本語解説はどうやらないみたい。

[こっちのGitHubレポジトリ](https://github.com/takashiishida/pconf)に3mの発表動画やスライドがある。テストコードも。

# Introduction

Positiveデータしか持たないが、その中で各データに信頼度というパラメタがある。Unknown、Negativeはない。

1クラス分類はあるが、ハイパーパラメタの調整ができない手法しかなかった。しかも、どんなにデータを増やしても、分布の境界を予測はできない。類似したものを見つけることができても、分類=境界線をいい感じに引く、ことはできない。

このPConfは、ハイパーパラメタを客観的に選ぶこともできる。これは、ERM(経験損失最小化 = **選んだ標本誤差の最小化**)を使って実現されている。

PU学習とも似てる手法ではあるが、PU学習で必要なPositiveのデータの事前確率の推定(2007年のPU学習では、$p(ラベル付き|データ)$を推定していた(結局あんま精度高くないよね)　これを指してるのだと思う)は**不要**。これを高精度で求めるのは苦労がかかるので、朗報。実は**各データについての信頼度という情報には、事前確率、条件付確率、事後確率はすべて含まれている**。

# 問題の定式化

データは$\mathbf{x} \in \mathbb{R}^d$の形。ラベルは二値分類なので、$-1, 1$。これらは、未知の分布関数$p(\mathbf{x}, y)$に従う。

**実現したいのは、$g(\mathbf{}) : \mathbb{R}^d \to \mathbb{R}$という二値分類器**(出力が二値ではないのは、確からしさを残すため？ by me)

例によって、損失がどれぐらい出てくるのかの**リスク関数を定義**する。

$$
R(g) = \mathbb{E} _{p(\mathbf{x}, y)} [l(y g(\mathbf{x}))]
$$

$l(m)$は損失関数。ラベルと予測器が同じ符合なら$l(m)$は0に、違う符合なら関数に渡されるのは負値で、**より自信満々に間違えるほど大きな負値を渡す**ことになる。$l(m)$は大きなマイナスほど大きな値を返す。

例によって、**このリスク関数を最小化**していく。

ここで、$p(\mathbf{x}, y)$は未知なので、普通のERM(経験損失最小化)では、真の期待値を今あるテストデータの期待値で代用する。それはそう。しかし、PConfでは、違う。

PConfでは、与えられたデータ群(と言ってもPositiveしかないが)に対して、**$\chi$という集合で、信頼度を定義する**。

$$
\chi := (\mathbf{x}_i, r_i) で、iは全てのデータ
$$

$r_i$は各データの信頼度

$$
r_i = p(y = +1 | \mathbf{x}_i)
$$

$\mathbf{x}_i$はあるデータを全数つかわない場合、データ群から一様ランダムに抽出したものである。

Negativeデータがないので、普通のERMの計算方法ではリスク関数を計算するできない。次のセクションでは信頼度で代わりに表示する方法を表す。

# PConf分類のやり方

## ERM(経験損失最小化)の枠組み

$\pi _{+} = p(y=+1)$これは全データのうち、Positiveのデータの割合。あれ、Negativeとか見えないんじゃなかったっけ？by me

$r(\mathbf{x}) = p(y=+1 | \mathbf{x})$ $r_i$はデータ$\mathbf{x}_i$についての信頼度だった。これは、**引数に渡したデータ$\mathbf{x}$の信頼度**。

$\mathbb{E} _{+}$は、$\mathbb{E} _{p(\mathbf{x} | y = +1)}$の場合、つまり**Positiveのデータに限った場合の条件付期待値**である。これらを使って、以下のように$R(g)$を表せる。

$$
R(g) = \pi _{+} \mathbb{E} _{+} [ l(g(\mathbf{x})) + \frac{1 - r(\mathbf{x})}{r(\mathbf{x})} l(-g(\mathbf{x}))]
$$

当然だが全部Positiveなので、$r(\mathbf{x}) \neq 0$である。

### 式変形の意味

意味としては、以下の通り。(本来はAppendix Aにある)

$$
R(g) = \sum _{y = +1, -1} l(y g(\mathbf{x})) p(\mathbf{x}, y) d\mathbf{x} = \sum _{y = +1, -1} l(y g(\mathbf{x})) p(\mathbf{x}| y) p(y)d\mathbf{x}
$$

$\pi _{+} = p(y = +1)$、$\pi _{-} = p(y = -1)$は**積分の外に出せる**ので、

$$
=\pi _{+} \mathbb{E} _{p(\mathbf{x} | y = +1)} [l(yg(x))] + \pi _{-} \mathbb{E} _{p(\mathbf{x} | y = -1)} [l(-yg(x))]
$$

第2項では、$y=-1$なので損失関数のなかには負の符号がつく**。ここで、一旦以下の値の変形を見る。

$$
\pi _{+} p(\mathbf{x} | y = +1) + \pi _{-} p(\mathbf{x} | y = -1)
$$

$$
= p(\mathbf{x}, y = +1) + p(\mathbf{x}, y = -1) = p(\mathbf{x})
$$

$$
= \frac{p(\mathbf{x}, y = +1)}{p(p(y = +1 | \mathbf{x}))} = \frac{\pi _{+} p(\mathbf{x} | y = +1)}{r(\mathbf{x})}
$$

ここで、式の両辺に$\pi _{+} p(\mathbf{x} | y = +1)$が出てきてるので、$\pi _{-} p(\mathbf{x} | y = -1)$について解けば、

$$
\pi _{-} p(\mathbf{x} | y = -1) = \pi _{+} p(\mathbf{x} | y = +1) (\frac{1 - r(\mathbf{x})}{r(\mathbf{x})})
$$

とえられる。これを代入すれば元の式が得られる。

### 本筋の続き

$$
R(g) = \pi _{+} \mathbb{E} _{+} [ l(g(\mathbf{x})) + \frac{1 - r(\mathbf{x})}{r(\mathbf{x})} l(-g(\mathbf{x}))]
$$

上のように上手いこと変形することで、Negativeのデータにおける期待値を消去してる。さらに言えばこの$R(g)$の最小化を考えるにあたり、未知の量$\pi _{+}$は比例定数なので考えなくてもいい。

では、この$R(g)$は理想的な分布を使っていたわけだが、これを実際にあるデータの平均で期待値を代用すると、以下のような最適化問題を解く形になる。

$$
\min _{g} \sum _{i = 1} ^ {n} [ l(g(\mathbf{x}_i)) + \frac{r_i}{1 - r_i} l(-g(\mathbf{x}_i)) ]
$$

### 最適化問題だから分母消してもいいだろ←いいの？

これに$r_i$を乗じた以下のもの

$$
\min _{g} \sum _{i = 1} ^ {n} [ r_i l(g(\mathbf{x}_i)) + (1 - r_i) l(-g(\mathbf{x}_i)) ]
$$

この式、更に簡約できる。

$$
= \min _{g} \sum _{i = 1} ^ {n} p(y = +1 | \mathbf{x}_i) l(g(\mathbf{x}_i)) + p(y = -1 | \mathbf{x}_i) l(-g(\mathbf{x}_i))
$$

$$
= \min _{g} \sum _{i = 1} ^ {n} \sum _{y = -1, 1} p(y | \mathbf{x}  _i) l(y g(\mathbf{x} _i))
$$

このように簡約できる。だが、実はこれは**等価ではない**。外側の$\sum$が、$p(\mathbf{x})$に対してだが、これが$p(\mathbf{x} | y = +1)$ならば正しい。なぜなら、$r_i=p(y = +1 | \mathbf{x}_i)$を掛けたので、

$$
p(\mathbf{x}_i | y = +1) = \frac{r_i p(\mathbf{x}_i)}{p(y = +1)}
$$

$$
r_i = \frac{p(\mathbf{x}_i | y = + 1) p(y = +1)}{p(\mathbf{x}_i)}
$$

ここで、$p(y = +1)$は定数？なので、まあ掛けないとあかん理由わかるね。

### 本筋に戻る

[重点サンプリングとは](https://qiita.com/s-yonekura/items/287631cc894e436fd1d4)　以下に簡単に書く。

$$
\mathbb{E} _{\mu} [f(x)] = \int f(x) \mu(x) dx = \int f(x) \frac{\mu(x)}{\nu(x)} \nu(x) dx
$$

$w(x) = \frac{\mu(x)}{\nu(x)}$と言い換えておくと、この上の式は

$$
\int f(x) w(x) \mu(x) dx = \mathbb{E} _{\nu} [ f(x)w(x) ]
$$

と言い換えられる。つまり、重み関数$w(x)$を掛ければ、別の分布についての期待値となる。この手法を重点サンプリング=Importance Samplingという。

上の2つの式は、互いに重点サンプリングである、とも見れる。$p(\mathbf{x})$と$p(\mathbf{x} | y = +1)$についての期待値。

## 性能評価

長すぎて略したい...むずかしいし。

結論として、

$$
\min _{g} \sum _{i = 1} ^ {n} [ l(g(\mathbf{x}_i)) + \frac{r_i}{1 - r_i} l(-g(\mathbf{x}_i)) ]
$$

は真のリスク関数に収束するが、

$$
\min _{g} \sum _{i = 1} ^ {n} \sum _{y = -1, 1} p(y | \mathbf{x}  _i) l(y g(\mathbf{x} _i))
$$

は真のリスク関数には収束しない。

## 実装するには

$$
g(\mathbf{x}) = \mathbf{w} ^ T \mathbf{\Phi(\mathbf{x})}
$$

こんな風に線形識別器$g(\mathbf{x})$を作るとする。

L2正規化をしたERMは、以下の最小化を行う。$\lambda$は正規化定数であり、**$R$は半正定値行列**。

$$
\min _{\mathbf{a}} \frac{\lambda}{2} \mathbf{w} ^ T R \mathbf{w} + \sum _{i = 1} ^ {n} [ l( \mathbf{w}^T \mathbf{\Phi(\mathbf{x}_i)}) + \frac{r_i}{1 - r_i} l(-\mathbf{w}^T \mathbf{\Phi(\mathbf{x}_i)}) ]
$$

正規化項はいつもの$|| \mathbf{w} ||^2$ではないが、これは$R$が半正定値行列なので、コレスキー分解$R = C^T C$とすることができる。こうすると、$\mathbf{w} ^ T R \mathbf{w} = ((C\mathbf{w}) ^T (C\mathbf{w}))$と、別の線形空間に写像したうえでのL2正規化と見れる。

損失関数は

- $l(z) = (z - 1)^2$
- $l(z) = \max(0, 1 - z)$
- $l(z) = \log(1 + e^{-z})$ これが**一番よかった**。

これは確率的勾配法とかを使えば最適化できる。

これ、**SVMで行けるんじゃね？？？？？**by me(たぶんやってる)

# 実験

**どうやら線形モデルでも、DNNでも有効らしい。DNNで更に有効らしい**。なんだこれは。

## 線形モデルの合成実験

平均が$\mathbf{\mu} _{+}, \mathbf{\mu} _{-}$であり、共分散行列が$\Sigma _{+} \Sigma _{-}$であるデータを使う。

- 提案した手法
- 提案した手法に$r_i$掛けて、違うよね？とした手法
- 回帰ベースの手法。信頼度事態を予測して、0.5以上か以下で判断する
- 1クラスSVMはGaussianカーネルを使う。性質上、正例データのみ使う。
- 完全にラベル付けされた例も、比較実験として使う。

提案手法、違うよね？手法、完全にラベル付けした手法は、$g(\mathbf{x}) = \mathbf{w} ^ T \mathbf{x} + b$を使ったらしい。最適化ではエポック数5000、学習率0.001で行った。

なお、$r(\mathbf{x})$はどう計算されるんだ？に関しては「**2つのガウス密度から解析的に計算される**」そう。なにこれ？？？**何も言ってないけど、2つのクラスの真の中心の距離の逆比っすかね**？？？まあ、$r(\mathbf{x}) = p(y = +1 | \mathbf{x})$なので、$p(\mathbf{x} | y = +1)$からベイズの定理で求めるのかね？

結果としては、**提案手法のPConfは、完全にラベル判明したものと同等の精度があると判明**。

なお、**真の確信度わからずにガウス分布に従うノイズを加えると、性能が低下することにはするが、全然使える範囲**でもあると。

## DNNにおいての手法適用

[Fashion-MNIST](https://github.com/zalandoresearch/fashion-mnist)という服関連のデータセットでやった。1つのクラス(T-shirts Top)だけをPositive、他をNegativeにした。データは以下の4つに分ける。

- 訓練
- 検証
- テスト
- Positiveのデータに対して、$r(\mathbf{x}_i)$を推定するための確率的分類器の学習用データ
  - 本来は人がやるけど、ここはこれで代用や！
  - ロジスティック回帰で実装した。
  - **ここらの前提はまだそれなりにある　把握しきれない**。

3つの隠れ層でそれぞれ100個のニューロンがあり、最後に1つで総結合するニューロンを持つネットワークで、活性化関数はReLU関数を使った。

あといろいろある。CIFAR-10についてもやってる。

### 結果

提案したPconfはやはり素晴らしい性能だった。完全にラベル付けしたものの性能にも迫るぐらいね。

ほんへ終了