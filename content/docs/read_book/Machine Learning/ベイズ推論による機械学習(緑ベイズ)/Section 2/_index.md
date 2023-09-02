---
title: "第二章"
weight: 1
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# 期待値

$$
\mathbb{E} [f(\mathbf{x})] = \int f(\mathbf{x}) p(\mathbf{x}) d\mathbf{x}
$$

と定義される。純粋に$p(\mathbf{x})$の取る平均は$\mathbb{E} [\mathbf{x}] = \int \mathbf{x} p(\mathbf{x}) d\mathbf{x}$

期待値は独立かそうじゃないか関わらず、線形性を持つ。

分散は、
$$
\mathbb{E} [(\mathbf{x} - \hat{\mathbf{x}}) ^ T (\mathbf{x} - \hat{\mathbf{x}})]
= \mathbb{E} [\mathbf{x} ^ T \mathbf{x}] - \mathbb{E} [\mathbf{x} ^ T] \mathbb{E} [\mathbb{x}]
$$

## 2変数以上の期待値

$$
\mathbb{E} _{p(\mathbf{x}, \mathbf{y})} [\mathbf{x} ^ T \mathbf{y}]
$$

を計算する。もし$\mathbf{x}, \mathbf{y}$が**お互いに独立**であるなら、$p(\mathbf{x}, \mathbf{y}) = p(\mathbf{x}) p(\mathbf{y})$であり、

$$
\int \int \mathbf{x} ^ T \mathbf{y} p(\mathbf{x, y})d\mathbf{x} d\mathbf{y} 
= \int \mathbf{x} ^ T p(\mathbf{x}) d\mathbf{x} + \int \mathbf{y} p(\mathbf{y}) d \mathbf{y}
= \mathbb{E} [\mathbf{x} ^ T] \mathbb{E} [\mathbf{y}]
$$

が成り立つ。
しかし、独立ではない場合は、上の積分を丁寧に行うしかない。これは条件付期待値。

$$
\int \int \mathbf{x} ^ T \mathbf{y} p(\mathbf{x, y})d\mathbf{x} d\mathbf{y} 
= \mathbb{E}_{\mathbf{y}} [ \mathbb{E} _{\mathbf{x | y}} [\mathbf{x} ^ T \mathbf{y}] ]
$$

# エントロピー

エントロピー$H(\mathbf{x})$を定める。大きいほど乱雑で、0が一番整っている。ここで$\log 0 = 0$と定義。

$$
H(\mathbf{x}) = -\mathbb{E} [\log p(\mathbf{x})] = -\int p(\mathbf{x}) \log p(\mathbf{x}) d \mathbf{x}
$$

# KLダイバージェンス

2つの分布$p(\mathbf{x}), q(\mathbf{x})$について、**疑似的に分布の距離っぽいなにか(距離の公理は満たさない！！厳密にやるなら最適輸送)**。これをKLダイバージェンス。

$$
\mathrm{KL} [p(\mathbf{x}) || q(\mathbf{x})] = \mathbb{E}_{p(\mathbb{x})} [\log p(\mathbf{x})] - \mathbb{E} _{p(\mathbb{x})} [\log q(\mathbf{x})]
= -\int q(\mathbf{x}) \log \frac{q(\mathbf{x})}{p(\mathbf{x})} d\mathbf{x}
$$

# サンプリングによる期待値の推定

分散はそうでもないけど期待値はサンプリングの平均は不偏推定量なんで、$\frac{1}{n} \sum _{i = 1} ^ {n} f(\mathbf{x}_i)$

# 離散確率分布

|            | 事象2つ    | 事象3つ以上 |
| ---------- | ---------- | ----------- |
| 1回試行    | ベルヌーイ | カテゴリ    |
| 複数回試行 | 二項       | 多項        |

## ベルヌーイ分布

1回のコイン投げで、表の確率が$\mu$、裏が$1 - \mu$の時、$x$を取る確率は以下のようにまとめられる。
$x = 0$なら$1 - \mu$, $x = 1$なら$\mu$になるのをまとめている。

$$
\mathrm{Bern}(x | \mu) = \mu ^ x (1 - \mu) ^ {1 - x}, x \in \{0, 1\}
$$

なお、エントロピーは$- \mathbb{E} [\log p(\mathbf{x})] = -(1 - \mu)\log (1 - \mu) - \mu \log \mu$となり、この形は明らかに$\mu = 0.5$で最大値を取る。

## 二項分布

複数回のベルヌーイ分布をやる。$M$回の試行を繰り返し、表が出るのは$\mu$の確率である。
この時、**$x$回表が出る確率**は以下のようになる。

