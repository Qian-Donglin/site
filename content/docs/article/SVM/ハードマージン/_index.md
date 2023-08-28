---
title: "ハードマージン"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# ハードマージン

[このQiitaわかりやすい](https://qiita.com/sakigakeman/items/8e1566ebe5f07ca7954b)。

[こっちも補足してみてる](https://qiita.com/oirom/items/5094470b979cbdca0396)。

最適化問題に落としこむ部分では[こっちが一番わかりやすい](https://pynote.hatenablog.com/entry/svm-hardmargin)かな。

この記事の冒頭のリンクでは、SVMが識別器$f(X)$だとして、損失関数はConvexだとうまくいかないと言ってた。

SVMでは、ハードマージン(**完璧に2つのグループに分類できる　具体的には2つのグループの最近点=サポートベクトル、はいずれも正しく分類されてる前提**)をまず考える。この時、ヘッセの公式で、超平面$\mathbf{w}^T \mathbf{x} + \mathbf{\theta} = \mathbf{0}$との距離は、各$\mathbf{x}_i$について

$$
\frac{|\mathbf{w}^T \mathbf{x}_i + \mathbf{\theta|}}{|| \mathbf{w} ||}
$$

となる。この時、SVMが目的は$\mathbf{w, \theta}$を動かして、

$$
d_{max} = \max_{\mathbf{w, \theta}} [\min_{i = 1, \cdots, n} \frac{|\mathbf{w}^T \mathbf{x}_i + \mathbf{\theta}|}{|| \mathbf{w} ||} ]
$$

$$
= \max_{\mathbf{w, \theta}} [\frac{1}{|| \mathbf{w} ||} \min_{i = 1, \cdots, n} |\mathbf{w}^T \mathbf{x}_i + \mathbf{\theta}| ]
$$

を得る事。ただ、超平面はスケール倍しても変わらないため、$\min_{i = 1, \cdots, n} |\mathbf{w}^T \mathbf{x}_i + \mathbf{\theta}| = 1$となるように、$\mathbf{w, \theta}$を正規化する(こういう条件を付けられる)。**この条件下で、$\frac{1}{|| \mathbf{w} ||}$を最大化することになる**。

また、ハードマージンの原理上、全ての点について正しく分類させないといけないので、各点$\mathbf{x}_i$について、ラベル$y_i \in -1, 1$について、以下が成り立つべし。
- $y_i = 1$なら、$\mathbf{w}^T \mathbf{x_i} + \mathbf{\theta} \geq 0$
- $y_i = -1$なら、$\mathbf{w}^T \mathbf{x_i} + \mathbf{\theta} < 0$

これを改めて清書すると、ハードマージンの最適化問題は、($||\mathbf{w}||$は逆数を取った)

$$
\min_{\mathbf{w, \theta}} || \mathbf{w} ||
$$

$$
条件：y_i(\mathbf{w}^T \mathbf{x_i} + \mathbf{\theta})  \geq 0
$$

これを解くには、ラグランジュの未定乗数法を使う。[KKT条件](https://laid-back-scientist.com/lagrange-multiplier2)という形式の問題なので、それをやってみると、以下の関数の

$$
\hat{\mathbf{\alpha}} = \argmax_{\mathbf{\alpha}} [\sum _{i = 1} ^ {n} - \frac{1}{2} \sum _{i = 1}^{n} \sum _{j = 1}^{n} \alpha_i \alpha_j y^{(i)} y^{(j)} \mathbf{x}^{(i)T} \mathbf{x}^{(j)}]
$$

$$
条件: \alpha_i \geq 0, \sum _{i = 1} ^ {n} \alpha_i y^{(j)} = 0
$$

[このサイトでは式変形とかが詳しい](https://laid-back-scientist.com/svm-theory)。

これ自体は勾配法で最適解を求めることになる。

```python
import matplotlib.pyplot as plt
import numpy as np
from sklearn import svm
from sklearn.datasets import make_blobs
from sklearn.preprocessing import minmax_scale

# データセットを作成する。
X, y = make_blobs(n_samples=40, centers=2, random_state=6)

# データセットを描画する。
fig, ax = plt.subplots(facecolor="w")
ax.scatter(X[:, 0], X[:, 1], c=y, s=20, cmap="Paired")

# 学習する。
clf = svm.LinearSVC(C=1000)
clf.fit(X, y)

# 分類平面及びマージンを描画する。
XX, YY = np.meshgrid(np.linspace(*ax.get_xlim(), 100), np.linspace(*ax.get_ylim(), 100))
xy = np.column_stack([XX.ravel(), YY.ravel()])
Z = clf.decision_function(xy).reshape(XX.shape)
ax.contour(
    XX, YY, Z, colors="k", levels=[-1, 0, 1], alpha=0.5, linestyles=["--", "-", "--"]
)

plt.show()
```