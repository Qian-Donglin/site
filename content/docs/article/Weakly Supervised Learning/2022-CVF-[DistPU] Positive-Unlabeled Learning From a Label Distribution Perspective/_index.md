---
title: "2022-CVF-[DistPU] Positive Unlabeled Learning From a Label Distribution Perspective"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[元論文 HTML版](https://ar5iv.labs.arxiv.org/html/2212.02801#S3.E8)

[元論文 PDF版](https://openaccess.thecvf.com/content/CVPR2022/papers/Zhao_Dist-PU_Positive-Unlabeled_Learning_From_a_Label_Distribution_Perspective_CVPR_2022_paper.pdf)

## Introduction

cost-sensitiveの手法(2014 du Plessis、2015 du Plessis、2017 Kiryo　など)は、UnlabeledをNegative例のように見なして分類を行った。そこではクラス事前確率を与えてできるだけ補正しているが、どうしても**判定結果は負例に偏ってしまう**。これは**NNのような表現力の高いモデルでなおさら顕著**である。

クラス事前確率を与えられれば、真の分布と予測分布は一致するように学習しなければならない。ズレていう以上、補正を加える必要がある。この論文ではエントロピー最小化とmixup正則化を使っている。

しかし、NNで予測するとき、最後にcalibrationされた確率が出るが、自明に$\pi = p(y = +1)$としてしまえる。これは意味がない(分類できてないよね？)ので、エントロピー最小化を採用して、1か0に近い値を取るようにした。また、過学習防止にmixupを使った。

この論文の貢献は以下の3つ。

1. Dist-PUとよばれる分布の補正手法を開発。これは負例へ偏るのを改善。
2. エントロピー最小化とmixupを組み入れて、自明な解を学習目標にしないようにした。
3. Fashion-MNIST, CIFAR-10, Alzheimerのデータセットでいい成果がでた

## 関連研究

PU Learningはcost-sensitiveとsamplingがある。samplingは、以下の流れである。

1. 信頼度の高いPositive, Negativeの点をUnlabeled Dataから見つける。
2. 自己教師あり学習で得る。

基本的にPU Learningの手法はクラス事前確率が知られていることが前提である。故に、これを推定する手法も多い。

## 手法

### 問題設定

訓練データ$\mathbf{x} \in \mathbb{R} ^ d$として、二値のラベル$y = 0, 1$を持つ。0がNegativeで1がPositive。
Unlabeledのデータは$\mathbf{x} \sim p(\mathbf{x})$から得られ、事前確率$\pi = p(y = +1)$として、以下のように書ける。

$$
p(\mathbf{x}) = \pi p(\mathbf{x} | y = +1) + (1 - \pi) p(\mathbf{x} | y = -1)
$$

識別器$f \in \mathcal{F} : \mathbb{R} ^ d \to \mathbb{R}$を訓練して、以下の予測誤差を最小化するのが目的である。

$$
R = \mathbb{E} _{\mathbf{x}, y \sim p(\mathbf{x}, y)} [l _{01}(f(\mathbf{x}), y)]
$$

誤差は01損失だと予測誤差はすなわち確率であるが、微分ができないので現実的には代理誤差を用いる。

この論文でもそうだが、一般的にはPU LearningのPositiveのデータ選択は一様にランダムに選ばれる。

### 分布の補正

既存のcost-sensitiveの手法は、Unlabeledは何も情報がないので、クラス事前確率を元にpositiveデータとnegativeデータとみなして分割して、そのリスク関数を最小化していた。

ここでの提案手法は下図のようになる。

既存のPU Learningにおけるコストを全部捨てて、Positiveならば確率は`P : N = 1.0 : 0.0`、Unlabeledならば確率は`P : N = π : 1 - π`が正解データであり、こことのL1損失を使用して誤差逆伝搬をしてNNを更新する。

定式化してみる。$\mathbb{E} _{+}$はPositiveの分布における期待値であり、$\mathbb{E} _{-}$はNegativeの分布における期待値とする。$\mathbb{E} _{X}$はUnlabeledの分布における期待値である。最後には、Positiveのリスク$R _P$とNegativeのリスク$R _N$に分割できる。

$$
R(f) = \pi \mathbb{E} _{+} [l _{01}(f(\mathbf{x}), +1)] + (1 - \pi) \mathbb{E} _{-} [l _{01} [f(\mathbf{x}, 0)]] \\\\ 
= \pi \mathbb{E} _{+} [1 - f(\mathbf{x})] +  (1 - \pi) \mathbb{E} _{-} [l _{01} [f(\mathbf{x}, 0)]] \\\\ 
= \pi | \mathbb{E} _{+} [f(\mathbf{x})] - 1 | +  (1 - \pi) \mathbb{E} _{-} [l _{01} [f(\mathbf{x}, 0)]] \\\\ 
= \pi R _P + (1 - \pi) R _N
$$

$f(\mathbf{x})$は$0, 1$の二値を取るので、$R _P = 1 - \mathbb{E} _{+} [f(\mathbf{x})]$と書いても良い。

次に、Unlabeledのデータについて、予測と実際のラベルの間の差を評価する。$\pi$が組成比であることを念頭に考えると、以下の式のように変形できる。
ここでは既存のcost-sensitiveな手法なので、unlabeledのデータには、$f(\mathbf{x}) = 1$だとペナルティが発生する。(これがNegativeに偏るという問題の起因)

$$
\mathbb{E} _{X} [f(\mathbf{x})] - \mathbb{E} _{\mathbf{x}, y \sim p(\mathbf{x}, y)} [y]
=\pi \mathbb{E} _{+} [f(\mathbf{x})] + (1 - \pi) \mathbb{E} _{-} [f(\mathbf{x})] - \pi \\\\ 
=\pi(\mathbb{E} _{+} [f(\mathbf{x})] - 1) + (1 - \pi) \mathbb{E} _{-} [f(\mathbf{x})] \\\\ 
= -\pi R _P + (1 - \pi) R _N
$$

この式から、実際は知ることができないNegativeについてのリスク$(1 - \pi) R _N$を知ることができる。これを代入すると、2014 du Plessisらが導いた慣れ親しんだ式が出てくる。

$$
R(f) = \pi R _P + (1 - \pi) R _N = \pi R _P + + \pi R _P + \mathbb{E} _{X} [f(\mathbf{x})] - \mathbb{E} _{\mathbf{x}, y \sim p(\mathbf{x}, y)} [y] \\\\ 
=2 \pi R _P + \mathbb{E} _{X} [f(\mathbf{x})] - \pi 
$$

QUESTION: 負へ偏るバイアスはなぜある？代理損失だから？

しかし、この式は代理損失、負になって過学習することがあるので、以下のように絶対値で囲うことを提案する。絶対値で囲まれた部分は01損失を使う前提で考えてみる。期待値部分はまさに、正しく学習していれば$\pi$を推定していると言えるので、絶対値に囲まれた部分は本来0となるはずである。

$$
R _{lab} (f) = 2 \pi R _P + |\mathbb{E} _{X} [f(\mathbf{x})] - \pi|
$$

しかし、これではまだ実用的ではない。$f(\mathbf{x})$が$0, 1$の二値しかとらないという離散的な試行は、微分不可能であるので、$f(\mathbf{x})$を離散値を取る関数にして、正ならPositiveっぽい、負ならNegativeっぽいと考える。そして、以下のシグモイド関数を用いて疑似的な確率にcalibrationする。

QUESTION: 疑似的な確率にcalibrationされている以上、これをもってして厳密な確率に合わせたらだめでは？あと識別境界から近い、遠いとかも考えるべきでは？Positiveなら全部1は乱暴じゃね。

$$
p(\hat{y} = +1) = s = \frac{1}{1 + \exp(-f(\mathbf{x}))}
$$

結果的に、こうやって確率にcalibrationしたので、前述の手法である

1. Positiveデータならば、確率1.0との差の絶対値。
2. Unlabeledのデータならば、確率$\pi$との差の絶対値。

を用いて学習した。経験的な式は以下の通り。

$$
\hat{R} _{lab} = 2 \pi |\frac{1}{n _L} \sum _{\mathbf{x} \in pos} s - 1| + |\frac{1}{n _U} \sum _{\mathbf{x} \in unlabel} s - \pi| 
$$

### 上界の評価

VC次元が$V$として、$\delta \in (0, 1)$として、$1 - \delta$以上の確率で以下が成り立つ。$C$は定数(ラデマッハ複雑度由来の定数？)。

$$
R(f) \leq 2 \hat{R} _{lab} + 8 \pi C \pi \sqrt{\frac{V}{n _L}} + 12 \pi \sqrt{\frac{\log (4 / \delta)}{2 n _L}} + 4C \sqrt{\frac{V}{n _U}} + 6 \sqrt{\frac{\log (4 / \delta)}{2 n _U}}
$$

証明はこちら。基本的には同様に一様大数の法則、McDiarmidの不等式で抑える。

### エントロピー最小化

上の学習の評価の欠点として、**自明な解である「どのような入力であろうと、シグモイド関数に$f(\mathbf{x})$を渡したら$\pi$となる」ような収束では意味がない**、ことがある。

これをうけて、**できるだけcalibrationされた確率を0か1に近づけたいという気持ちがある**。これは決定する超平面近くのデータは少なく、同じカテゴリのクラスタがある場所のデータが密であるというクラスタリング仮説に基づいている。
結果的には、Unlabeledのデータに対して**以下の式も最小化する全体の式の一部に加える**。

$$
L _{ent} = -\frac{1}{n _U} \sum _{\mathbf{x} \in X _U} (1 - s) \log (1 - s) + s \log s
$$

### 確認バイアス

エントロピーの最小化も加えることにより確かに確率は0と1の近くに分布されるが、これはモデルの**自信過剰にもつながる**ことを意味している。
自信過剰のこの現象を確認バイアスという。これをmixupというよく使われる技術で本論文は解決を試みた。

基本的には、機械学習はVicinal Risk Minimizationの原則、すなわちデータの近傍ならば元のデータと同じ分類結果になるべきという原則。今回の場合、同じクラ

Mixupは以下のように、パラメタαから生成されるβ分布から得られる値を比として、既存のデータから新たなデータ点を生成するものである。

$$
\lambda \sim \mathrm{Beta} (\alpha, \alpha) \\\\ 
\lambda ^ \prime = \max(\lambda, 1 - \lambda) \\\\ 
\mathbf{x} _{new} = \lambda ^ \prime \mathbf{x} _1 + (1 - \lambda ^ \prime) \mathbf{x} _2
$$

このように生成された点に対しても、損失関数を考えて最適化の対象に加える。$l _{bce}$は二値のクロスエントロピーであり、$l _{bce}(a, b) = -(1 - a) \log(1 - b) - a \log b$である。

$$
L _{mix} = \frac{1}{n} \sum _{\mathbf{x} _1, \mathbf{x} _2 \in X _P \cup X _U} \lambda ^ \prime l _{bce}(\mathrm{sig}(-f(\mathbf{x} _{new})), \mathrm{sig} (-f(\mathbf{x} _1))) + (1 - \lambda ^ \prime ) l _{bce}(\mathrm{sig} (-f(\mathbf{x} _{new})), \mathrm{sig}(-f(\mathbf{x} _2)))
$$

mixupで確認バイアスが改善される理由としては、以下の3点がある。

1. PとU関係なく、mixupで生成された点に対して、元の2点とcalibrationされた確率が近いほどロスが小さくなる。極端な確率の場合は極端な2つからmixupされるとロスが大きくなる。
2. 偽陰性と偽陽性の2点からmixupしたとしても、混合比が1:1ならば真陽性と真陰性の2点のmixupと同じ評価となり、無駄にならない。
3. データ量を増加させて訓練に相当するので、そもそもロバストになりやすい。

更にその上、mixupで合成した各点についても、Unlabeledだとみなして、エントロピーの最小化で確率を0か1へと偏らせることを行う。

$$
s(\mathbf{x}) = \mathrm{Sig}(-f()\mathbf{x}) \\\\ 
L _{ent} ^ \prime = - \frac{1}{n} \sum _{\mathbf{x} _1, \mathbf{x} _2 \in X _P \cup X _U} l _{bce}(s(\mathbf{x}), s(\mathbf{x}))
$$

### 結局損失関数はどういう構成か？

$$
L _{dist} = \hat{R} _{lab} + a L _{ent} + b L _{mix} + c L _{ent} ^ \prime 
$$

$a, b, c$はハイパーパラメタ。含むのは以下の要素。

1. Positiveなら`1.0 : 0.0`、Unlabeledなら`π : 1 - π`が理想のラベルとして扱う。
2. 確率を0と1に偏らせるため、エントロピーの最小化を導入。
3. mixupした点の予測結果も同じようにするとペナルティが小さくなる項を追加。
   1. mixupした点自体もunlabeledとして扱えば、実質的にはデータ点の増加ともいえる。その際、mixupした点に対しても、エントロピー最小化も行うこと。

## 実験

省略。いい結果が出ました。後でちゃんと読みます。