$$
\mathrm{Bin}(x | M, \mu) = _M C _x \mu ^ x (1 - \mu) ^ {M - x} 
$$

明らかに上式は$M = 1$において、ベルヌーイ分布と同じようになる。$x = 0$ならば表が出ない、$x = 1$ならば表が出る。$M > 1$だと$x$は回数という意味だが、$x = 1$である限りベルヌーイ分布と同じ。

主な期待値は以下のようになる。

$$
\mathbb{E} [x] = M \mu \\\\ 
\mathbb{E} [x ^ 2] = M \mu ((M - 1) \mu + 1)
$$

## カテゴリ分布

ベルヌーイ分布は表と裏の2通りしかないが、これを3通り以上の状態に拡張する。
$\mathbf{s}$はカテゴリを示すone-hotベクトル。$\mathbf{\pi}$は各カテゴリにおける出現確率。
以下の式では総乗であるが、$s_i = 1$は一つだけなので、そこの$\pi_i$だけ出てそれ以外は$1$となる。掛け合わせると$\pi_i$が出てくる。

$$
\mathrm{Cat} (\mathbf{s}, \mathbf{\pi}) = \prod_{i = 1} ^ {N} \pi_i ^ {s_i}
$$

同様にエントロピーを計算すると、

$$
-\mathbb{E} [\log \prod_{i = 1} ^ {N} \pi _i ^ {s_i}] = - \sum _{i = 1} ^ {N} \pi _i \log \pi _i
$$

## 多項分布

二項分布においてカテゴリを3つ以上にも拡張したもの。3つ以上の種類の$N$個の球を1列に並べる並び方が$\frac{N!}{a!b!c!\cdots}, a + b + c + \cdots = N$であるので、同様に分布の式も以下のようになる。$\mathbf{x}$は各カテゴリにおける出現数。

$$
\mathrm{Mult}(\mathbf{x} | \mathbf{\pi}, M) = M! \prod _{i = 1} ^ {N} \frac{\pi_i ^ {x_i}}{x_i !}
$$

これの期待値は、

$$
\mathbb{E} [ x_i ] = M \pi_i \\\\ 
\mathbb{E} [ x_i x_j ] = M \pi_i ((M - 1)\pi_i + 1) \\:\\: \mathrm{if} j = k \\\\ 
= M(M - 1) \pi_i \pi_j \\: \\: \mathrm{otherwise}
$$

## ポアソン分布

非負の整数$x$について、以下の確率で生成する。

$$
\mathrm{Poi} (x | \lambda) = \frac{\lambda ^ x}{x!} e^ {-\lambda} \\\\ 
\log \mathrm{Poi} (x | \lambda) = x \log \lambda - \log x! - \lambda
$$

$x$が大きくなると明らかに確率は下がるが、**完全には0にならない**のが二項分布とかとの違い。

$$
\mathbb{E} [x] = \lambda \\\\ 
\mathbb{E} [x^2] = \lambda (\lambda + 1)
$$

# 連続確率分布

## β分布

**$x \in (0, 1)$の値を生成する分布**。$\Gamma(a) = (a - 1)!$のガンマ関数(非自然数も定義しているが)　下のガンマ関数部は**正規化項**。
意味していることは**表の確率が$x$コインを投げて表$a - 1$回、裏$b - 1$回が出る分布**。

$$
\mathrm{Beta}(x | a, b) = \frac{\Gamma(a + b)}{\Gamma(a) \Gamma(b)} x ^ {a - 1} (1 - x) ^ {b - 1} \\\\ 
\log \mathrm{Beta}(x | a, b) = (a - 1) \log x + (b - 1) \log (1 - x) + \log \frac{\Gamma(a + b)}{\Gamma(a) \Gamma(b)}
$$

期待値は以下の通り。$\psi(x) = \frac{d}{dx} \log \Gamma(x) = \frac{\Gamma ^ {\prime}(x)}{\Gamma(x)}$
というdigamma関数だとすると、

$$
\mathbb{E} [x] = \frac{a}{a + b} \\\\ 
\mathbb{E} [\log x] = \psi(a) - \psi(a + b) = \mathbb{E} [\log (1 - x)] \\\\ 
$$

なお、同じ$a = 1, b = 2$と$a = 3, b = 9$でも、期待値は同じだけど、後者の方がとがっている分布になる。**試行回数が増えてより自信をもって確実に言える**みたいなもの。

β分布は、**ベルヌーイ分布と二項分布の共役事前分布**である。

## ディリクレ分布

