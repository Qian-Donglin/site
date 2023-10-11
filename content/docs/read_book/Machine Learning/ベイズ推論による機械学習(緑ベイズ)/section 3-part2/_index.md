---
title: "第三章その2"
weight: 3
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

## 1次元ガウス分布、多次元ガウス分布

ガウス分布はハイパーパラメタとして、平均値$\mu$、分散$\sigma ^ 2$の2つがあり、どれを学習するのかで話が変わる。なお、精度$\lambda = \frac{1}{\sigma ^ 2}$とする。

なお、一次元をやりだしたら次は多次元もやりたくなる(当然の帰結)。共分散行列などで特徴をとらえやすく実用的には結構使われる。本質的には一次元と同じで、色々行列版になっただけ。多次元になると、

- データは$x \to \mathbf{x} \in \mathbb{R} ^ d$
- 平均は$\mu \to \boldsymbol{\mu}$
- 精度は$\lambda \to \boldsymbol{\Lambda}$ 共分散行列の逆行列に当たる。各次元の間でお互いに依存関係がないなら対角行列となる。共分散行列が非正則ならば疑似逆行列を使う...

なお、多次元ガウス分布は対数を取ると、

$$
\mathcal{N} ^ N (\mathbf{x} | \boldsymbol{\mu}, \boldsymbol{\Sigma}) = \frac{1}{(2 \pi) ^ {N / 2}} \frac{1}{|\boldsymbol \Sigma| ^ {1 / 2}} \exp(- \frac{1}{2} (\mathbf{x} - \boldsymbol \mu) ^ T \boldsymbol{\Sigma} ^ {-1} (\mathbf{x} - \boldsymbol \mu)) \\\\ 
\log \mathcal{N} ^ N (\mathbf{x} | \boldsymbol{\mu}, \boldsymbol{\Sigma})
= -\frac{1}{2}(N \log 2 \pi + \log |\boldsymbol{\Sigma}| + (\mathbf{x} - \boldsymbol \mu) ^ T \boldsymbol{\Sigma} ^ {-1} (\mathbf{x} - \boldsymbol \mu))
$$

### 平均の推定

観測値$x$に対して、ガウス分布を考える。平均だけ推定するので、精度$\lambda$は既知だとする。

$$
p(x | \mu) = \mathcal{N} (x | \mu, \lambda ^ {-1})
$$

この平均値$\mu$自体の事前分布も、共役事前分布の別のガウス分布に従うとする。$m, \lambda _{\mu}$はハイパーパラメタ。

$$
p(\mu) = \mathcal{N}(\mu | m, \lambda _{\mu} ^ {-1})
$$

今回、一連の観測データ$\mathbf{x} = (x_1, \cdots, x_n)$を得たとする。ここから事後分布$p(\mu | \mathbf{x})$を学習する。

$$
p(\mu | \mathbf{x}) \propto p(\mathbf{x} | \mu) p(\mu)
= \prod _{i = 1} ^ N (\mathcal{N} (x_i | \mu, \lambda ^ {-1})) \cdot \mathcal{N} (\mu | m, \lambda _{\mu} ^ {-1}) \\\\ 
$$

ここで、掛け算なので正規分布の再生性は使えない。対数を取って丁寧に計算していく。

$$
\log \mathcal{N} (x | \mu, \lambda ^ {-1}) = \frac{1}{4} \log \lambda - \frac{1}{2} \log 2 \pi + \frac{(x - \mu) ^ 2}{2} \lambda
$$

であるので、

$$
\log p(\mu | \mathbf{x}) \propto \frac{1}{4}(\log \lambda ^ {N} \lambda _{\mu}) - \frac{N + 1}{2} \log 2\pi + \frac{\lambda}{2} \sum _{i = 1} ^ {N} (x_i - \mu) ^ 2 + \frac{\lambda _{\mu}}{2} (\mu - m) ^ 2f \\\\ 
= \frac{1}{2} \mu ^ 2 (\lambda N + \lambda _{\mu}) - \mu (\lambda _{\mu} m + \lambda \sum _{i = 1} ^ N x_i) + \mathrm{const}
$$

天下り的に計算するとこれは、$p(\mu | \mathbf{x}) = \mathcal{N} (\mu | \hat{m}, \hat{\lambda _{\mu}})$と分布を書くと、

$$
\hat{\lambda _{\mu}} = N \lambda + \lambda _{\mu} \\\\ 
\hat{m} = \frac{1}{\hat{\lambda _{\mu}}} (\lambda \sum _{i = 1} ^ N x_i + \lambda _{\mu} m)
$$

