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