β分布は連続かつ、$x$か$1 - x$の二択であったが、これを三択以上に拡張したもの。
同様に**$x \in (0, 1)$の値を生成する分布**である。

$$
\mathrm{Dir}(\mathbf{x} | \mathbf{\alpha}) = \frac{\Gamma(\sum _{i = 1}^{K} \alpha_i)}{\prod _{i = 1}^{K} \Gamma(\alpha_i)} \prod _{i = 1}^{N} c_i ^ {\alpha_i - 1} \\\\ 
\log \mathrm{Dir}(\mathbf{x} | \mathbf{\alpha}) = \sum _{i = 1} ^ K (\alpha_i - 1) \log x_i + \log \frac{\Gamma(\sum _{i = 1}^{K} \alpha_i)}{\prod _{i = 1}^{K} \Gamma(\alpha_i)}
$$

期待値関連はβ分布と似ている。

$$
\mathbb{E} [x_k] = \frac{\alpha_k}{\sum _{i = 1} ^ {K} \alpha_i } \\\\ 
\mathbb{E} [\log x_k] = \psi(\alpha_k) - \psi(\sum _{i = 1}^{K} \alpha_i)
$$

**ディリクレ分布は、β分布からして、カテゴリ分布と多項分布の共役事前分布**。

## γ分布

正の実数$x$を生成する。

$$
\mathrm{Gam}(x | a, b) = \frac{b ^ a}{\Gamma(a)} x ^ {a - 1} e ^ {-b \lambda} \\\\ 
\log \mathrm{Gam}(x | a, b) = \log (a - 1)x - b \lambda + \log \frac{b ^ a}{\Gamma(a)}
$$

期待値は以下の通り。

$$
\mathrm{E} [x] = \frac{a}{b} \\\\ 
\mathrm{E} [\log x] = \psi(a) - \log b
$$

**ガンマ分布はポアソン分布と1次元ガウス分布の分散の逆数の共役事前分布である**。

## ガウス分布

期待値$mu$, 分散$\sigma$のパラメタを持つ。

$$
\mathcal{N} (x | \mu, \sigma^2) = \frac{1}{\sqrt{2 \pi} \sigma} \exp( - \frac{(x - \mu) ^ 2}{2 \sigma^2}) \\\\ 
\log \mathcal{N} (x | \mu, \sigma^2) = -\frac{1}{2} (\frac{(x - \mu) ^ 2}{2 \sigma^2} + 2\log \sigma + \log 2 \pi)
$$

期待値として、

$$
\mathbb{E} [x] = \mu \\\\ 
\mathbb{E} [x^2] = \mu^2 + \sigma^2
$$

エントロピーは以下の通り。

$$
- \mathbb{E} [\log p(x)] = \mathbb{E} \frac{1}{2} [\frac{x^2 - 2 \mu x + \mu^2}{\sigma ^ 2} + 2 \log \sigma + \log 2 \pi ]\\\\ 
= \frac{1}{2} \mathbb{E}[\frac{\mu ^ 2 + \sigma ^ 2 - 2\mu^2 + \mu^2}{\sigma ^ 2} + 2 \log \sigma + \log 2 \pi]
= \frac{1}{2} (1 + 2 \log \sigma + \log 2 \pi)
$$

2つのガウス分布$\mathcal{N} (\mu_1, \sigma_1 ^ 2)$と$\mathcal{N} (\mu_2, \sigma_2 ^ 2)$のKLダイバージェンスは、

$$
\mathbb{E}_{p} [\log p(x)] - \mathbb{E}_{p} [\log q(x)]
= -\frac{1}{2} (\frac{(\mu_1 - \mu_2) ^ 2 + \sigma_2 ^ 2}{\sigma_1 ^ 2}) + 2\log \frac{\sigma_1}{\sigma_2} - 1
$$

## 多次元ガウス分布

先ほどのは1変数であったが、これを多次元にしたもの。$\mathbf{\mu}$は期待値のベクトル、$\mathbf{\Sigma}$は今日分散行列。まあ正定値で対称ですね。
$D$は次元数。

$$
\mathcal{N} (\mathbf{x} | \mathbf{\mu}, \mathbf{\Sigma})
= \frac{1}{\sqrt{2\pi} ^ D |\Sigma|} \exp(-\frac{1}{2} (\mathbf{x} - \mathbf{\mu}) ^ T \mathbf{\Sigma} ^ {-1} (\mathbf{x} - \mathbf{\mu})) \\\\ 
\log \mathcal{N} (\mathbf{x} | \mathbf{\mu}, \mathbf{\Sigma})
= -\frac{1}{2} ((\mathbf{x} - \mathbf{\mu}) ^ T \mathbf{\Sigma} ^ {-1} (\mathbf{x} - \mathbf{\mu}) + \log |\mathbf{\Sigma}| + D \log 2 \pi)
$$

