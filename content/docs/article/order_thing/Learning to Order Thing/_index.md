---
title: "Learning to Order Thing"
weight: 1
# bookFlatSection: false
# bookToc: true
bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# 問題設定

$a, b\in X$とする。$a, b$に対して、$a < b$($b$は$a$より良い)か$a > b$($a$は$b$より良い)のように、優劣を伝えるクエリがあるとする。

大量にその形のクエリを読み込んで、**全体で$X$の各要素をランキングを作りたい**。

人が判断していると考えるので、$a < b$というクエリと$a > b$というクエリは両方存在しても矛盾とはみなさない。

# 論文情報

| タイトル | Learning to Order Things                                     |
| -------- | ------------------------------------------------------------ |
| 刊行物   | Journal of Artificial Intelligence Research 10(1999) 243-270 |
| 著者     | William W. Cohen, Robert E. Schapire, Yoram Singer           |

# 論文の手法

## 設定

- $X$は比較対象の集合とする。
- pref(a, b)を、$X \times X \rightarrow [0, 1]$の関数とする。
  - 1に近いほど、$a > b$の傾向が強い。
  - 0に近いほど、$b < a$の傾向が強い。
  - 0.5に近ければ、どちらとも言えない。
- pref(a, b)は、本質的にはいくつかの「好み関数」の線形合成である。下図参照。

![](/fig1pref_linear_composing.png)

これは、左と真ん中は、「**ある順序(大概はわからないけど)に従って、作られるpref()**」であり、それらを線形合成して新たなpref()を右図に作っている。

また、明確に順位を持っているときに、pref()を作ること自体は容易である。以下のようにすればいい。

- pref(a, b)で、$a < b$なら、1、$a > b$なら0、$a = b$なら0.5

## 学習のやり方

あらかじめ、各人の意見に重み$w_i$を設定する。

毎回、何人かの人からそれぞれ各々の$X$内でのランキングを作る。そのランキングをもとに、重みつきでpref()を作る。つまり以下の式。

$$
pref(a, b) = \Sigma_{i = 1}^{N} w_i pref_{sub}(a, b)
$$

そして、後述の方法でpref()から、あらたにランキングを生成し、それをuserにfeedback。

そして、今のprefを使い、ロスを以下の式で計算。

$$
ロス = 1 - \frac{1}{|F|} \Sigma_{(a, b) \in F} pref(a, b)
$$

このロスをもって、$w_i$を更新して、一周が終わる。

$$
w_i \leftarrow \frac{w_i \beta ^{ロス}}{Z}
$$

この$\beta$はハイパーパラメタであって、$Z$はweightの和が1となる正規化のために割っている。

## ランキングの推定方法

私が知りたかったやつ。

AGREE()という関数を考える。これは**順位で$a$が$b$より優れる**ときの**pref(a, b)を全て足し合わせたもの**。

**一般論をいうと、厳密なランキングを見つけるのはNP困難**である。

というわけで貪欲法