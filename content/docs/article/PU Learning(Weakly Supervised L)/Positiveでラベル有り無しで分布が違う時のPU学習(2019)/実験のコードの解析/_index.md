---
title: "バイアスつきPU Learningクラスの実装"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

[こちら](https://github.com/MasaKat0/PUlearning/tree/master/BiasedPUlearning/PUSB)のGitHubのレポジトリのコードについて読み込んでみた。

# [pusb_liner_kernel.py]()

## import部分

```python
import numpy as np
from scipy import optimize

import chainer
from chainer import cuda, Function, gradient_check, Variable
from chainer import optimizers, serializers, utils
from chainer import Link, Chain, ChainList
import chainer.functions as F
import chainer.links as L
```

numpy、scipyの最適化を使う。chainerはDNNの訓練と評価を行うための深層学習フレームワーク。汎用的なDNN作成を支援してるっぽい。

## `PU#__init__()`

```python
def __init__(self, pi):
    self.pi = pi
    self.loss_func = lambda g: self.loss(g)
```

あらかじめ$\pi = p(y = +1)$だけ与えておく。

識別器$g(\mathbf{x})$を受け取って損失を計算するように、`loss_func()`を定義。もっとも`loss()`も[定義されてる](#puloss)ものだがそれを使う。

## `PU#loss()`

```python
def loss(self, g):
    g = np.log(1+np.exp(-g))
    return g
```

損失関数を定義。意味してるのは、

$$
\log (1 + e^{g})
$$

うまいことなめらかなReLU関数みたいに。論文中では

$$
l(f(\mathbf{x}), y = +1) = -\log g(\mathbf{x})
$$

と書いてあったけど真数マイナスにならんの？と思ったが、どうやら$f(\mathbf{x})$は$[a, 1 - a], a \in (0, 1/2)$の定義である。識別器、0か1かを出すの......?謎。0がNegativeで1がPositiveなの？

と思ったやん。これ最後に$- \theta_{\pi}$をかけて、マイナスならNegative、プラスならPositiveをするみたい。

## `PU#pu()`

```python
def pu(self, x, b, t, reg):
    xp = x[t == 1]
    xu = x[t == 0]
    n1 = len(xp)
    n0 = len(xu)
    gp = np.dot(xp, b)
    gu = np.dot(xu, b)
    loss_u = self.loss_func(-gu)
    J1 = -(self.pi/n1)*np.sum(gp)
    J0 = (1/n0)*np.sum(loss_u)
    J = J1+J0+reg*np.dot(b,b)
    return J
```

- `x`　与えられるDataFrameの訓練データ。`t`が1ならPositive、0ならUnlabeled。
- `b`　は与えられる引数。puを最小化するのに最終的に最適な`b`がゆくゆくは収束してほしい([別のメソッドでやる]())。下の式で言うと$g$を$\mathbf{x}^T \mathbf{b}$で実現させてるみたい？
  - つまり、ここでは$g$ははっきりとSVMであると言える。と思う。
- `t`　はTarget。つまり各データに対して、0か1か。
- `reg`　は正則化定数。ここではL2正則化をかけている。

`x[t == 1]`は、1である場合がTrueでそれ以外がFalse。それをxに噛ませて、該当のindexのところだけ抽出してる感じ。

これはPU Learning(オリジナル)の損失関数

$$
R(g) = \pi \mathbb{E} _{p(\mathbf{x} | y = +1)} [l(g(X), y = +1) - l(g(X), y = -1)] + \mathbb{E} _{p(\mathbf{x})} [ l(g(X), y = -1) ]
$$

について計算しているようだ。論文で言うと(5)。

### `J1`

J1の項は$p(\mathbf{x} | y = +1)$についての期待値のもの、すなわち

$$
\pi \mathbb{E} _{p(\mathbf{x} | y = +1)} [l(g(X), y = +1) - l(g(X), y = -1)]
$$

を求めているようだ。実際は不偏推定量で計算しているから、すべての`xp`のデータについて、理想の`b`の係数ベクトルの積？の和を個数で割っている感じ。どうやら具体的な損失関数は別でやってるっぽい？

### `J0`

J0の項は$p(\mathbf{x})$についての期待値のもの。

どうやらUnlabeledのデータになんか$\mathbf{b}$で内積を取って、それをなぜか先ほど定義した`\log (1 + e^{g})`に入れてReLUから平滑化？の変換を施してからJ0を足している。謎。

## `PU#prob()`

```python
def prob(self, x, b):
    x = self.x
    g = np.dot(x, b)
    prob = 1/(1+np.exp(-g))
    return prob
```

**どこでも使われてなかったもの**。

データの$\mathbf{x}$と識別器のパラメタ行列の$\mathbf{b}$を受け取ったら、積を取るともっともらしさみたいなのが出るから、これをシグモイド関数に入れて確率に変換している。

## `PU#optimize()`

```python
def optimize(self, x, t, x_test):
    x_train, x_test, lda_chosen = self.kernel_cv(x, t, x_test)
    res = self.minimize(x_train, t, lda_chosen)
    return res, x_test
```

- `x`　訓練データ。[kernel_cv]()でテストデータに分割してもらう。
- `t`　訓練データに対してラベルデータ。0 ro 1
- `x_test`　テストのデータ。この関数では何もしない(kernel_cvで分割の参考にさせてるだけ)

`x_train, x_test`はkernel_cvで分割してもらってる。

`lda_chosen`は正規化項の係数。これもkernel_cvで求まるらしい。

中身はラッパーで、[self.minimize](#puminimize)をやっているだけ。

## `PU#minimize()`

```python
def minimize(self, x, t, reg):
    b0 = np.zeros(x.shape[1])
    func = lambda b: self.pu(x, b, t, reg)
    grad = lambda b: self.gradient(x, b, t, reg)
    self.result = optimize.minimize(func, b0, jac=grad, method="BFGS")
    self.result = self.result.x
    return self.result
```

- `x`　訓練データ。[kernel_cv](TODO)でテストデータに分割してもらう。
- `t`　訓練データに対してラベルデータ。0 ro 1

初期解は0埋めしている。funcはscipyの最適化のtargetの関数にしている、ただラップしてるだけ。

gradient定義してる関数を参照。これをラップしたものをgradient関数にしてるらしい。自動微分させない感じ？

`scipyのoptimize.minimize`では、ヤコビアンを。BFGSは典型的な方法。

最後にresultの計算して得た最適な係数たちを返して終わり。

## `PU#gradient()`

```python
def gradient(self, x, b, t, reg):
    xp = x[t == 1]
    xu = x[t == 0]
    n1 = len(xp)
    n0 = len(xu)s
    g = np.dot(xu, b)
    z = 1/(1+np.exp(-g))
    dg = np.sum(xp, axis=0)/n1
    grad = -self.pi*dg + np.dot(z.T, xu)/n0 + reg * b
    return grad
```

これは[pu()](#pupu)と同様に最初はPositive=1とUnlaveled=0に分割している。

まあ**やってることは`pu()`の微分**だと思うけど。元々の式は以下のものに正則化項を入れていた。

$$
R(g) = \pi \mathbb{E} _{p(\mathbf{x} | y = +1)} [l(g(X), y = +1) - l(g(X), y = -1)] + \mathbb{E} _{p(\mathbf{x})} [ l(g(X), y = -1) ]
$$

これの式は、不偏推定量で記述した上の形になれば、

$$
-\pi \frac{1}{n_1} \sum _{i = 1}^{n_1} \mathbf{x} _{P,i} ^ T \mathbf{b} + \frac{1}{n_0} \sum _{j = 1}^{n_0} \mathbf{x} _{U, j} ^ T \mathbf{b} + r \mathbf{b} ^ T \mathbf{b}
$$

これはどうやら$f(\mathbf{x}) = \mathbf{x} ^ T \mathbf{b}$だとしたときみたい。$\mathbf{b}$で微分するので、

$$
-\pi \frac{1}{n_1} \sum _{i = 1}^{n_1} \mathbf{x} _{P,i} + \frac{1}{n_0} \sum _{j = 1}^{n_0} \mathbf{x} _{U, j} + 2r \mathbf{b}
$$

**なんか、ソースコードでは正規化項に2倍かかってないんだよな？間違えてやってないかね**？

なお、実際のところ、$\log (1 + e^{x})$の変換してるところはそこは配慮せなあかん。それが上のコード。

## `PU#test()`

```python
def test(self, x, b, t, quant=True, pi=False):
    theta = 0
    f = np.dot(x, b)
    if quant is True:
    temp = np.copy(f)
    temp = np.sort(temp)
    theta = temp[np.int(np.floor(len(x)*(1-pi)))]
    pred = np.zeros(len(x))
    pred[f > theta] = 1
    acc = np.mean(pred == t)
    return acc
```

与えられた$\mathbf{x}$と、識別器の係数ベクトル$\mathbf{b}$について演算して、実際に検証する。

`quant is True`ならば、論文での

$$
n^{test} \pi = \sum _{i = 1}^{n^{test}} 1(\mathrm{if} \hat{r}(\mathbf{x} _i) \geq \hat{\theta _{\pi}})
$$

をもとに、$\theta$を逆算している。

最後の比較はバイアスの$\theta$を引いてプラスならPositive、それ以外ならNegativeで。

## `PU#dist()`

```python
def dist(self, x, T=None, num_basis=False):
    (d,n) = x.shape
    # check input argument

    # set the kernel bases

    if num_basis is False:
        num_basis = 300

    idx = np.random.permutation(n)[0:num_basis]
    C = x[:, idx]

    # calculate the squared distances
    XC_dist = CalcDistanceSquared(x, C)
    TC_dist = CalcDistanceSquared(T, C)
    CC_dist = CalcDistanceSquared(C, C)

    return XC_dist, TC_dist, CC_dist, n, num_basis
```

これはkernel_cv内で使われているので、本筋はそちらを見たほうがいい。

- `T`　TODO


訓練データは$d$次元のもので、$n$個存在している。`num_basis`がない場合、デフォルトでベース300個とする。

次に`num_basis`個だけランダムに訓練データのidxを選択する。

そして、`XC, TC, CC`の距離をそれぞれ計算して、返す。

[距離関数](#calcdistancesquared)はユークリッド距離である。



## `CalcDistanceSquared()`

```python
def CalcDistanceSquared(X, C):
    Xsum = np.sum(X**2, axis=0).T
    Csum = np.sum(C**2, axis=0)
    XC_dist = Xsum[np.newaxis, :] + Csum[:, np.newaxis] - 2*np.dot(C.T, X)
    return XC_dist
```

与えられた訓練データ集合について、**$X$と$C$の全ての要素の間のユークリッド距離の和**を計算する。

## `PU#kernel_cv()`

非常に長いので、複数に分割する。

```python
def kernel_cv(self, x_train, t, x_test, folds=5, num_basis=False, sigma_list=None, lda_list=None):
    x_train, x_test = x_train.T, x_test.T
    XC_dist, TC_dist, CC_dist, n, num_basis = self.dist(x_train, x_test, num_basis)
```

- `x_train, t`　訓練データとそのPUのラベル。
- `x_test`　テストデータ。
- `folds`　交叉検証の時の分割数。デフォルトは5。

まず、先ほど定義した[dist](#pudist)でランダムに数点を選択させて、Cとする。そこから

- 訓練データとCの距離
- テストデータとCの距離
- C同士の距離

を求める。

```python
# setup the cross validation
cv_fold = np.arange(folds) # normal range behaves strange with == sign
cv_split0 = np.floor(np.arange(n)*folds/n)
cv_index = cv_split0[np.random.permutation(n)]
```

次に、交叉検証を行う。`cv_split0 = np.floor(np.arange(n)*folds/n)`これによって、`cv_split0`は`[0, 0, ..., 1, 1, ..., 2, 2, ...]`のような配列に。

そして、`cv_index`はつまり、`cv_split0`をシャッフルしたものである。


```python
# set the sigma list and lambda list
if sigma_list==None:
    sigma_list = np.array([0.01, 0.05, 0.1, 0.5, 1, 2, 5, 10, 20])
if lda_list==None:
    lda_list = np.array([0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1., 5.])

score_cv = np.zeros((len(sigma_list), len(lda_list)))
```

ガウシアンカーネル(カーネル関数が$\exp(|| \mathbf{x} - \mathbf{c}_l || ^ 2 / (2 \sigma ^2))$)の分散について、いろいろ候補を用意している。

ldaはTODO。

```python
for sigma_idx, sigma in enumerate(sigma_list):
    # pre-sum to speed up calculation
    h_cv = []
    t_cv = []
    for k in cv_fold:
        # 各k=テストにする分割。
        # XC_distについて、exp(- || x_i - c ||^2 / (2σ^2))となるが、cは与えられたデータで、あらかじめPU#dist()でランダムに訓練データから300個抽出した感じみたい。
        h_cv.append(np.exp(-XC_dist[:, cv_index==k]/(2*sigma**2)))
        # 今回のk=テストにする分割のデータの正解ラベル
        t_cv.append(t[cv_index==k])

    for k in range(folds):
        # calculate the h vectors for training and test
        # kがテストにする分割ID。
        # hte, tteにはテストデータのガウシアンカーネルの値と正解ラベル。
        # htr, ttrには訓練データの同上のデータが。
        count = 0
        for j in range(folds):
            if j == k:
                hte = h_cv[j].T
                tte = t_cv[j]
            else:
                if count == 0:
                    htr = h_cv[j].T
                    ttr = t_cv[j]
                    count += 1
                else:
                    htr = np.append(htr, h_cv[j].T, axis=0)
                    ttr = np.append(ttr, t_cv[j], axis=0)

        # htrは本来のhtrと全て1の列を結合。
        # hteは本来のhteと同上。
        # つまり、いずれも特徴の方。
        one = np.ones((len(htr), 1))
        htr = np.concatenate([htr, one], axis=1)
        one = np.ones((len(hte), 1))
        hte = np.concatenate([hte, one], axis=1)

        # ldaは正規化係数。正規化係数も交叉検証する。
        for lda_idx, lda in enumerate(lda_list):
            # 交叉検証して、実際にminimizeしてみる。
            res = self.minimize(htr, ttr, lda)
            # calculate the solution and cross-validation value
            score = self.pu(hte, res, tte, lda)
            score_cv[sigma_idx, lda_idx] = score_cv[sigma_idx, lda_idx] + score
```

大きなループのなかで、次のことを行っている。

```python
    # get the minimum

    # 交叉検証の結果一番よかったものを選ぶ。
    (sigma_idx_chosen, lda_idx_chosen) = np.unravel_index(np.argmin(score_cv), score_cv.shape)
    sigma_chosen = sigma_list[sigma_idx_chosen]
    lda_chosen = lda_list[lda_idx_chosen]

    x_train = np.exp(-XC_dist/(2*sigma_chosen**2)).T
    x_test = np.exp(-TC_dist/(2*sigma_chosen**2)).T

    one = np.ones((len(x_train),1))
    x_train = np.concatenate([x_train, one], axis=1)
    one = np.ones((len(x_test),1))
    x_test = np.concatenate([x_test, one], axis=1)

    return x_train, x_test, lda_chosen
```

## `PU#liner_cv()`

つかわれてない。

```python
def linear_cv(x0, x1, folds=5, lda_list=None):
    if lda_list==None:
        lda_list = np.array([0.001, 0.01, 0.1, 1.])

    scores = []
    for lda in lda_list:
        func = lambda b: self.linear_uu(b, x0, x1)
        res = self.minimize(func, b0)
        score = func(res)
        scores.append(score)

    scores = np.array(scores)
    lda = lda_list[scores.argmin()]
    return lda
```

ガウシアンカーネルを使わず、普通にminimize()させた感じ。