これが意味するのは、

- **$\hat{m}$は観測データを集めれば集めるほど、当初の事前分布の期待値$m$の影響が薄まり、代わりに学習した$x_i$が決定に寄与する**ようになるということ。
- **$\hat{\lambda _{\mu}}$は観測データを集めれば集めるほど高くなる=分散は小さくなる**。つまり、観測データが集まるほど、$\mu$の事後分布のばらつきは小さくなる。

次は同様に予測分布を計算する。

$$
p(x _{pred}) = \int p(x _{pred} | \mu) p(\mu) d\mu
= \int \mathcal{N} (x _{pred} | \mu, \lambda ^ {-1}) \mathcal{N} (\mu | m, \lambda _{\mu} ^ {-1}) d\mu \\\\ 
\log p(x _{pred} | \mu) p(\mu) = \frac{1}{4} (\log \lambda + \lambda _{\mu}) - \frac{1}{2} (\log 2 \pi) + \frac{1}{2}(\lambda(x _{pred} - \mu) ^ 2 + \lambda _{\mu} (\mu - m) ^ 2) \\\\ 
= \frac{1}{2} \mu ^ 2 (\lambda + \lambda _{\mu}) - 2\mu (\lambda x _{pred} + \lambda _{\mu} m) + \mathrm{const}
$$

同様に、予測分布$p(x _{pred}) = \mathcal{N} (x _{pred} | \hat{m}, \hat{\lambda})$として、

$$
\frac{1}{\hat{\lambda}} = \frac{1}{\lambda} + \frac{1}{\lambda _{\mu}} \\\\ 
\hat{m} = m
$$

となる。予測分布の期待値はまさに事前分布$p(\mu)$の期待値そのものであり、**予測分布の分散は事前分布と観測分布の分散の和であると言える**。

### 多次元での平均推定

同様に、精度行列$\boldsymbol{\Lambda}$は既知であるとして、$\mathbf{x}$は、

$$
p(\mathbf{x} | \boldsymbol \mu) = \mathcal{N} ^ d (\mathbf{x} | \boldsymbol{\mu}, \boldsymbol{\Sigma})
$$

そして一次元と同様、複数個のサンプルが存在すると考えられるので、$\mathbf{X}$とする。

事前分布は、一次元の時と同様に$m \to \mathbf{m}, \lambda \to \boldsymbol{\Lambda}$。精度は例によって精度行列に。

$$
p(\boldsymbol{\mu}) = \mathcal{N} ^ d (\boldsymbol \mu | \mathbf{m}, \boldsymbol{\Lambda} _{\mu})
$$

事後分布は例によって、

$$
p(\boldsymbol \mu | \mathbf{X}) \propto p(\mathbf{X} | \boldsymbol \mu) p(\boldsymbol{\mu})
= \mathcal{N} ^ d (\mathbf{X} | \boldsymbol{\mu}, \boldsymbol{\Sigma}) \mathcal{N} ^ d (\boldsymbol \mu | \mathbf{m}, \boldsymbol{\Lambda} _{\mu}) \\\\ 
\log p(\boldsymbol \mu | \mathbf{X}) 
= -\frac{1}{2}(\sum _{i = 1} ^ N (\mathbf{x} _i - \boldsymbol{\mu}) ^ T \boldsymbol{\Lambda} (\mathbf{x} _i - \boldsymbol{\mu}) + 
(\boldsymbol{\mu} - \mathbf{m}) ^ T \boldsymbol{\Lambda} _{\mu} (\boldsymbol{\mu} - \mathbf{m})) + \mathrm{const} \\\\ 
= -\frac{1}{2} (\boldsymbol{\mu} ^ T (N \boldsymbol{\Lambda} + \boldsymbol{\Lambda} _{\mu}) \boldsymbol{\mu} - 2 \boldsymbol{\mu} (\boldsymbol{\Lambda} \sum _{i = 1} ^ N \mathbf{x} _i + \boldsymbol{\Lambda} _{\mu} \mathbf{m})) + \mathrm{const}
$$

一次元と同様に、これは対数のガウス分布のかたちであり、$\mathcal{N} ^ d (\mathbf{x} | \hat{\boldsymbol{\mu}}, \hat{\boldsymbol{\Lambda}})$として、

