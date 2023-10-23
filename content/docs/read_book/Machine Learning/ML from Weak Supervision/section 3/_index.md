---
title: "Section 3"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

第3章　PN Learning

# 二値分類でのPN Learning

## 定式化

PとNの訓練データが同時に取得したか、別々に取得したかで定式化が異なる。

### 同時に取得

$$
[ (\mathbf{x} _i, y _i) ] _{i = 1} ^ N \overset{i.i.d}{\sim} p(\mathbf{x}, y)
$$

このように同時に取得されたものは、分類が容易である。以下の損失関数を最小化すればよい。正則化項を加えるのを忘れずに。

$$
R(g) = \mathbb{E} _{p(\mathbf{x}, y)} [ l(g(\mathbf{x}), y) ] \\\\ 
\hat{g} = \argmin _{g} \hat{R} (g) + \lambda \Omega(g)
$$

どう見てもこれソフトマージンのSVMとほぼ同じじゃないですか。ただ訓練データのpositive vs negativeが1:1に限らないが。

### 別々に取得

$$
[ \mathbf{x} _{P, i} ] _{i = 1} ^ {N _P} \overset{i.i.d}{\sim} p(\mathbf{x} | y = +1) \\\\ 
[ \mathbf{x} _{N, i} ] _{i = 1} ^ {N _N} \overset{i.i.d}{\sim} p(\mathbf{x} | y = -1)
$$

このように別々に、前提条件が異なる分布から取得する。この場合、より柔軟に強力な分類アルゴリズムを開発できる(らしい)

$N _P \neq N _N$もあり得るので、サンプル数が不均等であることもあり得る。損失関数を以下のように定義する。これに正則化項を加えた

$$
R _P(g) = \mathbb{E} _{p(\mathbf{x}, y = +1)} [ l(g(\mathbf{x}), +1) ] \\\\ 
R _N(g) = \mathbb{E} _{p(\mathbf{x}, y = -1)} [ l(g(\mathbf{x}), -1) ] \\\\ 
R(g) = p(y = +1) R _P(g) + p(y = -1) R _N (g)
$$

実を言うと別々に取得の式は同時取得を包含している形ともいえる。

## 理論的解析

分類器$\hat{g}$の訓練がちゃんと最適な解へ収束すれば、理論的にはこの方法はうまく行く。収束先として3つの分類器を考える(当然この3つの理想的な分類器はいずれも最適である)。

1. 真の代理損失関数を使ったリスク関数(不偏推定量でつくるものではなく)を最小化したものを収束目標とする。ただし、$g$は所定のモデル$\mathcal{G}$に所属している前提で。
   1. $g ^ {*} = \argmin _{g \in \mathcal{G}} R(g)$
2. 真の代理損失関数を使ったリスク関数を最小化する。$g$は任意の可測関数。
   1. $g  _R ^ {*} = \argmin _{g} R(g)$
   2. これはベイズ最適化分類器(optimal classifier)の最小化とも呼ばれているらしい。
   3. $R ^ * = \inf _{g} R(g) = R(g _R ^ *)$。真のリスク関数に対しての最小化を行った関数。
      1. これは$R(g)$に対して完全に一致する。
3. もっと割り切るなら、代理損失関数を使わない0-1損失関数自体で定義すればいい。
   1. $g _I ^ {*} = \argmin _{g} \mathbb{E} _{p(\mathbf{x}, y)} [ l _{01} (g(\mathbf{x}, y)) ]$
   2. $I ^ {*} = \inf _{g} I(g) = g _{I} ^ *$
      1. gradientがない最適関数になる。
      2. これも$I(g)$に対して完全に一致する。

正確な厳密さは3 > 2 > 1となる。

理論的には、代理損失関数$l$は$l _{01}$の代替をするにあたって、以下の要件を満たす必要がある。

ある写像$\alpha$を考える。
1. $\alpha : [0, 1] \to [0, +\infty)$
2. $\alpha$は凸の写像で、広義単調増加。
3. $\alpha$は逆写像を持つ。
4. $\alpha(0) = 0$

以上の要件を満たす$\alpha(x)$の例としては、$\tan(\frac{2x}{\pi})$がある。

そして、そのような$\alpha$に対して、以下の条件をみたさなければならない。

$\alpha(I(g) - I ^ {*}) \leq R(g) - R ^ *$

上式の条件を使うと以下のように落ち着く。

$$
0 \leq I(g) - I ^ * \leq \alpha ^ {-1} (R(g) - R ^ *) = \alpha ^ {-1} (0) = 0
$$

両側0で挟めるので、やはり$I ^ *, R ^ *$もそれぞれ$I(g), R(g)$の最小値となる。つまり、理論上の01損失、代理損失関数(ただしαのような都合の良い写像がある場合)のどちらに収束する、としても問題はない。

