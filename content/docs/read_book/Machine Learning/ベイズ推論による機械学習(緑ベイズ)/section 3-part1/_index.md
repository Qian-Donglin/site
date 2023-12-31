---
title: "第三章その1"
weight: 3
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

## 学習と予測

ベイズ統計では、**各値は一定ではなくある分布に代表される**というもの。
なので、**ベイズ統計での機械学習とは、パラメタの事前分布と、学習データを前提条件とした事後分布を求める**ことに当たる。

つまり、訓練データ$\mathbf{D}$を用いて、知りたい未知のパラメタ$\theta$は、

$$
p(\mathbf{D}, \theta) = p(\mathbf{D} | \theta) p(\theta)
$$

このような式となる。この$p(\mathbf{D} | \theta), p(訓練データ | 知りたいパラメタ)$は尤度関数という。逆の$p(\theta | \mathbf{D}), p(知りたいパラメタ | 訓練データ)$は事後分布。ベイズの定理によって、尤度関数から事後分布を計算できる。

$$
p(\theta | \mathbf{D}) = \frac{p(\mathbf{D} | \theta) p(\theta)}{p(\mathbf{D})}
$$

**この左辺を計算することが学習にあたる**。

### 予測分布

1章にもあったように、訓練済のパラメタを使って予測結果を得るのは、以下の式になる。グラフィカルモデルからも同じような形が得られる。

$$
p(x_a | \mathbf{D}) = \int p(x_a | \theta) p(\theta | \mathbf{D}) d \theta \\\\
p(x_a, \theta | \mathbf{D}) = \frac{p(x_a, \theta, \mathbf{D})}{p(\mathbf{D})} = \frac{p(\mathbf{\theta}) p(x_a | \mathbf{\theta}) p(\mathbf{D} | \theta)}{p(\mathbf{D})}
$$

この枠組み、パラメタの事後分布$p(\theta | \mathbf{D})$がすべての学習データ$D$を持つことになるが、**データ量に対してモデルの表現能力が変化しないという大きな制限をもってしまう**。ガウス過程などのベイジアンノンパラメトリクスの手法で改善できるらしい。この本はやらない。

### 共役事前分布

事前分布$p(\theta)$と事後分布$p(\theta | \mathbf{D})$が、数学的な計算をすると**全く同形になる分布のことを共役事前分布**という。

逆に言えば普通は同じ分布にならない。ならないので面倒な積分を解く必要があるし、厳密に計算できるとも限らない場合は常々ある。

どのような事前分布が共役になるかは、尤度関数$p(\mathbf{D} | \theta)$の設計に依存する。

**共役性があると、同じ分布のパラメタの関係式があって、それで積分せずに目的の分布の値がわかっちゃう！すごい！**

特にデータを小分けしながら学習してアップデートする場合、一章の逐次学習の式では、

$$
p(\theta | D_n, D_{n - 1}, \cdots, D_1) \propto p(\theta | D_{n - 1}, \cdots , D_1) p(D_n | \theta)
$$

で学習できる(ただし、それぞれの小分けしたデータは互いに独立であるという前提で)。
ここで共役事前分布を使うことで、上の逐次学習にあるすべての項は同じ形の分布なのでやりやすい。
そうでないときは色々やり方があるが、一例としてはKLダイバージェンスを最小化するとか。ここら辺の最適化は共役勾配法とかで解くしかないね。

#### 共役事前分布の表

| 尤度関数         | パラメタ                            | 共役事前分布             | 予測分布               |
| ---------------- | ----------------------------------- | ------------------------ | ---------------------- |
| ベルヌーイ分布   | $\mu$                               | β分布                    | ベルヌーイ分布         |
| 二項分布         | $\mu$                               | β分布                    | β・二項分布            |
| カテゴリ分布     | $\mathbf{\pi}$                      | ディリクレ分布           | カテゴリ分布           |
| 多項分布         | $\mathbf{\pi}$                      | ディリクレ分布           | ディリクレ・多項分布   |
| ポアソン分布     | $\lambda$                           | γ分布                    | 負の二項分布           |
| 1次元ガウス分布  | $\mu$                               | 1次元ガウス分布          | 1次元ガウス分布        |
| 1次元ガウス分布  | $\sigma ^ 2$                        | γ分布                    | 1次元のstudentのt分布  |
| 1次元ガウス分布  | $\mathbf{\mu}, \sigma ^ 2$          | ガウス・γ分布            | 1次元のstudentのt分布  |
| 多次元ガウス分布 | $\mathbf{\mu}$                      | 多次元ガウス分布         | 多次元ガウス分布       |
| 多次元ガウス分布 | $\mathbf{\sigma} ^ 2$               | ウィシャート分布         | 多次元のstudentのt分布 |
| 多次元ガウス分布 | $\mathbf{\mu}, \mathbf{\sigma} ^ 2$ | ガウス・ウィシャート分布 | 多次元のstudentのt分布 |