$$
\hat{\boldsymbol{\Lambda}} = N \boldsymbol{\Lambda} + \boldsymbol{\Lambda} _{\mu} \\\\ 
\hat{\boldsymbol{\mu}} = \hat{\boldsymbol{\Lambda}} ^ {-1}(\boldsymbol{\Lambda} \sum _{i = 1} ^ N \mathbf{x} _i + \boldsymbol{\Lambda} _{\mu} \mathbf{m})
$$

同様に、予測分布についても計算する。一次元の時と同様、$p(\mathbf{x} _{pred})$を求めていく。頑張って積分も良いが、ベイズの定理の対数版を使うと楽。

$$
\log p(\mathbf{x} _{pred}) = \log p(\mathbf{x} _{pred} | \boldsymbol{\mu}) - \log p(\boldsymbol{\mu} | \mathbf{x} _{pred})
$$

先ほどの平均予測で、$p(\boldsymbol{\mu} | \mathbf{x} _{pred})$を$p(\boldsymbol{\mu} | \mathbf{X})$の予測の式をそのまま持ってくることができ、$\mathcal{N} ^ d (\boldsymbol{\mu} | \mathbf{m}(\mathbf{x} _{pred}), (\boldsymbol{\Lambda} + \boldsymbol{\Lambda} _{\mu}) ^ {-1})$
となる(なお、$\mathbf{m}(\mathbf{x} _{pred}) = (\boldsymbol{\Lambda} + \boldsymbol{\Lambda} _{\mu}) ^ {-1} (\boldsymbol{\Lambda}\mathbf{x} _{pred} + \boldsymbol{\Lambda} _{\mu} \mathbf{m})$)。これを代入して$\mathbf{x} _{pred}$に着目すると、以下のかたちになる。
計算ミスしがちなものとして、$2 \mathbf{x} _{pred} ^ T$の項で、$(\cdots - (a \mathbf{x} + b \mathbf{y})) ^ T C (\dots - (a \mathbf{x} + b \mathbf{y}))$の$\mathbf{x}$の項は、**$\cdots$と$a \mathbf{x}$の積のみならず、$(a \mathbf{x} + b \mathbf{y}) ^ T (a \mathbf{x} + b \mathbf{y})$の$\mathbf{x}$の部分も忘れてはならない**！

$$
\mathbf{x} _{pred} ^ T (\boldsymbol{\Lambda} (\boldsymbol{\Lambda} + \boldsymbol{\Lambda} _{\mu}) ^ {-1} \boldsymbol{\Lambda}) \mathbf{x} _{pred} - 2 \mathbf{x} _{pred} ^ T (\boldsymbol{\Lambda} \boldsymbol{\mu} - \boldsymbol{\Lambda} (\boldsymbol{\Lambda} + \boldsymbol{\Lambda} _{\mu}) ^ {-1} \boldsymbol{\Lambda} _{\mu}) + \mathrm{const}
$$

(上手いこと$\boldsymbol{\Lambda} \boldsymbol{\mu}$が消える)これをもとに、ベイズの定理の対数版でまとめると、$p(\mathbf{x} _{pred} | \boldsymbol{m} _{pred}, \boldsymbol{\Lambda} _{pred})$として、

$$
\log p(\mathbf{x} _{pred}) = \mathbf{x} _{pred} ^ T (\boldsymbol{\Lambda} - \boldsymbol{\Lambda} (\boldsymbol{\Lambda} + \boldsymbol{\Lambda} _{\mu}) ^ {-1} \boldsymbol{\Lambda}) \mathbf{x} _{pred} - 2 \mathbf{x} _{pred} ^ T (\boldsymbol{\Lambda} (\boldsymbol{\Lambda} + \boldsymbol{\Lambda} _{\mu}) ^ {-1} \boldsymbol{\Lambda} _{\mu}) + \mathrm{const} \\\\ 
\boldsymbol{\Lambda} _{pred} = \boldsymbol{\Lambda} - \boldsymbol{\Lambda} (\boldsymbol{\Lambda} + \boldsymbol{\Lambda} _{\mu}) ^ {-1} \boldsymbol{\Lambda} 
= (\boldsymbol{\Lambda} ^ {-1} + \boldsymbol{\Lambda} _{\mu} ^ {-1}) ^ {-1} \\\\ 
\mathbf{m} _{pred} = \boldsymbol{\Lambda} _{pred} ^ {-1} \boldsymbol{\Lambda} (\boldsymbol{\Lambda} + \boldsymbol{\Lambda} _{\mu}) ^ {-1} \boldsymbol{\Lambda} _{\mu} \mathbf{m} = \mathbf{m}
$$

