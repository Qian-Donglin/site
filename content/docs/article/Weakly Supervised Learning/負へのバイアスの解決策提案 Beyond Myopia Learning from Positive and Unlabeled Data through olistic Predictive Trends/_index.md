---
title: "負へのバイアスの解決策提案 Beyond Myopia Learning from Positive and Unlabeled Data Through Holistic Predictive Trends"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

Beyond Myopia: Learning from Positive and Unlabeled data through predictive Trends

[発表スライド](https://neurips.cc/media/neurips-2023/Slides/71684.pdf)

[論文リンク](https://arxiv.org/abs/2310.04078)

解説スライドメインに読み適宜論文から補足していく。

# 手法

Positive Negative Learningは損失関数としては以下のように定義。

$$
\pi = p(y = +1) \\\\ 
\hat{R} _{PN}(g) = \pi R _{P} ^ + (g) + (1 - \pi) \hat{R} _{N} ^ - (g)
$$

- $R _{P} ^ + (g) = \frac{1}{n _{+}} \sum _{\mathbf{x} _i \in X _+ } l(g(\mathbf{x} _i), +1)$ positiveの分類誤差の不偏推定量。
- $R _{P} ^ + (g) = \frac{1}{n _{-}} \sum _{\mathbf{x} _i \in X _- } l(g(\mathbf{x} _i), -1)$ negativeの分類誤差の不偏推定量。

DuPlessisら？が考えたUnlabeledは、$\pi$だけの割合のpositiveが混ざり、$1 - \pi$だけの割合のnegativeが混ざるというもの。