## 離散確率分布の学習と予測

$$
p(x | \mu) = \mathrm{Bern}(x | \mu) = \mu ^ x (1 - \mu) ^ {1 - x}
$$

に従い、ベルヌーイ分布を考える。

**ベイズ統計ではパラメタは定まった値ではなく、ある分布に従って値を産むという考え**。なので、**パラメタ$\mu \in (0, 1)$を産む分布=共役事前分布**を考える。
ここで、[表](#共役事前分布の表)によれば、ベルヌーイ分布の共役事前分布は**β分布**。
よって、以下のようになる。パラメタ$a, b$は**モデルの生成する値をコントロールするハイパーパラメタであり**、ここではあらかじめ与えられてると考える。

$$
p(\mu) = \mathrm{Beta}(\mu | a, b) = \frac{\Gamma(a + b)}{\Gamma(a - 1) \Gamma(b - 1)} \mu ^ {a - 1} (1 - \mu) ^ {b - 1}
$$

この事前分布$p(\mu)$と訓練データ点の分布$p(x)$について、事後分布$p(\mu | x)$を計算したい。ベイズの定理により、以下が成り立ち代入する。ここで、分布の概形だけわかってあとで正規化係数を入れればいいので、概形だけに着目する。

$$
p(\mu | x) = \frac{p(x | \mu) p(\mu)}{ p(x)} \propto p(x | \mu) p(\mu) \\\\ 
\propto \mu ^ x (1 - \mu) ^ {1 - x} \times \mu ^ {a - 1} (1 - \mu) ^ {b - 1} 
= \mu ^ {a - 1 + x} (1 - \mu) ^ {b - x}
$$

やはり、**概形は事前分布のβ分布と同じ形になっている**。$x = 1$の時は$a \leftarrow a + 1$で、$x = 0$の時は$b \leftarrow b + 1$となる。意味付けとしては、

- β分布の$a, b$はそれぞれ事前に$a - 1, b - 1$回コインの表と裏が出たことを意味している。
- ベルヌーイ試行を通して、**β分布のハイパーパラメタをアップデートした形**。

なお、経験ベイズ法(Empirical Bayes)はハイパーパラメタの値をも学習によって変動させるやつ。ハイパーパラメタにも事前分布を用意するという感じ。

この例では、事前分布がお互い違うとしても、やはりどんどん近づいていくということに。ただし、**事前分布で0と設定されている確率密度=絶対に無いと確信している場合、いくら学習を繰り返しても事後分布ではそこは0から変わらない**。

次に、未観測の値$x_n \in 0, 1$に対して予測を行ってみる。今までの訓練データを$\mathbf{X}$とする。また事後分布は$\mathrm{Beta}(\mu | a, b)$に従うと学習で分かったとする。

$$
p(x_n | \mathbf{X}) = \int p(x_n | \mu) p(\mu | \mathbf{X}) d\mu
= \frac{\Gamma(a + b)}{\Gamma(a) \Gamma(b)} \int \mu ^ {x_n} (1 - \mu) ^ {1 - x_n} \times \mu ^ {a - 1} (1 - \mu) ^ {b - 1} d\mu \\\\ 
= \frac{\Gamma(a + b)}{\Gamma(a) \Gamma(b)} \int \mu ^ {a - 1 - x_n} (1 - \mu) ^ {b - x_n} d\mu
$$

β関数について、以下の式が成り立つことを利用して式変形をする。

$$
\int \mu ^ {a - 1} (1 - \mu) ^ {b - 1} d\mu = \frac{\Gamma(a) \Gamma(b)}{\Gamma(a + b)} \\\\ 
\frac{\Gamma(a + b)}{\Gamma(a) \Gamma(b)} \int \mu ^ {a - 1 - x_n} (1 - \mu) ^ {b - x_n} d\mu 
= \frac{\Gamma(a + b)}{\Gamma(a) \Gamma(b)} \frac{\Gamma(a + x_n) \Gamma(b - x_n + 1)}{\Gamma(a + b + 1)} \\\\ 
= \frac{\Gamma(a + b)}{\Gamma(a + b + 1)} \frac{ \Gamma(a + x_n) \Gamma(b - x_n + 1)}{\Gamma(a) \Gamma(b)}
= \frac{a}{a + b} (\mathrm{if} \:\: x_n = 1), \frac{b}{a + b} (\mathrm{if} \:\: x_n = 0)
$$

このように予測分布を得られた。β分布の期待値でもある。そして、**この予測分布出さえ、ベルヌーイ分布である**。

$$
(\frac{a}{a + b}) ^ {x_n} (\frac{b}{a + b}) ^ {1 - x_n} = \mathrm{Bern}(x_n | \frac{a}{a + b})
$$

## カテゴリ分布の学習と予測

先ほどは2択試行を1回だったベルヌーイ分布であったが、今回は3択以上1回の選択のカテゴリ分布を考える。パラメタ$\mathbf{\pi}$を使って、

$$
p(\mathbf{x} | \mathbf{\pi}) = \mathrm{Cat}(\mathbf{x} | \mathbf{\pi}) = \prod _{i = 1} ^ {N} \pi_i ^ {x_i} \\\\ 
\log p(\mathbf{x} | \mathbf{\pi}) = \sum _{i = 1} ^ {N} x_i \log \pi_i
$$
パラメタ$\mathbf{\pi}$は、**カテゴリ分布の共役事前分布であるディリクレ分布に従う**。$\mathbf{\alpha}$はハイパーパラメタ。

$$
p(\mathbf{\pi}) = \mathrm{Dir} (\mathbf{\pi} | \mathbf{\alpha}) = \frac{\Gamma(\sum _{i = 1} ^ N \alpha_i)}{\prod _{i = 1} ^ N \Gamma(\alpha_i)} \prod _{i = 1} ^ {N} \pi _i ^ {\alpha _i - 1} \\\\ 
\log p(\pi) = \sum _{i = 1} ^ N (\alpha _i - 1) \log \pi _i + \log \Gamma ( \sum _{i = 1} ^ {N} \alpha_i) - \sum _{i = 1} ^ N \Gamma(\alpha _i)
$$

累乗は扱いづらいので、ここでlogを取ったもので計算する。
なお、概形を見るために、係数に関してはいつも通り無視しておく。

$$
\log p(\mathbf{\pi} | \mathbf{x}) \propto \log p(\mathbf{x}, \mathbf{\pi}) = \log p(\mathbf{x} | \mathbf{\pi}) p(\mathbf{\pi}) \\\\ 
= \sum _{i = 1} ^ {N} x_i \log \pi_i + \sum _{i = 1}^{N} (\alpha_i - 1) \log \pi_i = \sum _{i = 1} ^ {N} (\alpha_i - 1 + x_i) \log \pi_i \\\\ 
p(\mathbf{\pi} | \mathbf{x}) \propto \prod _{i = 1} ^ {N} \pi_i ^ {\alpha_i - 1 + x_i}
$$

このように事後分布も、ディリクレ分布。ハイパーパラメタは$\mathbf{\hat{\alpha}} = (\alpha_1 + x_1, \cdots, \alpha_N + x_N)$。
これも、事前分布が学習によって事後分布となって記憶されるということになる。

また、予測を考えてみる。予測は

$$
\int p(\mathbf{x} _{new} | \mathbf{\pi}) p(\mathbf{\pi}) d\pi
= \int \mathrm{Cat}(\mathbf{x} _{new} | \mathbf{\pi}) \mathrm{Dir}(\mathbf{\pi} | \mathbf{\alpha}) d \mathbf{\pi}
$$

$$
\propto \int \prod _{i = 1} ^ {N} \pi_i ^ {x _{new} ^ {i}} \prod _{i = 1} ^ {N} \pi _{i} ^ {\alpha_i - 1} d \mathbf{\pi}
= \int \prod _{i = 1} ^ {N} \pi_i ^ {x _{new} ^ {i} + \alpha_i - 1} d\pi
$$

## ポアソン分布の学習と予測

ポアソン分布の意味は、ある指定期間の間に$\lambda$回起きる、とわかっている事象が$x$回起こる確率の分布。

ポアソン分布の事後分布は、

$$
p(x | \lambda) = \mathrm{Poi}(x, \lambda) = \frac{\lambda ^ x}{x!} e ^ {-\lambda}
$$

共役事前分布はγ分布であると知られている。

$$
p(\lambda) = \mathrm{Gam}(\lambda | a, b) = \frac{b ^ a}{\Gamma(a)} \lambda ^ {a - 1} e^ {-b \lambda}
$$

よって、学習すると、

$$
p(x | \lambda) p(\lambda) = \frac{b ^ a}{\Gamma(a) \Gamma(x + 1)} \lambda ^ {x + a - 1} e ^ {-(b + 1)\lambda} \\\\ 
= \frac{b ^ a}{\Gamma(a + x + 1)} \lambda ^ {x + a - 1} e ^ {-(b + 1)\lambda}
$$

これは、パラメタが$x + a, b + 1$のγ分布となる。$b$は回数、$a$は重みみたいな感じ。

予測分布の計算もしてみる。

$$
\int p(x_{new} | \lambda) p(\lambda) d \lambda
= \int \mathrm{Poi} (x_{new} | \lambda) \mathrm{Gam}(\lambda | a, b) d \lambda \\\\ 
= \int \frac{b ^ a}{\Gamma(a + x + 1)} \lambda ^ {x_{new} + a - 1} e ^ {-(b + 1)\lambda} d \lambda
$$

これを計算すると、負の二項分布となるらしい。
普通の二項分布は試行の回数を固定。負の二項分布は失敗回数を固定。