なお、$\boldsymbol{\Lambda} _{pred}$の計算はウッドベリーの公式という逆行列を作る公式を使うと求まる。期待値は$\mathbf{m}$になるのは丁寧に代入して計算すると劇的にすべて消える。信じられない！

つまり、一次元の時と結局同じ結果になった。

- 予測分布の共分散行列は、生成分布と事前分布の共分散行列の和。(ばらつきがそのまま和になる)
- 予測分布の期待値は、事前分布の期待値$\mathbf{m}$と一致する。それはそうな結果をちゃんと導けた、ヨシ！

### 精度の推定

平均$\mu$は既知であるが、精度$\lambda$を学習したい場合を考える。データ$x$は次の分布に従う。

$$
p(x | \lambda) = \mathcal{N}(x | \mu, \lambda ^ {-1})
$$

パラメタ$\lambda$は次の事前分布に従うとする。γ分布を選ぶことで、共役事前分布となる。

$$
p(\lambda) = \Gamma(\lambda | a, b) = \frac{b ^ a}{\Gamma(a)} \lambda ^ {a - 1} e ^ {-b \lambda}
$$

学習するデータは複数個あると考えると、$p(\mathbf{x} | \lambda) = \prod _{i = 1} ^ N p(x_i | \lambda)$

ここで、ベイズの定理を使って同様に事後確率$p(\lambda | \mathbf{x})$を学習してみる。

$$
p(\lambda | \mathbf{x}) \propto p(\mathbf{x} | \lambda) p(\lambda) = \mathcal{N}(\mathbf{x} | \mu, \lambda ^ {-1}) \Gamma(\lambda | a, b)
$$

ここで、確率密度関数の対数はそれぞれ以下のとおりである。

$$
\log \mathcal{N} (x | \mu, \lambda ^ {-1}) = \frac{1}{2} \log \lambda - \frac{1}{2} \log 2 \pi - \frac{(x - \mu) ^ 2}{2} \lambda \\\\ 
\log \Gamma(\lambda | a, b) = a \log b - \log \Gamma(a) + (a - 1) \log \lambda - b \lambda
$$

これを使って、$p(\lambda | \mathbf{x})$を計算し、そこから$\lambda$に関係する項だけを選び出すと、

$$
p(\mathbf{x} | \lambda) p(\lambda) = \mathcal{N}(\mathbf{x} | \mu, \lambda ^ {-1}) \Gamma(\lambda | a, b) = \prod _{i = 1} ^ N \mathcal{N}(x | \mu, \lambda ^ {-1}) \Gamma(\lambda | a, b)\\\\ 
= \frac{N}{2}(\log \lambda - \log 2 \pi) - \frac{\lambda}{2} \sum _{i = 1} ^ N (x _i - \mu) ^ 2 + a \log b - \log \Gamma(a) + (a - 1) \log \lambda - b\lambda \\\\ 
= -\lambda (\frac{1}{2} \sum _{i = 1} ^ N (x _i - \mu) ^ 2 + b) + \log \lambda (\frac{N}{2} - a + 1) + \mathrm{const}
$$

この形はγ分布の対数版と同じ形であるので、この形は$\Gamma(\lambda | \hat{a}, \hat{b})$として、

$$
\hat{a} = \frac{N}{2} + a \\\\ 
\hat{b} = \frac{1}{2} \sum _{i = 1} ^ N (x _i - \mu) ^ 2 + b
$$

これが意味することは、ようわからん。パラメタの更新がこんな風ってことかな...?

次は例によって予測分布を計算する。

$$
p(x _{pred}) = \int p(x _{pred} | \lambda) p(\lambda) d \lambda = \int \mathcal{N} (x _{pred} | \mu, \lambda ^ {-1}) \Gamma(\lambda | a, b) d \lambda
$$

これは計算してもいいが、ベイズの定理$p(\lambda | x _{pred}) = \frac{p(x _{pred} | \lambda) p(\lambda)}{p(x _{pred})}$を対数を取って、$p(\lambda)$の項を無視すれば、

$$
\log p(x _{pred}) = \log p(x _{pred} | \lambda) - \log p(\lambda | x _{pred}) + \mathrm{const}
$$

これから計算を進めると、

$$
\log p(x _{pred}) = -\frac{2a + 1}{2} \log (1 + \frac{1}{2b} (x _{pred} - \mu) ^ 2) + \mathrm{const}
$$