上式において、$\mathbf{\Sigma}$は対角行列ならば、お互いの共分散が0なので$D$個の独立したガウス分布に分けられる。
共分散が0でなくても、シルベスター標準形に直すことで$D$この独立したガウス分布、ともやはりみなせる。

期待値は以下の通り。二次元の$x x ^ T$は行列を作る。

$$
\mathbb{E} [\mathbf{x}] = \mathbf{\mu} \\\\ 
\mathbb{E} [\mathbf{x} \mathbf{x} ^ T] = \mathbf{\mu} \mathbf{\mu} ^ T + \mathbf{\Sigma}
$$

エントロピーは以下の通り。期待値の部分は$\mathbf{x} ^ T D \mathbf{x}$で対角行列Dなので、**実質的にはtraceそのもの**。

$$
-\mathbb{E} [\log p(\mathbf{x})] = \frac{1}{2} (\mathbb{E} [(\mathbf{x} - \mathbf{\mu}) ^ T \mathbf{\Sigma} ^ {-1} (\mathbf{x} - \mathbf{\mu})] + \log |\mathbf{\Sigma}| + D \log 2\pi) \\\\ 
\mathbb{E} [(\mathbf{x} - \mathbf{\mu}) ^ T \mathbf{\Sigma} ^ {-1} (\mathbf{x} - \mathbf{\mu})]
= \mathbb{E} [\mathrm{Tr}((\mathbf{x} - \mathbf{\mu}) ^ T \mathbf{\Sigma} ^ {-1} (\mathbf{x} - \mathbf{\mu}))]  \\\\ 
= \mathbb{E} [\mathrm{Tr}((\mathbf{x} - \mathbf{\mu}) (\mathbf{x} - \mathbf{\mu}) ^ T) \mathbf{\Sigma} ^ {-1} ]
= \mathbb{E} [\mathrm{Tr}(\mathbf{x} \mathbf{x} ^ T - \mathbf{x} \mathbf{\mu} ^ T - \mathbf{\mu} \mathbf{x} ^ T + \mathbf{\mu} \mathbf{\mu} ^ T) \mathbf{\Sigma} ^ {-1} ] \\\\ 
=  \mathbb{E} [\mathrm{Tr}(\mathbf{\mu} \mathbf{\mu} ^ T + \mathbf{\Sigma} - 2\mathbf{\mu} \mathbf{\mu} ^ T + \mathbf{\mu} \mathbf{\mu} ^ T) \mathbf{\Sigma} ^ {-1} ] = \mathrm{Tr}(\mathbb{E} [I]) = D
$$

よって、

$$
-\mathbb{E} [\log p(\mathbf{x})] = \frac{1}{2} (\log |\mathbf{\Sigma}| + D(\log 2 \pi + 1))
$$

同様に、KLダイバージェンスは、

$$
\mathbb{E}_p [\log p(\mathbf{x})] - \mathbb{E}_p [\log q(\mathbf{x})] = \\\\ 
\frac{1}{2} (\mathrm{Tr}[((\mathbf{\mu_1} - \mathbf{\mu_2}) (\mathbf{\mu_1} - \mathbf{\mu_2}) ^ T + \mathbf{\Sigma}_2) \mathbf{\Sigma}_1 ^ {-1}] + \log \frac{|\mathbf{\Sigma_1}|}{|\mathbf{\Sigma_2}|} - D)
$$

## ウィシャート分布

$D \times D$の正定値行列を生成する分布。なので、先ほどの多次元ガウス分布の分散の逆行列(=**精度行列**)の生成に使える。$\nu$は自由度という量であり、$\nu > D - 1$。$\mathbf{W}$は$D \times D$の正定値行列であるパラメタ。

$$
\mathcal{W} (\mathbf{X} | \nu, \mathbf{W}) = (正則化項) |\mathbf{X}| ^ {\frac{\nu - D - 1}{2}} \exp(\frac{1}{2} \mathrm{Tr}(\mathbf{W} ^ {-1} \mathbf{X})) \\\\ 
\log \mathcal{W} (\mathbf{X} | \nu, \mathbf{W}) = \frac{\nu - D - 1}{2} \log |\mathbf{X}| - \frac{1}{2} {Tr}(\mathbf{W} ^ {-1} \mathbf{X}) + \log (正則化項)
$$

期待値は以下の通り。

$$
\mathbb{E} [\mathbf{X}] = \nu \mathbf{W} 
$$