これは実はstudentのt分布である。

### 多次元での精度推定

平均$\boldsymbol{\mu}$は既知であり、$\boldsymbol{\Lambda}$を推定する。

$$
p(\mathbf{x} | \boldsymbol{\Lambda}) = \mathcal{N} ^ d (\mathbf{x} | \boldsymbol{\mu}, \boldsymbol{\Lambda})
$$

ここでも、実際は複数個のデータなので、$\mathbf{x}$の代わりに$\mathbf{X}$を考える。そして、$\boldsymbol{\Lambda}$の事前分布として、ウィシャート分布を与える。$\mathbf{W}$は$d \times d$の正定値行列であり、$\nu > d - 1$の実数である。

$$
p(\boldsymbol{\Lambda}) = \mathcal{W} (\boldsymbol{\Lambda} | \nu , \mathbf{W})
$$

ウィシャート分布を対数表示してみると以下の通り。カイ二乗分布の行列拡充版で、正定値行列を生成できる。その正定値行列を精度行列として使えばいいので。

$$
\mathcal{W}(\boldsymbol{\Lambda} | \nu, \mathbf{W}) = C _{w} |\boldsymbol{\Lambda}| ^ {\frac{\nu - d - 1}{2}} \exp (- \frac{1}{2} \mathrm{Tr}(\mathbf{W} ^ {-1} \boldsymbol{\Lambda})) \\\\ 
\log \mathcal{W}(\boldsymbol{\Lambda}) = \log C _{w} \frac{\nu - d - 1}{2} |\boldsymbol{\Lambda}| - \frac{1}{2} \mathrm{Tr}(\mathbf{W} ^ {-1} \boldsymbol{\Lambda})
$$

これを計算すると、多次元のstudentのt分布が出てくる。

### 平均と精度両方同時に推定

$$
p(x | \mu, \lambda) = \mathcal{N} (x | \mu, \lambda ^ {-1})
$$

この時、事前分布は以下のようなガウス・γ分布である。上記のようなガウス分布、γ分布の積という感じ。ハイパーパラメタは$m, \beta, a, b$。同様にデータは複数個ある$\mathbf{x}$と考える。
この分布は、ガウス分布が平均、γ分布が精度にあたる。グラフィカルモデルとしては、以下のように、平均も精度に依存する。

```
λ -- > μ
 \     |
  \    |
   \   |
    \  |
     \ v
      x_n
```

$$
p(\mu, \lambda) = \mathcal{N} (\mu | m, (\beta \lambda) ^ {-1}) \mathrm{Gam} (\lambda | a, b)
$$

事後分布は、以下の値となる。

$$
p(\mu, \lambda| \mathbf{x}) \propto p(\mathbf{x} | \mu, \lambda) p(\mu, \lambda) \\\\ 
= \mathcal{N} (x | \mu, \lambda ^ {-1}) \mathcal{N} (\mu | m, (\beta \lambda) ^ {-1}) \mathrm{Gam} (\lambda | a, b)
$$

ここで、$p(\mu, \lambda | \mathbf{x}) = p(\mu | \lambda, \mathbf{x}) p(\lambda | \mathbf{x})$より、前半と後半の二つに分けてそれぞれ計算する。前半の$p(\mu | \lambda, \mathbf{x})$は、平均推定の時のガウス分布のやり方と同じようにやればよい。
天下り的に最終的にはガウス・γ分布となるとわかっているので、$\mathcal{N} (\mu | \hat{m}, (\hat{\beta} \hat{\lambda}) ^ {-1}) \mathrm{Gam} (\hat{\lambda} | \hat{a}, \hat{b})$のパラメタである$\hat{m}, \hat{\beta}$を特定できる。

$$
\hat{\beta} = N + \beta \\\\ 
\hat{m} = \frac{1}{\hat{\beta}} (\sum _{i = 1} ^ N x _n + \beta m) 
$$

次は$p(\lambda | \mathbf{x})$を求める。

$$
p(\mathbf{x}, \mu, \lambda) = p(\mu | \lambda, \mathbf{x}) p(\lambda | \mathbf{x}) p(\mathbf{x}) \\\\ 
p(\lambda | \mathbf{x}) = \frac{p(\mathbf{x}, \mu, \lambda)}{p(\mu | \lambda, \mathbf{x}) p(\mathbf{x})}
\propto p(\lambda | \mathbf{x}) = \frac{p(\mathbf{x}, \mu, \lambda)}{p(\mu | \lambda, \mathbf{x})}
$$

ここから、先ほどの計算結果を代入して計算すると、$p(\lambda | \mathbf{x}) = \mathrm{Gam}(\lambda | \hat{a}, \hat{b})$として、

$$
\hat{a} = \frac{N}{2} + a \\\\ 
\hat{b} = \frac{1}{2} (\sum _{i = 1} ^ N x _i ^ 2 + \beta m ^ 2 - \hat{\beta} m ^ 2) + b
$$

さらに、予測分布もいつも通り考える。今回は2つのパラメタを予測しているので、2つの周辺確率の積分が必要。

$$
p(x _{pred}) = \int \int p(x _{pred} | \mu, \lambda) p(\mu, \lambda) d\mu d\lambda
$$

ベイズの定理で$p(x _{pred}) \propto \frac{p(x _{pred} | \mu, \lambda)}{p(\mu, \lambda | x _{pred})}$なので、
$$
\log p(x _{pred}) = \log p(x _{pred} | \mu, \lambda) - \log(\mu, \lambda | x _{pred}) + \mathrm{const}
$$

いい感じにできます(理解できなかった　むずかC)

### 多次元での平均と精度の推定

力尽きた。。。ガウス・ウィシャート分布らしいです。

## 線形回帰

線形回帰の定式化を行う。入力値$\mathbf{x} _n \in \mathbb{R} ^ d$、パラメタ$\mathbf{w} \in \mathbb{R} ^ d$、そしてノイズ$\epsilon _n \in \mathbb{R}$で$\mathcal{N} (\epsilon _n | 0, \lambda ^ {-1})$に従うとする。($\lambda$は既知)

さらに言えば、$\epsilon _n$のものを$y_n$の分布ともいえる。平行移動しただけだし。

$$
y _n = \mathbf{w} ^ T \mathbf{x} _n + \epsilon _n \\\\ 
p(y_n | \mathbf{x} _n, \mathbf{w}) = \mathcal{N} (y_n | \mathbf{w} ^ T \mathbf{x} _n, \lambda ^ {-1})
$$

$\mathbf{w}$を普通に決定するなら最小二乗法でいいが、分布によって決定されるというベイズ統計の考えだと、$\mathcal{N} ^ d(\mathbf{w} | \mathbf{m}, \boldsymbol{\Lambda} ^ {-1})$

1章でも述べてるように、例えば多項式回帰なら、データ$x _n$を$\mathbf{x} = (1, x _n, x _n ^ 2, \cdots)$と定義すればできる。

事前分布はここではガウス分布としたが、明らかにパラメタ$\mathbf{w}$に合わないことが生じる分布は望ましくない。

### 線形回帰の定式化

事前分布$p(\mathbf{w})$に対して、事後分布$p(\mathbf{w} | \mathbf{y}, \mathbf{X})$をベイズの定理によって得ることができる。

$$
p(\mathbf{w} | \mathbf{y}, \mathbf{X}) 
= \frac{p(\mathbf{w}) p(\mathbf{y} | \mathbf{w}, \mathbf{X})}{p(\mathbf{y} | \mathbf{X})}
= \frac{p(\mathbf{w}) \prod _{i = 1} ^ N p(y _i | \mathbf{x} _i, \mathbf{w})}{p(\mathbf{y} | \mathbf{X})}
\propto p(\mathbf{w}) \prod _{i = 1} ^ N p(y _i | \mathbf{x} _i, \mathbf{w}) 
$$

これは多次元ガウス分布同士の積なのでいつも通り$\mathbf{w}$に着目して対数で考える。多次元ガウス分布の平均推定の結果を借りたいところだが微妙に違うので惜しい。

$$
\log p(\mathbf{w} | \mathbf{y}, \mathbf{X})
= \log p(\mathbf{w}) + \sum _{i = 1} ^ N \log p(y _i | \mathbf{x} _i, \mathbf{w}) + \mathrm{const} \\\\ 
= (\mathbf{w} - \mathbf{m}) ^ T \boldsymbol{\Lambda} (\mathbf{w} - \mathbf{m}) + \sum _{i = 1} ^ N (y_i - \mathbf{w} ^ T \mathbf{x} _i) ^ T \lambda (y_i - \mathbf{w} ^ T \mathbf{x} _i) + \mathrm{const} \\\\ 
= \mathbf{w} ^ T (\boldsymbol{\Lambda} + \lambda \sum _{i = 1} ^ N \mathbf{x} _i \mathbf{x} _i ^ T) \mathbf{w} + 
2 \mathbf{w} ^ T (\boldsymbol{\Lambda} \mathbf{m} + \lambda \sum _{i = 1} ^ N y _i \mathbf{x} _i)
$$

よって、$p(\mathbf{w} | \mathbf{y} | \mathbf{X}) = \mathcal{N} (\mathbf{w} | \hat{\mathbf{m}}, \hat{\boldsymbol{\Lambda}})$として、

$$
\hat{\boldsymbol{\Lambda}} = \boldsymbol{\Lambda} + \lambda \sum _{i = 1} ^ N \mathbf{x} _i \mathbf{x} _i ^ T \\\\ 
\hat{\mathbf{m}} = \hat{\boldsymbol{\Lambda}} ^ {-1} (\boldsymbol{\Lambda} \mathbf{m} + \lambda \sum _{i = 1} ^ N y _i \mathbf{x} _i)
$$

ということは、予測分布の$p(y _{pred} | \mathbf{x} _{pred}, \mathbf{X}, \mathbf{Y})$も計算できるはずである。
ここでは、先ほどの結果を例によって利用するベイズの定理の式変形を考えて解くことができる。

$$
p(\mathbf{w} | y _{pred}, x _{pred}) = \frac{p(\mathbf{w}) p(y _{pred} | \mathbf{x} _{pred}, \mathbf{w})}{p(y _{pred} | \mathbf{x} _{pred})} \\\\ 
\log p(y _{pred} | \mathbf{x} _{pred}) = \log p(y _{pred} | \mathbf{x} _{pred}, \mathbf{w}) - \log p(\mathbf{w} | y _{pred}, \mathbf{x} _{pred}) + \mathrm{const}
$$

先ほどの$\mathbf{w}$の事後分布で、$\mathbf{X} \to \mathbf{x} _{pred}$とすれば、$p(\mathbf{w} | y _{pred}, \mathbf{x} _{pred})$を得る事ができる。($\lambda$は与えた$\mathbf{x} _{pred}$の精度、$\mathbf{m}$は$\mathbf{w}$の事前分布の平均)

$$
\log p(\mathbf{w} | y _{pred}, \mathbf{x} _{pred}) = (\mathbf{w} - \mathbf{m}(\mathbf{x} _{pred})) ^ T \boldsymbol{\Lambda} _{pred} (\mathbf{w} - \mathbf{m}(\mathbf{x} _{pred})) \\\\ 
\boldsymbol{\Lambda} _{pred} = (\lambda \mathbf{x} _{pred} \mathbf{x} _{pred} ^ T + \boldsymbol{\Lambda}) ^ {-1} \\\\ 
\mathbf{m} (\mathbf{x} _{pred}) = \boldsymbol{\Lambda} _{pred} (\lambda y _{pred} \mathbf{x} _{pred} + \boldsymbol{\Lambda} \mathbf{m})
$$

同様に、$p(y _{pred} | \mathbf{x} _{pred}, \mathbf{w})$は前述の平行移動した分布だと考えられる。つまり、$\mathcal{N} (y _{pred} | \mathbf{w} ^ T \mathbf{x} _{pred}, \lambda)$である。

これらをもとに$y _{pred}$でまとめると、

$$
\log p(\mathbf{w} | y _{pred}, \mathbf{x} _{pred}) = y _{pred} ^ 2 (\lambda - \lambda ^ 2 \mathbf{x} _{pred} ^ T (\lambda \mathbf{x} _{pred} \mathbf{x} _{pred} ^ T + \boldsymbol{\Lambda}) ^ {-1} \mathbf{x} _{pred}) \\\\ - 
2 y _{pred} (\lambda \mathbf{x} _{pred} ^ T (\lambda \mathbf{x} _{pred} \mathbf{x} _{pred} ^ T + \boldsymbol{\Lambda}) ^ {-1} \boldsymbol{\Lambda} \mathbf{m})
$$

整理するとやはりきれいになる。

$$
p(y _{pred} | \mathbf{x} _{pred}) = \mathcal{N} (y _{pred} | \hat{\mathbf{m}}, \hat{\lambda}) \\\\ 
\hat{\mathbf{m}} = \mathbf{m} ^ T \mathbf{x} _{pred} \\\\ 
\hat{\lambda} ^ {-1} = \lambda ^ {-1} + \mathbf{x} _{pred} ^ T \boldsymbol{\Lambda} ^ {-1} \mathbf{x} _{pred}
$$

平均値の推定と似た結果になった。すなわち、

- 予測分布の平均値は、$\mathbf{w}$の事前分布$\mathcal{N} (\mathbf{w} | \mathbf{m}, \boldsymbol{\Lambda})$の中央値$\mathbf{m}$である。
  - つまり、事前分布の期待値そのもの。
- 予測分布の分散は、データ自体の分散$\lambda ^ {-1}$と、$\mathbf{w}$の事前分布の分散と予測値$\mathbf{x} _{pred}$の二次形式の和。

## モデル選択

モデルを選び間違えると学習してもいい結果につながらない。色々選んで検討を重ねるべし。モデルの良さを評価する場合、低次元データなら可視化が一番いいが、高次元にもなるとそうはいかない。評価する量は周辺尤度というもの。

**周辺尤度、エビデンス**は、今考えるモデルに基づいてデータ$\mathbf{X}$から実際にラベル$\mathbf{y}$を生成する尤もらしさである。
これは下式のように$p(\mathbf{y} | \mathbf{X})$を流用すればいい。

$$
p(\mathbf{y} | \mathbf{X}) = \frac{p(\mathbf{w}) \prod _{i = 1} ^ N p(y _i | \mathbf{x} _i, \mathbf{w})}{p(\mathbf{w} | \mathbf{y}, \mathbf{X})}
$$

この式自体は右から計算できるが、$[0, 1]$に収まるという保証はない(完全に理想的なものなら収まるがそこはもうどうしようもない)。この式を計算するときは、式変形すると以下のような対数化した周辺尤度が望ましい。なお、$\hat{\mathbf{m}}$などは$\mathbf{w}$の事後分布のものである。(まじ？)

$$
\log p(\mathbf{y} | \mathbf{X}) = - \frac{1}{2} (\sum _{i = 1} ^ N (\lambda y _i ^ 2 - \log \lambda + \log 2 \pi) + \mathbf{m} ^ T \boldsymbol{\Lambda} \mathbf{m} - \log |\Lambda| - \hat{\mathbf{m}} ^ T \hat{\boldsymbol{\Lambda}} \hat{\mathbf{m}} + \log |\hat{\boldsymbol{\Lambda}}|)
$$

## 最近傍法

与えられた点に対して、最も距離(定義による)が近い点のラベルを予測とすること。つまり、下式のように各成分との距離で測る感じ。

$$
y _{pred}= \argmin _{n \in 1, \cdots, N} \sum _{i = 1} ^ d (x _{n, i} - x _{pred, i}) ^ 2
$$

この手法は先ほどのベイズ推定とは違う。先ほどはパラメタ数は固定であり、データ数が増えても最初に規定した分布の概形は変わらないが、この手法ではデータ数が増えるにつれて予測の線がどんどんアップデートされていくようになる(ついでにいえばベイズ推定でも当然ない)。これは**ノンパラメトリックな手法**である。

ただ明確に最近傍法は愚直に実装したら$O(Nd)$、上手いこと二分探索とかやれても良くて$O(d\log N)$の時間計算量が必要であり、どうあがいても遅いのが難点。ベイズ推定は一度求まれば$O(1)$でわかる。

## 最尤推定

なんでわざわざパラメタ$\mathbf{a}$はそのまま求めずに分布に従うとか言うんですか？そのままやればいいじゃないですか。これが最尤推定です。

$$
\argmax _{\theta} p(\mathbf{X} | \theta)
$$

最尤推定は最適な1点を推定する=点推定となるが、無限に適合できてしまうので過学習はもちろんひどい。また、これからやる混合モデルの推定を行うにあたり、0除算や非正則の行列なので逆行列持たないとか色々面倒なので非実用的。

## MAP推定

最尤推定に救いはないんですか？あります。事前分布を導入しておきましょう。これで過学習は起きなくなる。

ベイズの定理を使って、尤度$p(\mathbf{X} | \theta)$ではなく、事後確率$p(\theta | \mathbf{X})$を推定している。

$$
\argmax _{\theta} p(\mathbf{X} | \theta) p(\theta) = \argmax _{\theta} p(\theta | \mathbf{X})
$$

**ベイズ推定もMAP推定と同じ、最大値を取るような事後分布のパラメタを選ぶことであるが、ただ1点違うのはパラメタ$\theta$を直接推定するのではなく、わざわざ分布だとおいて分布のまま推定することである**。